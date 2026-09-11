# Paced Serial Plugin

This plugin adds character-pacing tools to the Serial MCP Server:

https://github.com/es617/serial-mcp-server

Many vintage computers have no handshaking on their serial ports. At even
moderate baud rates, they drop characters from a raw data stream. This
plugin adds a short, configurable gap between characters, the same way many
terminal emulators do.

## Want the pacing built in, with no plugin install?

Use our fork of the Serial MCP Server instead:

https://github.com/trgeuy/serial-mcp-server

The fork includes this plugin's tools as a native, built-in feature (set
`SERIAL_MCP_PACED=1`). It also adds a TCP mirror transport, exclusive
write-locking for multi-step command sequences, and other fixes for
vintage-hardware use. Use this plugin only if you want the tools as a
separate add-on for the original, unforked server.

## Tools

This plugin adds four tools to the base MCP server.

**`paced.configure`**
Sets or reads the default `inter_char_gap_ms` and `eol_gap_ms` for a
connection. Set these once, then skip them on every later `paced.write`
call. Example: 1.5 ms / 10 ms for a 9600-baud connection.

**`paced.write`**
Writes data one byte at a time, with the configured gap between characters.
It applies the (larger) end-of-line gap once, after a full CR or CR/LF. It
does not double the gap for CR/LF. If you do not pass gap values, it uses
the `paced.configure` defaults.

**`paced.calibrate`**
Tests one gap setting. It sends known test lines, waits for the echo to go
quiet, then compares the sent and received bytes with `difflib`. It reports
the counts of dropped, inserted, and substituted bytes, and a pass/fail
verdict.

**`paced.sweep`**
Runs `paced.calibrate` across a set of candidate gaps. Pass separate
inter-char and EOL gap lists to test every combination, or set
`pairwise=true` to test them as matched pairs. It flushes and settles the
connection between attempts, then returns every result plus the smallest
gap combination that passed.
