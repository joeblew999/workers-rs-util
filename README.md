# workers-rs-util

Extra Rust crates for building on Cloudflare [`workers-rs`](https://github.com/cloudflare/workers-rs),
plus the nushell plugins that depend on them.

The anchor is **`cloudflare-shell`** — a backend-agnostic `FileSystem` trait so
nushell's file operations are portable: the same calls work on a native FS and
on Cloudflare (Durable Object SQLite + R2). That FS abstraction is *why* the
plugins live here alongside it.

## Related repositories

- **[http-nu](https://github.com/joeblew999/http-nu)**: Nushell-scriptable HTTP server; consumes `cloudflare-shell` from here.
- **[xs](https://github.com/joeblew999/xs)**: cross.stream event store, http-nu's companion.

## Crates

| Crate | Target | What it is |
|-------|--------|------------|
| `cloudflare-shell` | host + wasm | The `FileSystem` trait + types. The stable contract everything pins to. |
| `cloudflare-shell-workspace` | wasm | CF impl of the trait (DO SQLite + R2) — native-equivalent FS. |
| `cloudflare-shell-rpc/{types,server,client,demo-rust,demo-js}` | wasm | The FS exposed as a Worker RPC binding. Any Worker (JS or Rust) can bind to it. Standalone service. |
| `nu_plugin_push` | host | Web Push + notifications + a2hs nushell plugin. |
| `nu_plugin_email` + `cf_email_worker` | host + wasm | Email send/receive via Cloudflare Email Service. |
| `nu_plugin_cedar` | host | Cedar policy admin/editor nushell plugin. |

## Quickstart

```sh
mise trust
mise run plugins:build      # build the nushell plugin binaries
mise run plugins:test       # unit-test them

# FS-RPC service, end-to-end locally (no deploy, no auth) via wrangler dev:
mise run cf:fs:smoke:all    # boots server + JS + Rust demos, round-trips them
mise run cf:fs:up           # leave them running; cf:fs:down to stop

# cf_email_worker:
mise run email:worker:check # wasm type-check
```

Deploy tasks (`cf:fs:deploy:all`, `email:worker:deploy`) pull `CLOUDFLARE_API_TOKEN`
and runtime secrets from `fnox` (keychain provider — see `fnox.toml`).

## Consuming the FS-RPC from another project

Deploy `cloudflare-shell-rpc/server`, then bind to it as a service from your own
Worker (JS or Rust via `cloudflare-shell-rpc-client`). Auth is the
`SHELL_FS_TOKEN` bearer.
