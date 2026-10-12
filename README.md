# ps4
Enhanced PS4 Webkit (5.00 - 13.52)
PS4 WebKit Exploit — Console

A single-shot PlayStation 4 WebKit exploit harness with a built-in on-screen console, real-time PASS/FAIL status reporting, and .txt log export. The exploit builds a fake host object to obtain an arbitrary read/write primitive, then optionally performs a kernel jailbreak.

Author tag: mansoor0x + OptiTronOffical
Revision: genuine-u8-rw-3-1352
Live host: https://1350-webkit-test.vercel.app/

WARNING — For research and educational use only.
The kernel stage does NOT fire by default — no kernel corruption will occur unless you explicitly pass ?jb=1. Only run this on hardware you personally own. Kernel-level memory corruption can brick your console or cause data loss.

NOTE — This is not a full jailbreak. It is the userland portion that can be chained into a jailbreak. That is why, by default, it halts cleanly after the R/W primitive passes.

==================================================
TABLE OF CONTENTS
==================================================

1. What It Does
2. Features
3. Requirements
4. Quick Start
5. Hosting It Yourself
6. URL Parameters
7. Reading the Output
8. Log Export
9. How It Works
10. Supported Firmware
11. Troubleshooting
12. Credits

==================================================
WHAT IT DOES
==================================================

The page runs a WebKit memory-corruption exploit that produces a genuine arbitrary read/write primitive. When the primitive is verified, the page halts by default and reports the result. The kernel jailbreak stage is opt-in via a URL parameter, so a failed exploit cannot trap the console in a crash/reload loop.

Stage 1 — R/W primitive
What it does: Builds a fake JSObject host, leaks object addresses, and demonstrates read + write through a corrupted Uint8Array.
When it runs: Always.

Stage 2 — Kernel jailbreak
What it does: Hijacks sysent, patches ucred, and swaps the file descriptor root vnode.
When it runs: Only when ?jb=1 is present.

==================================================
FEATURES
==================================================

- Live console — every log line is timestamped and color-coded by severity (pass / fail / warn / info).

- Status bar — shows the current state at a glance: INITIALIZING, RUNNING, PASS, FAIL, or HALTED, with live PASS/FAIL/WARN counters.

- Single-shot execution — no auto-retry loop. One attempt, one clean terminal state.

- Jailbreak gate — the kernel stage is disabled unless you explicitly opt in with ?jb=1.

- Log export — download the full session as a timestamped .txt, copy to clipboard, or auto-save on completion.

- Firmware auto-detection — reads the PS4 firmware version from the user agent and selects the correct kernel offsets.

==================================================
REQUIREMENTS
==================================================

- A PlayStation 4 whose firmware is listed in the Supported Firmware section.

- A way to load the HTML file on the console (local web server, USB-hosted page, or a host payload loader).

- A browser/user-agent string that includes the firmware version, e.g. PlayStation 4/9.00.

==================================================
QUICK START
==================================================

1. Basic run — R/W primitive only (safe, default)

Load the page with no parameters:

    index.html

On success you will see:

    N-rw-GENUINE-UB-RW-PASS-carrier-vector-restored=true
    N-rw-HALTED-BEFORE-JAILBREAK-rw-primitive-ready-no-kernel-writes-performed
    N-rw-HALT-NOTE-append-?jb=1-to-url-to-run-kernel-jailbreak

The status bar turns amber with "R/W PASS — HALTED". No kernel memory is touched.

2. Full run — R/W primitive + kernel jailbreak

Append ?jb=1:

    index.html?jb=1

On success you will see:

    N-rw-JAILBREAK-GATE-explicitly-enabled-by-url-param
    N-rw-===== PS4 JAILBREAK START =====-fw=9.00
    ...
    N-rw-===== JAILBREAK COMPLETE =====-PS4 9.00 root=true-fs=true

The status bar turns green with "JAILBREAK COMPLETE — PASS".

Note: Firmware label (fw=9.00 above) is auto-detected from the user agent.

3. Debug run

Add ?dbg=1 alongside ?jb=1 to enable verbose kernel scan logging:

    index.html?jb=1&dbg=1

This adds SCAN-HIT, UCRED-SIG-CANDIDATE, and (on failure) a PROC-DUMP block.

==================================================
HOSTING IT YOURSELF
==================================================

Since it's a single HTML file, the simplest options are:

Python 3 — serve the current directory on port 8000

    python3 -m http.server 8000
    → http://<your-lan-ip>:8000/

Node (npx) — same idea

    npx serve .

Or upload index.html to any static host (GitHub Pages, Netlify, Vercel, Cloudflare Pages) and open the URL on the PS4.

==================================================
URL PARAMETERS
==================================================

Parameter: jb
Values: 1
Default: (unset)
Description: Enable the kernel jailbreak stage. Without this, the exploit halts after the R/W primitive.

Parameter: n
Values: 64–1024
Default: 128 (falls back to 512 if unparsable)
Description: Number of 64 KiB ArrayBuffers to allocate during heap grooming. Clamped to the valid range.

