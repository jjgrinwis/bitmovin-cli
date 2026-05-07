# Bitmovin CLI Encoding Templates

Bitmovin encoding templates for VOD workflows using Akamai NetStorage as both input and output. These templates are designed to be used with the [Bitmovin CLI](https://developer.bitmovin.com/encoding/docs/bitmovin-cli) to launch encoding jobs.

## Templates

| File | Codec | DRM |
|------|-------|-----|
| `template.yaml` | H.264 (Per-Title) | Yes — CENC (Widevine + PlayReady) |

## Workflow

```
Akamai NetStorage (input)
        │
        ▼
  Bitmovin Encoder  ←── AKAMAI_NL_AMS cloud region
  ┌─────────────────────────────────┐
  │  Per-Title analysis (3-pass)    │
  │  Video: H.264                   │
  │  Audio: AAC 128 kbps / 48 kHz   │
  │  Muxing: fMP4 (4s segments)     │
  │  DRM: CENC CTR (template.yaml)  │
  └─────────────────────────────────┘
        │
        ▼
Akamai NetStorage (output)
  ├── manifest.mpd        (DASH)
  ├── manifest.m3u8       (HLS)
  ├── aac_128000/         (audio segments, unencrypted)
  ├── {width}_{bitrate}_{uuid}/  (video segments, unencrypted)
  ├── drm/audio/          (audio segments, CENC encrypted)
  └── drm/video/{width}_{bitrate}_{uuid}/  (video segments, CENC encrypted)
```

## Usage

```bash
bitmovin-cli encoding create --template template.yaml
```

## Template Details

### `template.yaml` — H.264 with DRM

**Video**
- Codec: H.264 (profile: HIGH)
- Mode: Per-Title with auto representations — Bitmovin automatically selects the optimal bitrate ladder
- Encoding: 3-pass for best quality

**Audio**
- Codec: AAC
- Bitrate: 128 kbps
- Sample rate: 48 kHz

**Packaging**
- Container: fragmented MP4 (fMP4)
- Segment length: 4 seconds
- Segment naming: `seg_%number%.m4s`
- Init segment: `init.mp4`

**DRM — CENC (CTR mode)**

Both audio and video tracks are encrypted. Two DRM systems are configured:

| System | Detail |
|--------|--------|
| Widevine | PSSH embedded in manifest |
| PlayReady | License URL via EZDRM (`playready.ezdrm.com`) |

> **Note:** FairPlay has been excluded from this template.

> **Note:** The `key` and `kid` values in the template are example/test values. Replace them with your own keys from your DRM key provider before use in production.

**Output paths (Akamai NetStorage)**
```
/789214/bitmovin/episode-1/
  ├── manifest.mpd
  ├── manifest.m3u8
  ├── aac_128000/
  ├── {width}_{bitrate}_{uuid}/
  ├── drm/audio/
  └── drm/video/{width}_{bitrate}_{uuid}/
```

---

## Configuration Reference

The following IDs are hardcoded in the templates and must match your Bitmovin account configuration:

| Field | ID | Description |
|-------|----|-------------|
| `inputId` | `2e8e3c80-11de-4ced-87d6-ef96a73581ae` | Akamai NetStorage input |
| `outputId` | `dcf019cd-3b03-4331-97ff-3ca091160df6` | Akamai NetStorage output |
| `cloudRegion` | `AKAMAI_NL_AMS` | Bitmovin encoder running on Akamai (Amsterdam) |

> **Note:** The `inputId` and `outputId` values above are examples and will not work as-is. Replace them with the actual IDs from your own Bitmovin account (found in the Bitmovin dashboard under Infrastructure), just as you need to supply your own EZDRM credentials.

To use these templates with different source files or output locations, update `inputPath` and `outputPath` accordingly.

## Prerequisites

- A Bitmovin account with API access
- Akamai NetStorage input and output configured in the Bitmovin dashboard (matching the IDs above)
- Bitmovin CLI installed and authenticated
- For DRM: valid key/KID pair and EZDRM account (or substitute your own DRM provider)
