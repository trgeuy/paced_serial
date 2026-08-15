# Paced Serial Plugin

This is a plugin that was created for use with the Serial MCP Server found here

https://github.com/es617/serial-mcp-server

It is needed when using Claude to interact with vintage computers over a serial connection. Many of these older systems have no handshaking on their serial ports and cannot process a raw stream of data at even moderate baud rates without dropping characters. The plug in adds the following 4 tools to the base MCP server.

paced.configure — set/get per-connection default inter_char_gap_ms / eol_gap_ms, so you don't have to pass them on every write once you've found your device's numbers (e.g. 1.5ms / 10ms for your 9600-baud case).

paced.write — writes byte-by-byte with the inter-character gap, and applies the (separate, larger) end-of-line gap exactly once after a full CR or CR/LF is sent — not mid-sequence, and not doubled for CR/LF. Falls back to paced.configure defaults if gaps aren't specified per-call.

paced.calibrate — one measurement: sends known test lines at a candidate gap setting, waits for the echo to go quiet (not just the first chunk — it polls until there's been silence for quiet_ms), then diffs sent vs. received bytes using difflib and reports dropped/inserted/substituted byte counts plus a clean verdict.

paced.sweep — runs calibrate across a set of candidate gaps (cartesian product of your inter-char and eol lists, or paired lists via pairwise=true), flushing and settling between attempts, and returns every result plus the smallest clean combination found — this is the "run tests that the gaps are tuned" piece.