Parameter: l
Values: 1
Default: (unset)
Description: Reserved — currently parsed but unused.

Parameter: dbg
Values: 1
Default: (unset)
Description: Verbose kernel scan logging inside runJailbreak().

Examples:

    index.html                                  # R/W only, halt after pass
    index.html?jb=1                             # full run
    index.html?jb=1&n=512                       # full run, larger groom
    index.html?jb=1&dbg=1                       # full run with debug dump
    index.html?n=1024                           # R/W only, max drain

==================================================
READING THE OUTPUT
==================================================

Status bar colors:

Grey             → INITIALIZING  → Page loaded, exploit not yet running
Blue (pulsing)   → RUNNING       → Exploit in progress
Green            → PASS          → R/W primitive verified, or jailbreak complete
Amber            → HALTED        → R/W primitive passed; jailbreak skipped (no ?jb=1)
Red              → FAIL          → Terminal failure — see the console for the reason

Console line levels:

pass  → A stage succeeded           → GENUINE-UB-RW-PASS, JAILBREAK COMPLETE, SANDBOX-ESCAPE success
fail  → A stage failed terminally   → FAILED-TERMINAL, HEADER-MISMATCH, LOAD-THREW, JAILBREAK-ERROR
warn  → Non-fatal anomaly or halt   → HALTED-BEFORE-JAILBREAK, SSV-PLACEMENT-MISS, NO-RESULT
info  → Normal progress             → BOOT, SSV-BUILT, ADDROF-POINTERS, ROOTS-LIVE

Key milestone tags:

BOOT                      → Exploit started; shows revision, k, n, and jailbreak gate state
ADDROF-POINTERS           → Leaked host and target object addresses (HOST=, TARGET=)
FAKE-ADDRESS              → Computed the fake host address (fake = host + 0x10)
ARBITRARY-READ-PASS       → Read primitive verified
ARBITRARY-WRITE-PASS      → Write primitive verified and restored
GENUINE-UB-RW-PASS        → Both primitives confirmed — R/W is live
HALTED-BEFORE-JAILBREAK   → R/W passed; kernel stage skipped
JAILBREAK COMPLETE        → Kernel jailbreak finished successfully

==================================================
LOG EXPORT
==================================================

The toolbar at the top of the page provides three actions:

⬇ Download log (.txt)  → Saves the full session as ps4-exploit-log-<timestamp>.txt
Copy log               → Copies the same text to the clipboard
Clear                  → Empties the console (does not affect the exploit)

Two checkboxes are also available:

auto-scroll                 → keep the newest line visible (on by default).
auto-save .txt on finish    → automatically download the log when a terminal state (PASS, HALTED, or FAIL) is reached.

Log file contents — the exported .txt contains a header block followed by every console entry:

    ==================================================
     WebKit Exploit Log — OptiTronOffical
    ==================================================
    Revision     : genuine-u8-rw-3-1352
    Firmware     : 9.00
    User Agent   : PlayStation 4/9.00 ...
    Started      : 2026-...
    Ended        : 2026-...
    Jailbreak    : DISABLED (halt before runJailbreak)
    Final Status : R/W PASS — HALTED  [HALTED]
    Counts       : PASS=8  FAIL=0  WARN=2  INFO=47
    --------------------------------------------------
    [HH:MM:SS.mmm] [INFO] 1-rw-BOOT-...
    ...
    --------------------------------------------------
    END OF LOG (57 entries)

==================================================
HOW IT WORKS
==================================================

Stage 1 — Heap grooming and fake host

1. Drain — Allocate DRAIN_COUNT 64 KiB ArrayBuffers to stabilize the heap.
2. Slab transfer — Send a 4 MiB buffer through a MessageChannel to free it on the other side.
3. Hole carving — Free butterfly holes, separators, guards, and a predecessor buffer at controlled offsets.
4. Fake host — Build a fakeHost object whose q0 slot contains an encoded ArrayBuffer header and whose q2 / q3 slots point at a Uint8Array and a length word.
5. Serialize — Store a graph into history.replaceState with a duplicate reference at index 2.

Stage 2 — Address leak (addrof)

The exploit leaks object addresses via a Symbol.prototype.toString trick:

- A Leaker class returns super.foo, which triggers a proxy get on the prototype.
- The proxy returns the receiver, leaking the scope object.
- A getter is installed on the leaked scope, and Object(scope.g) wraps it.
- Calling symbolToString on that wrapper copies the getter's arguments (including the host and target pointers) into a string.
- The string's UTF-16 characters are read back to reconstruct the 64-bit host and target addresses.

Stage 3 — R/W primitive

- The predecessor buffer is filled with copies of fakeAddress so the allocator reuses it as a fake ArrayBuffer cell.
- Loading history.state pulls the corrupted cell back into JS.
- The exploit verifies the fake cell header, upgrades fakeHost.q0, and points the carrier's vector at the target cell.
- A round-trip read (rwView[0] === 0xa5) and write (rwView[0] = 0x5a) through both views confirms the primitive.
- The original vector is restored so the heap stays consistent.

