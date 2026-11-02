# Roadmap

What each release contains is in the [changelog](../CHANGELOG.md); this page is about
what comes next.

## Planned

### Data island packets

cartouche grows from InfoFrames to the data island packets that describe or control the
stream: the General Control Packet, Audio Clock Regeneration, ACP, ISRC1 and ISRC2, gamut
metadata, and Extended Metadata Packets (EMP, with VTEM and CVTEM), plus the SPD InfoFrame
and the HDMI 1.4 vendor-specific InfoFrame (HDMI_VIC, 3D). The packet type codes follow
Linux's `include/linux/hdmi.h` and Xilinx's `v_hdmi_common/src/xv_hdmic.h`. Audio sample
packets, which carry the audio stream rather than describe it, stay out.

- **An additive `decode_packet`** returns a data island packet enum, dispatching on the
  header's type byte: from 0x80 to today's InfoFrame decode, below it to the new packet
  types. Type 0x20 keeps going to the Dynamic HDR decode until its framing is settled
  (below). `decode` stays, as InfoFrame-only.
- **Per-type framing:** only InfoFrames get the length and checksum check. An EMP carries no
  InfoFrame checksum, its HB1 holds first and last flags and its HB2 a sequence index
  (Xilinx's `v_hdmi_common/src/xv_hdmic.c`, `XV_HdmiC_GenerateCvtem`; Amlogic's
  `hdmitx21/hw/hdmi_tx_pktmgmt.c`); an EMP spans several packets, so a separate stateful
  assembler collects a sequence, as Dynamic HDR decoding already does.
- **Packet types with fewer than two public sources** for their format stay `Unknown`.

### The next breaking release

- **`Unknown` keeps all 31 bytes**, in both `InfoFramePacket::Unknown` and
  `InfoFrame::Unknown`, so decoding and encoding still round-trip. Today it drops bytes 2
  and 3, against cartouche's own rule that nothing is discarded.
- **The new SPD and HDMI 1.4 VSI variants** ship here: today those bytes decode to
  `Unknown`, and a caller matching on `Unknown` would otherwise see different variants
  without a version change telling it so.
- **Private warning storage** (`Decoded<T, W>`): the same API in every build, `alloc` adding
  methods only.

### Dynamic HDR framing

cartouche encodes Dynamic HDR as a type-0x20 InfoFrame with a per-packet checksum. If it is
carried in EMPs instead, it moves onto the EMP layer and `IntoPackets` changes. No public
source examined shows the framing yet; it is settled once two do.

### Broader test corpus

Real-world captured InfoFrame packets from hardware, providing regression coverage and
confidence for future changes to field decoding.
