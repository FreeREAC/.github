# FreeREAC

Plug a Roland REAC stagebox into Linux, and it is ready.

FreeREAC reverse-engineered Roland's REAC — the proprietary Layer-2 protocol
(EtherType `0x8819`) Roland's digital mixers use to carry many channels of
audio to their stageboxes over one Ethernet cable — and proved the result on
real hardware: an M-200 driving REAC at 44.1 and 48 kHz, an M-5000 at 96 kHz,
and S-1608, S-4000S and S-0808 stageboxes on the other end. Head-amp control
— gain, phantom power, pad — travels on the wire itself, a Linux box can take
either the master or the slave role, and a segment can run over a tagged VLAN
trunk rather than a dedicated cable.

## The stack

Three pieces. **libreac** (the protocol: frame codec, control block, head-amp
records, box identity, master/slave state machines) and **libreac-transport**
(built from the same repository: AF_PACKET sockets, the real-time pacer that
clocks the wire, VLAN sub-interface handling) carry no opinion about audio
APIs. **reac-pw** links both and puts a stagebox on a Linux PipeWire graph as
source and sink nodes — no bridge, no second encapsulation. **reac-stageboxes**
is a GTK4/libadwaita desktop app that sets phantom power, pad and sensitivity
on every input of every box on the graph, through reac-pw's own node
parameters.

## Repositories

- **[libreac](https://github.com/FreeREAC/libreac)** — the REAC protocol and
  its transport, libreac-transport, one repository and one release.
- **[reac-pw](https://github.com/FreeREAC/reac-pw)** — the PipeWire-native
  REAC endpoint; runs as master or as slave.
- **[reac-stageboxes](https://github.com/FreeREAC/reac-stageboxes)** — the
  desktop app for the preamps.
- **[reac-protocol](https://github.com/FreeREAC/reac-protocol)** — the
  wire-format reference every repository above is verified against.

Also in the organisation: **[reac-tools](https://github.com/FreeREAC/reac-tools)**
(capture analysis — loss, reordering, jitter, head-amp record parsing) and
**[reac-captures](https://github.com/FreeREAC/reac-captures)** (a CC0
public-domain corpus of address-sanitised capture fixtures).

## Install

Fedora 44:

```
sudo dnf config-manager addrepo --from-repofile=https://freereac.github.io/rpm/freereac.repo
sudo rpm --import https://freereac.github.io/rpm/RPM-GPG-KEY-freereac
sudo dnf install reac-pw
```

## Licence

GPL-3.0-or-later, every repository.

FreeREAC is an independent interoperability project, not affiliated with,
sponsored by, or endorsed by Roland. REAC is a trademark of Roland
Corporation, used here only to identify the protocol. No Roland firmware,
binaries, symbols or decompilation listings are redistributed; everything
published is FreeREAC's own re-expression of behaviour observed on its own
equipment.

More at [freereac.github.io/freereac-www](https://freereac.github.io/freereac-www/).