Stage 4 — Kernel jailbreak (opt-in)

Only runs with ?jb=1:

1. Hijack sysent[661] — Point it at a jmp rsi gadget with 2 args.
2. Patch ucred — Scan the proc struct for the 0x726f7272 ("rror") marker, then walk back to find ucred. Set UID/GID to 0 and swap the prison pointer.
3. Swap root vnode — Read rootvnode and point the file descriptor's fd_rdir / fd_jdir at it.
4. Restore sysent[661] — Put the original handler back.

The read64 / write64 helpers temporarily retarget globalFakeArray's vector at an arbitrary address, perform the access, and restore the vector.

==================================================
SUPPORTED FIRMWARE
==================================================

The PS4_OFFSETS table covers these firmware versions:

5.x  : 5.00, 5.03, 5.50, 5.53, 5.55, 5.56
6.x  : 6.00, 6.20, 6.50, 6.70
7.x  : 7.00, 7.50
8.x  : 8.00, 8.50
9.x  : 9.00, 9.03, 9.50
10.x : 10.00, 10.50
11.x : 11.00, 11.02, 11.50
12.x : 12.00, 12.50
13.x : 13.00, 13.02, 13.04, 13.50, 13.52

Each entry provides evf, prison0, rootvnode, sysent, and jmp offsets. Firmware outside this table will fall back to an empty offset set and the jailbreak stage will misbehave — the R/W primitive itself is firmware-agnostic.

The firmware label is derived from the user agent:

    PlayStation 4/9.00          → "9.00"
    PS4 Update (9.00)           → "9.00"

==================================================
TROUBLESHOOTING
==================================================

Symptom: Status stuck on RUNNING
Likely cause: Capture or composition task never fired
What to do: Reload from a fresh boot; the heap is likely fragmented

Symptom: HEADER-MISMATCH
Likely cause: Fake cell header was not validated
What to do: Reboot the console and try once, cleanly

Symptom: NORMAL-CLONE-MISS
Likely cause: The duplicate reference returned the original object
What to do: Heap placement missed; reboot and retry

Symptom: SSV-PLACEMENT-MISS
Likely cause: Groom did not land in the intended hole
What to do: Try a different ?n= value (e.g. 512, 1024)

Symptom: LOAD-THREW
Likely cause: Loading history.state threw a TypeError
What to do: Reboot; the session may be tainted

Symptom: SANDBOX-ESCAPE FAILED-not-found
Likely cause: ucred signature scan did not match
What to do: Add ?dbg=1 and inspect the PROC-DUMP output

Symptom: JAILBREAK-ERROR
Likely cause: Kernel write hit an unmapped address
What to do: Verify the firmware offsets match your console

Symptom: Page reloads repeatedly
Likely cause: Host/launcher wrapper is reloading it
What to do: Confirm the URL and that the build halts before runJailbreak() when ?jb=1 is absent

Symptom: Status bar stays grey
Likely cause: Exploit script threw during setup
What to do: Check the console for SETUP-THREW

GOLDEN RULE: if the exploit fails, power-cycle the console before trying again. A tainted heap will cause every subsequent attempt to fail the same way.

==================================================
FILE OVERVIEW
==================================================

<style>                → Dark console theme, status bar, toolbar, log line styling
#statusbar             → Live state indicator with PASS/FAIL/WARN chips
#toolbar               → Log export buttons and checkboxes
#consoleWrap           → Scrollable, color-coded console output
LOG object             → In-memory log store, counters, and auto-save state
classify()             → Maps exploit tags to pass / fail / warn / info
mark()                 → Central logging entry point used by all exploit stages
PS4_OFFSETS            → Per-firmware kernel offsets
RUN_JAILBREAK          → URL-param gate for the kernel stage
buildFakeHost()        → Constructs the fake JSObject with an encoded ArrayBuffer header
leakScopeObject()      → Leaks the function scope via a proxy receiver
prepareSymbolWrapper() → Installs the getter that exposes host/target pointers
loadHistoryCritical()  → Re-acquires the corrupted cell and verifies the fake header
reportComposition()    → Decides PASS / HALTED / FAIL and dispatches the jailbreak gate
read64() / write64()   → Arbitrary read/write via the retained fake array
runJailbreak()         → Kernel sysent hijack, ucred patch, root vnode swap
runAttempt()           → Single-shot entry point

==================================================
CREDITS
==================================================

mansoor0x        → Base HTML script (originally leaked; not confirmed if he intended to share it)
primehat06-wq    → Uploaded that script to GitHub — made it discoverable
OptiTronOffical  → Vibe-coded the harness, tested on console, did the daily repo-dorking to find the source

Live host: https://1350-webkit-test.vercel.app/

==================================================
DISCLAIMER
==================================================

This is not a full jailbreak. It is the userland R/W primitive that can be chained into one.

Use at your own risk. No warranty is provided, express or implied. The authors are not responsible for console damage, bans, or data loss.
