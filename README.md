# Video Editing Agent

A folder-based workflow for making short tech videos (Rust, AI, and more). You write and record. Claude edits through the AI Producer MCP server (`plugin:aip:ai-producer`).

## Folder layout

```
video-editing-agent/
├── README.md                  <- this file
├── STYLE.md                   <- standing editing rules (quality, zoom, captions, music, color)
├── series/rust-in-ai/         <- the Rust in AI series: theme, graphics kit, teleprompters, recording + AI Producer guides
├── tools/sync_audio.py        <- aligns a separately recorded mic track to a phone video
├── tools/key_green.py         <- removes the green screen, puts the speaker on the series studio background
├── .claude/skills/series-style <- script and editing style learned from reference channels
├── _backup/                   <- retired tools, not used (see _backup/README.md)
├── color-grading/             <- preprocessing pipeline and LUTs
│   ├── preprocess.sh          <- auto white balance + denoise + LUT + sharpen
│   ├── analyze-color.py       <- color analysis and correction suggestions
│   ├── luts/                  <- .cube LUT files (download from luttie.app etc.)
│   └── presets/               <- named FFmpeg filter chains
├── templates/
│   ├── script-template.md     <- copy for every new script
│   └── video-status.md        <- copy for every new video's STATUS.md
└── videos/
    └── 001-rust-ownership/    <- one folder per video: NNN-short-slug
        ├── STATUS.md          <- where this video is in the lifecycle
        ├── 01-script/         <- script.md (Claude writes, you read and approve)
        ├── 02-raw/            <- YOU drop footage here
        │   ├── talking-head/      camera clips (vertical): the only thing you must record
        │   └── screen-recordings/ optional, only if you want to show a real screen
        ├── 03-edit/           <- Claude's working notes: edit plan, project link, change log
        └── 04-export/         <- final mp4 and thumbnail
```

Rule of thumb: you only write in `02-raw/` (and you edit the script if you want to). Claude owns `01-script/`, `03-edit/` and `04-export/`.

## Lifecycle (every video)

| # | Stage | Who | Output |
|---|-------|-----|--------|
| 1 | **Topic** | You | One line: "video 002 about <topic>" |
| 2 | **Script** | Claude | `01-script/script.md`, teleprompter-ready, with shot cues |
| 3 | **Approve** | You | Read it aloud once, ask for changes, then say "script approved" |
| 4 | **Record** | You | Clips into `02-raw/talking-head/` and `02-raw/screen-recordings/` |
| 4.5 | **Pre-process** | Claude | Auto color correct, denoise, grade raw clips via `color-grading/preprocess.sh` |
| 5 | **Edit** | Claude | Project in AI Producer, plan and link in `03-edit/edit-plan.md` |
| 6 | **Review** | You | Watch the preview link and send change requests |
| 7 | **Export** | Claude, only when you say "export" | mp4 in `04-export/` |

`STATUS.md` inside each video folder tracks the stage. Claude updates it at each step.

## How to start a new video

1. Tell Claude: "new video: <topic>". Claude copies the templates into `videos/NNN-slug/` and writes the script.
2. Read the script, then ask for changes or approve it.
3. Record. Name files in order: `take01.mov`, `take02.mov`, `screen01.mov`.
4. Drop the files in `02-raw/` and tell Claude "footage is in 002".
5. Review the preview link, request changes, and say "export" when happy.

## Recording guide

- **Teleprompter:** each script folder has a `teleprompter.html`. Double-click it, press Space to start, then read. Arrow up/down changes the speed. Place the laptop right under the camera lens.

- **Talking head:** vertical 9:16 phone-style framing. Face in the upper-middle, with some space above the head. Good front light (a window in front of you, not behind). Keep the camera still.
- **Screen recordings:** optional. By default Claude creates the code and terminal visuals (see below), so you only read the script on camera.
- **Audio:** use your good mic for everything. If you also record screen audio, record a few seconds of silence at the start for noise reference.
- **Mistakes:** if you flub a line, pause for 2 seconds and repeat the line. Claude cuts the retake.
- **Clap or say "take two"** between takes if it helps you find them later.

## Production quality checklist (do this before every recording session)

