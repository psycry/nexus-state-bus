# 🔧 Upstream Technical Wedge PR Bundles: `MCP-RUST-ORCH-2026-01` (`T-037`)

- **Parent Opportunity:** [`submissions/MCP-RUST-ORCH-2026-01_SUBMISSION_PACKAGE.md`](../MCP-RUST-ORCH-2026-01_SUBMISSION_PACKAGE.md) (`\$8,500.00` Fixed-Scope 14-Day SOW)
- **Capability Anchor:** `agent_smith` (`claw-mcp-server v0.2.0` — Rust `rmcp` + Hybrid Router + PostgreSQL `pgvector`)
- **Prepared By:** William Jasper Land (`PsyCry`) — **Digital Nomad LLC**
- **Verification Date:** `2026-11-05` (Line-verified against live upstream `main` HEAD blobs via GitHub API)

---

## Wedge PR 1A — Official Rust MCP SDK (`modelcontextprotocol/rust-sdk#1420`)

- **Target Issue:** [`modelcontextprotocol/rust-sdk#1420`](https://github.com/modelcontextprotocol/rust-sdk/issues/1420) — *OAuth discovery drops explicit default ports from advertised issuer identifiers* (`P1`, `T-security`, `T-transport`, `ready for work`)
- **Target File:** `crates/rmcp/src/transport/auth.rs` (verified against upstream blob `3d7816b540786a404ddde753a159de8ddc78385a`, lines `2911–2953`)
- **Problem Statement:** `authorization_metadata_from_resource_metadata()` parses each `candidate` with `Url::parse(candidate)` and passes `Some(candidate_url.as_str())` as the expected issuer to `try_discover_oauth_server()`. Because the `url` crate strips explicit default ports (`:443` for `https`, `:80` for `http`) and normalizes host casing during serialization, RFC 8414 §3.3 / §4 exact string identity validation fails whenever an authorization server advertises an explicit default port (e.g., `https://auth.example.com:443/tenant1`) or asymmetrically matches normalized metadata.

### Unified Diff (`crates/rmcp/src/transport/auth.rs`)

```diff
--- a/crates/rmcp/src/transport/auth.rs
+++ b/crates/rmcp/src/transport/auth.rs
@@ -2911,16 +2911,16 @@ impl AuthorizationManager {
         for candidate in &candidates {
-            let candidate_url = match Url::parse(candidate) {
-                Ok(url) => url,
+            let (candidate_url, expected_issuer) = match Url::parse(candidate) {
+                Ok(url) => (url, Some(candidate.as_str())),
                 Err(_) => match resource_metadata_url.join(candidate) {
-                    Ok(url) => url,
+                    Ok(url) => (url, None),
                     Err(e) => {
                         debug!("Failed to resolve authorization server URL `{candidate}`: {e}");
                         continue;
                     }
                 },
             };
@@ -2942,9 +2942,10 @@ impl AuthorizationManager {
             // Discovery URL construction removes a non-root trailing slash. Keep the issuer
-            // advertised by protected resource metadata for the exact RFC 8414 comparison.
+            // advertised by protected resource metadata (including explicit default ports
+            // and host casing) for the exact RFC 8414 §3.3 comparison.
+            let expected = expected_issuer.unwrap_or_else(|| candidate_url.as_str());
             if let Some(metadata) = self
-                .try_discover_oauth_server(&candidate_url, Some(candidate_url.as_str()))
+                .try_discover_oauth_server(&candidate_url, Some(expected))
                 .await?
             {
                 return Ok(Some(metadata));
             }
```

### Regression Test Snippet (`crates/rmcp/src/transport/auth.rs`)

```rust
#[tokio::test]
async fn test_oauth_discovery_preserves_explicit_default_port_in_issuer() {
    let client = RecordingOAuthHttpClient::new()
        .with_json_response(
            "https://mcp.example.com/.well-known/oauth-protected-resource",
            serde_json::json!({
                "resource": "https://mcp.example.com/",
                "authorization_servers": ["https://auth.example.com:443/tenant1"]
            }),
        )
        .with_json_response(
            "https://auth.example.com:443/.well-known/oauth-authorization-server/tenant1",
            serde_json::json!({
                "issuer": "https://auth.example.com:443/tenant1",
                "authorization_endpoint": "https://auth.example.com/tenant1/authorize",
                "token_endpoint": "https://auth.example.com/tenant1/token"
            }),
        );

    let manager = AuthorizationManager::new("https://mcp.example.com/", client).unwrap();
    let metadata = manager
        .resolve_metadata()
        .await
        .expect("discovery should succeed")
        .expect("metadata should be present");
    assert_eq!(metadata.issuer, "https://auth.example.com:443/tenant1");
}
```

---

## Wedge PR 1B — Linux Foundation / Solo.io `agentgateway` (`agentgateway/agentgateway#4121`)

- **Target Issue:** [`agentgateway/agentgateway#4121`](https://github.com/agentgateway/agentgateway/issues/4121) — *MCP OpenAPI target: backend connect failure on `tools/call` returns plain-text 503, disconnecting streamable HTTP clients*
- **Target File:** `crates/agentgateway/src/mcp/upstream/openapi/mod.rs` (lines `706–711`)
- **Problem Statement:** For an `openapi` MCP target, `agentgateway` is the terminating MCP server translating `tools/call` into outbound REST calls. When `self.http_client.call(request).await` fails with a transport/connect error (`ProxyError::UpstreamCallFailed`), `call_tool` propagates `UpstreamError::Proxy(p)` to `mcp/session.rs` (`handle_error`), which emits a plain-text HTTP 503 with no JSON-RPC body and no request `id` — causing streamable HTTP MCP clients (VS Code, Cursor, Claude) to drop the entire MCP server session instead of allowing the model to observe `isError: true` on the tool invocation.

### Unified Diff (`crates/agentgateway/src/mcp/upstream/openapi/mod.rs`)

```diff
--- a/crates/agentgateway/src/mcp/upstream/openapi/mod.rs
+++ b/crates/agentgateway/src/mcp/upstream/openapi/mod.rs
@@ -704,7 +704,17 @@ impl Handler {
 				return Err(UpstreamError::InvalidMethod(method.to_string()));
 			},
 			ClientRequest::CallToolRequest(ctr) => {
-				let res = self
-					.call_tool(ctr.params.name.as_ref(), ctr.params.arguments, ctx)
-					.await?;
+				let res = match self
+					.call_tool(ctr.params.name.as_ref(), ctr.params.arguments, ctx)
+					.await
+				{
+					Ok(res) => res,
+					Err(UpstreamError::Proxy(err)) => {
+						warn!(
+							"OpenAPI tool '{}' upstream transport error: {}",
+							ctr.params.name, err
+						);
+						CallToolResult::error(vec![ContentBlock::text(err.to_string())])
+					},
+					Err(err) => return Err(err),
+				};
 				Messages::from_result(id, res)
 			},
 			ClientRequest::ListToolsRequest(_) => Messages::from_result(
```
