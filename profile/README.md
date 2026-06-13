# FreeREAC

**REAC Exposed Audio Communications** — open documentation and tools for Roland's
**REAC**, the audio-over-Ethernet protocol (EtherType `0x8819`) used by Roland
digital mixers and stageboxes (the V-Mixer / M-series desks and their S-series
boxes).

REAC carries many channels of low-latency digital audio as a flat Layer-2 broadcast
on a dedicated link. The vendor publishes no specification for it. FreeREAC documents
the wire format and behaviour from its own packet captures and interoperability
analysis, and ships tools to **carry**, **de-jitter**, **convert**, and **analyse**
REAC on commodity hardware (OpenWrt routers, Linux).

> Reverse engineering for interoperability is permitted, and protocol interfaces are
> not copyrightable. FreeREAC publishes its own findings, re-expressed in its own
> words and tables; it does not redistribute any vendor firmware, binaries, or
> decompilation listings. Not affiliated with or endorsed by Roland.
> *(Not legal advice.)*

## The stack

```
  reac-aes67     terminate REAC -> AES67 (RTP L24 multicast)      reac-label  channel names
       |
  reac-repacer   de-jitter a REAC stream carried over a bursty link (e.g. Wi-Fi)
       |
  reac-transport carry REAC across an OpenWrt network: VLAN trunk | gretap tunnel

  reac-protocol = the protocol reference     reac-tools = analysis / diagnostics
  reac-docs     = findings + rig recipe       reac-lab  = raw captures + journal
```

A REAC link is just Layer-2 frames. **reac-transport** puts them on a defined
segment (a tagged VLAN on one LAN, or a gretap tunnel across Wi-Fi / L3).
**reac-repacer** sits on a jittery hop and re-clocks the stream so a clock-slave
stagebox stays locked. **reac-aes67** terminates REAC into standard AES67 audio.
Each piece is independent and composes with the others.

## Repositories

**Live**

- **[reac-repacer](https://github.com/FreeREAC/reac-repacer)** — transparent
  Layer-2 de-jitter / re-pacing relay for a REAC stream over a bursty Wi-Fi link.
  OpenWrt package (apk) + LuCI app.
- **[reac-transport](https://github.com/FreeREAC/reac-transport)** — carry REAC
  across an OpenWrt network: 802.1q VLAN trunk (same LAN) or gretap tunnel (across
  Wi-Fi / L3). OpenWrt package (apk).

**Coming**

- **reac-aes67** — real-time REAC → AES67 (RTP L24) converter, transport-agnostic.
- **reac-protocol** — the REAC protocol reference: wire format, how to capture and
  analyse it yourself, and a separate firmware-findings section.
- **reac-docs** — engineering findings (the Wi-Fi-jitter investigation → the
  clocking / hardware verdict) and the rig recipe (transport + repacer + converter).
- **reac-lab** — the raw working material the docs are distilled from: packet
  captures and a working journal.
- **reac-tools** — REAC traffic analysis and diagnostics (loss / jitter / cross-mix,
  codec, probes).
- **reac-label** — Roland M-series → channel-name labeller, feeds reac-aes67.

## License

Code is GPL-3.0-or-later. The documentation is FreeREAC's own re-expression of
observed protocol behaviour.