### Lighting
- [ ] Room/ceiling lights **OFF** (they cast ugly top-down shadows)
- [ ] **Key light** at 45° from face, angled slightly down, with softbox — just outside the frame
- [ ] **Rim/hair light** behind you pointing at shoulders (separates you from background)
- [ ] **Background accent light** on (cool teal or blue for tech vibe contrast)
- [ ] White balance **locked** on camera: 5000–5600K (never auto)
- [ ] No mixed light sources (don't combine daylight window + warm tungsten lamp)

### Clothing
- [ ] **Solid jewel tones** (emerald, sapphire, burgundy) or **earth tones** (olive, charcoal, navy)
- [ ] **No** tight stripes, small patterns, or checks (causes moiré shimmer on camera)
- [ ] **No** pure white (blows out exposure) or pure black ("floating head" effect)
- [ ] **No** shiny/reflective fabrics (creates hotspots under lights)
- [ ] **No** large logos (distracts from your message)
- [ ] Textured solids OK: knit, linen, subtle corduroy adds depth without artifacts
- [ ] Check yourself on monitor for 10 seconds — look for shimmer or exposure issues

### Background
- [ ] You are **4–6 feet away from the wall** (creates depth and natural bokeh)
- [ ] **3–5 curated props** visible: acoustic panels, plant, tech gear, lamp (not clutter)
- [ ] **Color contrast**: cool background tones (teal/blue accents) vs. warm face light
- [ ] No random items, loose cables, or cardboard boxes in frame
- [ ] Asymmetric layout (rule of thirds, not dead center)
- [ ] A practical light in the background (desk lamp, LED strip) adds life to the frame
- [ ] **No green screen needed** — styled real backgrounds look more premium for tech content

### Camera settings
- [ ] Resolution: **4K** (2160p) if possible — gives editing room for crop/reframe
- [ ] Frame rate: **30fps** (or 24fps for cinematic feel)
- [ ] White balance: **manual** 5000–5600K
- [ ] Exposure: **manual** or AE locked
- [ ] If mirrorless: 50mm+ lens at f/1.4–f/2.8 for subject separation
- [ ] Good **external mic** connected (lav or shotgun, not phone mic)
- [ ] Teleprompter positioned directly under the lens

### Quick color grading pre-check
- [ ] Film a 5-second test clip and run: `python3 color-grading/analyze-color.py test_clip.mov`
- [ ] Check the report for white balance cast, exposure, and skin tone issues
- [ ] Adjust lights or camera settings if the report flags problems

## Visuals Claude creates (you record nothing but your face)

- **Animated code and terminal visuals,** built as motion graphics. For Rust, Claude runs the real compiler (`rustc` is installed here) and uses the exact error text, so the demo is accurate.
- **Diagrams and callouts,** such as ownership arrows and key-term text.
- **B-roll images** found by search, and **Motion Library templates**.

These are animated recreations, not real screen captures. If you want a genuine screen recording for some video, drop it in `02-raw/screen-recordings/` and Claude will use it.

## What Claude does with the footage

See [STYLE.md](STYLE.md) for the full rules. Short version:

1. **Pre-process** raw clips: auto white balance, denoise, color grade, sharpen (via `color-grading/preprocess.sh`)
2. **Cut** dead air, filler words, retakes
3. **Assemble** with visual variety every 3–5 seconds
4. **Add** dynamic captions, cleaned voice, quiet music, light sound effects
5. **Apply** gentle reframing only (no heavy zoom), vertical talking head with horizontal screen cut-ins fitted into the 9:16 frame

## Color grading pipeline

Raw footage is automatically preprocessed before editing using FFmpeg:

```
Auto White Balance → Denoise → Exposure/Color Correct → Cinematic LUT (60%) → Sharpen
     grayworld        hqdn3d          eq/colorbalance          lut3d           unsharp
```

For the Rust in AI series use `--preset series-standard` (no auto white balance; see [series/rust-in-ai/TOOLS-AUDIT.md](series/rust-in-ai/TOOLS-AUDIT.md)). See [color-grading/README.md](color-grading/README.md) for details. Key commands:

```bash
# Preprocess all clips in a video folder
./color-grading/preprocess.sh videos/001-rust-ownership/02-raw/talking-head/

# Analyze color before recording (to check lighting setup)
python3 color-grading/analyze-color.py test_clip.mov

# Use a specific LUT
./color-grading/preprocess.sh --lut color-grading/luts/clean-neutral.cube take01.mov
```

## Known limits

- Claude can't generate footage of you. It can't screen-record either, but it makes animated code and terminal visuals in place of recordings.
- The animated visuals are authored as HTML motion graphics in the AI Producer workspace. This is untested here, so the first edit will confirm what it can render.
- Claude can't improve resolution beyond what the camera captured. It avoids hiding that by keeping zoom low and framing clean.
- Free usage on AI Producer has a daily allowance. Long videos use more of it.
- Claude doesn't inspect the rendered video. Watch the preview and report issues.
- The color grading pipeline uses placeholder LUTs — download real `.cube` files from [luttie.app](https://luttie.app) or [filtergrade.com](https://filtergrade.com) for best results.
