# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Building NGINX

```bash
# Configure (from repo root)
auto/configure

# Build
make

# Install (default: /usr/local/nginx/)
make install
```

Run a single test from the nginx-tests repo (clone separately):
```bash
perl t/TEST <test_name>
```

## Architecture Overview

NGINX is organized into functional modules under `src/`:

- **core/** — Core runtime: cycle management, memory pools (`ngx_palloc.*`), logging, hash tables, red-black trees, string utilities, resolver, thread pools, configuration parsing (`ngx_conf_file.*`), module framework (`ngx_module.*`)
- **event/** — Event loop and I/O multiplexing: `ngx_event.*` handles kqueue/epoll/select; `ngx_event_accept.*` for incoming connections; `ngx_event_connect.*` for upstream connections; `ngx_event_openssl.*` for TLS; `quic/` for HTTP/3 QUIC support
- **http/** — HTTP server: `ngx_http_core_module.c` is the main HTTP module handling location matching, request processing phases, and response filtering chain; `ngx_http_request.*` for request parsing and body handling; `ngx_http_upstream.*` for proxying; `ngx_http_variables.*` for config variables; `v2/` and `v3/` for SPDY/HTTP2 and HTTP/3; `modules/` for optional compiled-in modules
- **mail/** — Mail proxy: IMAP, POP3, SMTP protocol handlers and authentication
- **stream/** — TCP/UDP proxy and load balancing: `ngx_stream_core_module.c`, `ngx_stream_proxy_module.c`, upstream modules (round-robin, least_conn, hash, random)
- **os/unix/** and **os/win32/** — OS-specific implementations for Linux/FreeBSD/macOS and Windows

## Key Patterns

- **Modules** use the `ngx_module_t` struct with init/main/exit callbacks. Configuration is parsed via `ngx_command_t` arrays.
- **Filters** form a chain (e.g., write filter, header filter, copy filter) — each filter calls the next.
- **Upstreams** manage backend connections with keepalive, load balancing, and retry logic.
- **Shared memory** is used for rate limiting, cache zones, and state across worker processes.
- **Event-driven** design: non-blocking I/O, posted events, and timer management.

## Contributing

- Commit message format: single-line subject (72 chars), blank line, body (72 chars). Use prefixes: `Upstream:`, `QUIC:`, `Core:`, `Mail:`, `Stream:`.
- Code style must match existing NGINX sources — no braces on single-line conditionals, spaces after keywords, etc.
- Tests: https://github.com/nginx/nginx-tests
- F5 CLA required before merging (bot will prompt).