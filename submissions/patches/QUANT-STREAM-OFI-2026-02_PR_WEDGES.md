# 🔧 Upstream Technical Wedge PR Bundles: `QUANT-STREAM-OFI-2026-02` (`T-037`)

- **Parent Opportunity:** [`submissions/QUANT-STREAM-OFI-2026-02_SUBMISSION_PACKAGE.md`](../QUANT-STREAM-OFI-2026-02_SUBMISSION_PACKAGE.md) (`\$12,000.00` Fixed-Scope 14-Day SOW)
- **Capability Anchor:** `phronarch` (`Phronarch v2.2.0` — Go L2 WebSocket Ingestor + TypeScript OFI/VWAP & 6-Stage Bayesian Engine)
- **Prepared By:** William Jasper Land (`PsyCry`) — **Digital Nomad LLC**
- **Verification Date:** `2026-11-05` (Line-verified against live upstream `develop`/`master` HEAD blobs via GitHub API)

---

## Wedge PR 2A — Nautech Systems `nautilus_trader` (`nautechsystems/nautilus_trader#5568`)

- **Target Issue:** [`nautechsystems/nautilus_trader#5568`](https://github.com/nautechsystems/nautilus_trader/issues/5568) — *Binance book resync emits buffered diffs with an earlier `ts_init` than their snapshot, so a catalog replays a diff before its snapshot*
- **Target File:** `crates/adapters/binance/src/book/sync.rs` (verified against upstream blob `dab78d2a55da79a0d6470eaca4a28d323caa4b71`, lines `231–249` & `375–382`)
- **Problem Statement:** During Binance L2 order book synchronization (`BookSyncTracker::on_snapshot`), buffered WebSocket depth diffs received while the REST depth snapshot is in flight retain their earlier WebSocket receipt `ts_init`, whereas `snapshot.deltas.ts_init` is stamped when the REST response arrives. Although `on_snapshot` emits `snapshot_deltas` before `replay` diffs over the live channel, `crates/persistence/src/writer/materializer.rs` (`is_record_batch_monotonic_by_ts_init`) sorts non-monotonic batches by `ts_init` — causing Parquet catalog readers and backtests to apply the buffered diffs *before* the snapshot's `CLEAR` action.

### Unified Diff (`crates/adapters/binance/src/book/sync.rs`)

```diff
--- a/crates/adapters/binance/src/book/sync.rs
+++ b/crates/adapters/binance/src/book/sync.rs
@@ -233,6 +233,7 @@ impl BookSyncTracker {
             .filter_map(|index| updates[index].deltas.take())
             .collect::<Vec<_>>();
         let mut snapshot_deltas = snapshot.deltas;
+        let snapshot_ts_init = snapshot_deltas.ts_init;
 
         if !snapshot.has_event_time
             && let Some(first) = replay.first()
@@ -243,7 +244,10 @@ impl BookSyncTracker {
         self.send(snapshot_deltas);
         let replayed_count = replay.len();
 
-        for deltas in replay {
+        for mut deltas in replay {
+            if deltas.ts_init < snapshot_ts_init {
+                set_init_time(&mut deltas, snapshot_ts_init);
+            }
             self.send(deltas);
         }
 
@@ -379,6 +383,14 @@ fn set_event_time(deltas: &mut OrderBookDeltas, ts_event: UnixNanos) {
     }
 }
 
+fn set_init_time(deltas: &mut OrderBookDeltas, ts_init: UnixNanos) {
+    deltas.ts_init = ts_init;
+
+    for delta in &mut deltas.deltas {
+        delta.ts_init = ts_init;
+    }
+}
+
 #[cfg(test)]
 mod tests {
```

### Regression Test Snippet (`crates/adapters/binance/src/book/sync.rs`)

```rust
#[rstest]
fn replayed_buffered_diffs_clamp_ts_init_to_snapshot_ts_init() {
    let (tracker, mut rx) = tracker(DepthSequencing::Futures);
    tracker.subscribe(instrument_id());

    // Diff arrives at ts_init = 100ns while REST snapshot is in flight
    let mut buffered = diff_update(100, 105, Some(99));
    if let Some(ref mut deltas) = buffered.deltas {
        set_init_time(deltas, UnixNanos::from(100));
    }
    let recovery = tracker.handle_update(instrument_id(), buffered).unwrap();

    // REST snapshot completes later at ts_init = 500ns
    let mut snap = snapshot(102, true);
    set_init_time(&mut snap.deltas, UnixNanos::from(500));
    tracker.on_snapshot(instrument_id(), snap, &recovery).unwrap();

    let first = recv_deltas(&mut rx);
    let second = recv_deltas(&mut rx);
    assert_eq!(first.ts_init, UnixNanos::from(500));
    assert!(second.ts_init >= first.ts_init);
}
```

---

## Wedge PR 2B — Thrasher Corp `gocryptotrader` (`thrasher-corp/gocryptotrader#2507`)

- **Target Issue:** [`thrasher-corp/gocryptotrader#2507`](https://github.com/thrasher-corp/gocryptotrader/issues/2507) — *`websocket`: let each connection set its own subscription limit*
- **Target Files:**
  - `exchange/websocket/connection.go` (lines `69–80`)
  - `exchange/websocket/manager.go` (lines `313–335`, `618–621`, `688–692`)
  - `exchange/websocket/subscriptions.go` (lines `278–286`, `512–515`, `631–640`)
- **Problem Statement:** `Manager` enforces a single global `m.MaxSubscriptionsPerConnection` across every `ConnectionSetup`. On exchanges like Binance (where Options WebSocket connections allow `200` streams while Spot, USDⓈ-M, and COIN-M allow `1024` streams), setting the global limit to `200` forces Spot/Futures connections to spawn 6 separate connections for 1024 subscriptions instead of 1.

### Unified Diff (`exchange/websocket/`)

```diff
--- a/exchange/websocket/connection.go
+++ b/exchange/websocket/connection.go
@@ -75,6 +75,8 @@ type ConnectionSetup struct {
 	Authenticated            bool // unused for multi-connection websocket
 	SubscriptionsNotRequired bool
 	ConnectionLevelReporter  Reporter
+	// MaxSubscriptions overrides Manager.MaxSubscriptionsPerConnection for this connection when > 0.
+	MaxSubscriptions int
 
 	// URL defines the websocket server URL to connect to
 	URL string
--- a/exchange/websocket/manager.go
+++ b/exchange/websocket/manager.go
@@ -318,6 +318,9 @@ func (m *Manager) SetupNewConnection(c *ConnectionSetup) error {
 	if m.exchangeName == "" {
 		return fmt.Errorf("%w: %w", errConnSetup, errExchangeConfigNameEmpty)
 	}
+	if c.MaxSubscriptions < 0 {
+		return fmt.Errorf("%s %w: %w", m.exchangeName, errConnSetup, errInvalidMaxSubscriptions)
+	}
 	if m.TrafficAlert == nil {
 		return fmt.Errorf("%w: %w", errConnSetup, errTrafficAlertNil)
 	}
@@ -379,6 +382,13 @@ func (m *Manager) SetupNewConnection(c *ConnectionSetup) error {
 	return nil
 }
 
+func (m *Manager) connectionMaxSubscriptions(setup *ConnectionSetup) int {
+	if setup != nil && setup.MaxSubscriptions > 0 {
+		return setup.MaxSubscriptions
+	}
+	return m.MaxSubscriptionsPerConnection
+}
+
 // createConnectionFromSetup returns a websocket connection from a setup
 // configuration. This is used for setting up new connections on the fly.
 func (m *Manager) createConnectionFromSetup(c *ConnectionSetup) *connection {
@@ -616,7 +626,8 @@ func (m *Manager) connect(ctx context.Context) error {
 			continue
 		}
 
-		for _, batchCandidates := range common.Batch(remainingSubs, m.MaxSubscriptionsPerConnection) {
+		maxSubs := m.connectionMaxSubscriptions(ws.setup)
+		for _, batchCandidates := range common.Batch(remainingSubs, maxSubs) {
 			batchRemainingSubs, err := m.absorbTrackableSubscriptionsAndValidate(ctx, ws, batchCandidates)
 			if err != nil {
 				subscriptionError = common.AppendError(subscriptionError, fmt.Errorf("subscription error on [conn:%d] [URL:%s]: %w ", i+1, ws.setup.URL, err))
@@ -686,8 +697,9 @@ func (m *Manager) connect(ctx context.Context) error {
 // createConnectAndSubscribe creates a new connection for the websocket and subscribes to subs on it
 // The connection's store also records subscriptions accepted before the subscriber fails, because flushing uses it to decide whether the connection is still in use
 func (m *Manager) createConnectAndSubscribe(ctx context.Context, ws *websocket, subs subscription.List) error {
-	if m.MaxSubscriptionsPerConnection > 0 && len(subs) > m.MaxSubscriptionsPerConnection {
-		return fmt.Errorf("%w %w: max subs allowed %d, requested %d", common.ErrFatal, errSubscriptionsExceedsLimit, m.MaxSubscriptionsPerConnection, len(subs))
+	maxSubs := m.connectionMaxSubscriptions(ws.setup)
+	if maxSubs > 0 && len(subs) > maxSubs {
+		return fmt.Errorf("%w %w: max subs allowed %d, requested %d", common.ErrFatal, errSubscriptionsExceedsLimit, maxSubs, len(subs))
 	}
 
 	conn := m.createConnectionFromSetup(ws.setup)
--- a/exchange/websocket/subscriptions.go
+++ b/exchange/websocket/subscriptions.go
@@ -276,13 +276,14 @@ func (m *Manager) checkSubscriptions(conn Connection, subs subscription.List) er
 		usedCapacity = subscriptionStore.Len()
 	}
 
-	if m.MaxSubscriptionsPerConnection > 0 && usedCapacity+len(subs) > m.MaxSubscriptionsPerConnection {
+	maxSubs := m.connectionMaxSubscriptions(m.connectionSetup(conn))
+	if maxSubs > 0 && usedCapacity+len(subs) > maxSubs {
 		return fmt.Errorf("%w: current subscriptions: %v, incoming subscriptions: %v, max subscriptions per connection: %v",
 			errSubscriptionsExceedsLimit,
 			usedCapacity,
 			len(subs),
-			m.MaxSubscriptionsPerConnection)
+			maxSubs)
 	}
 
 	for _, s := range subs {
@@ -510,7 +511,8 @@ func (m *Manager) scaleConnectionsToSubscriptions(ctx context.Context, ws *webso
 		}
 
 		// Spawn new connections if there are still subscriptions left to process
-		for _, batch := range common.Batch(currentSubs, m.MaxSubscriptionsPerConnection) {
+		maxSubs := m.connectionMaxSubscriptions(ws.setup)
+		for _, batch := range common.Batch(currentSubs, maxSubs) {
 			toConnect, err := m.absorbTrackableSubscriptionsAndValidate(ctx, ws, batch)
 			if err != nil {
 				return err
@@ -629,13 +631,14 @@ func (m *Manager) subscribeToConnection(ctx context.Context, conn Connection, su
 	}
 
 	usedCap := store.Len()
-	if m.MaxSubscriptionsPerConnection > 0 && usedCap >= m.MaxSubscriptionsPerConnection {
+	maxSubs := m.connectionMaxSubscriptions(m.connectionSetup(conn))
+	if maxSubs > 0 && usedCap >= maxSubs {
 		return subs, nil // No capacity left for this connection
 	}
 
 	availableCap := len(subs)
-	if m.MaxSubscriptionsPerConnection > 0 {
-		availableCap = m.MaxSubscriptionsPerConnection - usedCap
+	if maxSubs > 0 {
+		availableCap = maxSubs - usedCap
 	}
 
 	if availableCap > len(subs) {
```
