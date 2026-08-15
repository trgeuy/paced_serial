Prompt used with Claude:

I need an mcp server that will allow claude to connect to vintage computers over a serial connection using USB to serial adapters. I have tested this currently available server - https://github.com/es617/serial-mcp-server - which has most of the functionality I need. Review the "What the agent can do" section for it's list of capabilities. It is missing the pacing ability available in many terminal emulation applications that place a short, configurable gap between characters when a block is send. In addition it needs to have the ability to tune the gaps using test data. It needs both an inter character gap and an end of line gap. 

I have observed that the inter character gap when running at 9600 baud generally is 1 to 2 ms. In certain situation an inter line gap, following a CR or CR/LF, of 10 ms is needed. My goal is to be able to run tests where the the gaps are tuned by sending text and observing when no characters are dropped from the stream echoed back from the external system.


The four tools it adds:

paced.configure — set/get per-connection default inter_char_gap_ms / eol_gap_ms, so you don't have to pass them on every write once you've found your device's numbers (e.g. 1.5ms / 10ms for your 9600-baud case).

paced.write — writes byte-by-byte with the inter-character gap, and applies the (separate, larger) end-of-line gap exactly once after a full CR or CR/LF is sent — not mid-sequence, and not doubled for CR/LF. Falls back to paced.configure defaults if gaps aren't specified per-call.

paced.calibrate — one measurement: sends known test lines at a candidate gap setting, waits for the echo to go quiet (not just the first chunk — it polls until there's been silence for quiet_ms), then diffs sent vs. received bytes using difflib and reports dropped/inserted/substituted byte counts plus a clean verdict.

paced.sweep — runs calibrate across a set of candidate gaps (cartesian product of your inter-char and eol lists, or paired lists via pairwise=true), flushing and settling between attempts, and returns every result plus the smallest clean combination found — this is the "run tests that the gaps are tuned" piece.

Plugin works with this Serial MCP Server:

	https://github.com/es617/serial-mcp-server

Register the MCP Server with Claude - plugin and mirror (rw mode) enabled:

	claude mcp add serial -e SERIAL_MCP_PLUGINS=paced_serial SERIAL_MCP_MIRROR=rw -- serial_mcp

Using the MCP Server:

Once the MCP serv er is registered, start Claude as normal. If you have already registered the MCP server in a prior session it sometimes stays registered between sessions but I have not nailed down the pattern yet. If it is already registered you will get an error. If you need to change the options, it appears you need to use 'claude mcp remove serial' to remove it and then reregister. If in doubt, using 'claude mcp list' will tell you what MCP servers are registered. The folder you are in defines the project origin for Claude when it is started. I believe at this point that registration is relative to the project. I'm sure there is a way to make it global, but have yet to figure it out.

Register MCP Server with Claude - plugin and mirror (rw mode) enabled:

	claude mcp add serial -e SERIAL_MCP_PLUGINS=paced_serial SERIAL_MCP_MIRROR=rw -- serial_mcp

Start CLaude:

	claude

Tell Claude to connect to your USB serial adapter:

	conenct to /dev/cu.usbserial-FTAJCNNY0 at 9600 baud, 8 data bits, no parity, 1 stop bit

Connecting the the PTY Mirror:

I use VOC9 in VT52 emulation as this is how I have my UCSD Pascl system configured. In this situation, the screen has to be 80x24 otherwise the terminal emulation gets confused. Connect to /tmp/serial-mcp0 once PTY Mirror is active. It becomes active after Claude has connected to the USB Serial device.

The paced serial plugin is used to slow down the character stream if you notice dropped characters. Start by telling Claude to test for drops. He sends a stream of characters and compares what he gets back. Then he set's what works and uses the paced.write tool from there on in place of the MCP's built in write. There are two gaps, inter character and inter line, to allow for "I can't process characters arriving at 9600 baud" and "I need time to process a new line before I can accept more input". Think of the seconnd as a way to handle pasting a text file of a Basic program into MBASIC.

Note: The MCP I've been using only supports the PTY Mirror on MacOS and Linux. I don't know if this is the result a Windows limitation or if the author didn't choose to add this feature.

Note: This should work with any AI application that supports MCP, just don't ask me how.


