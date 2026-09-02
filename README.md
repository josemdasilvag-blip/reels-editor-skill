# reels-editor

A [Claude Code](https://claude.com/claude-code) **skill** that turns raw phone clips of
someone talking to camera into a finished vertical reel: silences cut, word-level captions
synced to the audio, a text hook, brand typography and colours applied, exported at the
source frame rate.

The point of it is that the editing craft lives in the repository rather than in the
editor's head. Hand this to an agent and it makes the same decisions every time.

## Why this is not "call Whisper and burn subtitles"

Word-level caption sync is where naive pipelines fall apart. Whisper's word onsets drift,
renderers silently add audio delay, and a cut placed a few frames early clips the first
consonant off a word. Four of the scripts here exist to attack exactly those problems:

| Script | The problem it solves |
|---|---|
| `consensus-timing.mjs` | Averages **two transcription passes over sub-blocks with shifted boundaries**, which cancels most of Whisper's onset noise. One pass is a guess; two disagreeing passes tell you where the guess is weak. |
| `retranscribe-blocks.mjs` | Re-transcribes each cut in ~8 s sub-blocks split **at silences**, recovering onsets that drift by up to a second over a long take. |
| `merge-text-timing.mjs` | Aligns the whole-cut transcript against the sub-block timings by **longest common subsequence**, keeping the better text from one and the better timing from the other. |
| `measure-av-delay.mjs` | **Cross-correlates** the master audio against the rendered output to measure the delay the renderer introduced. It must read 0.0 ms after muxing, and it is checked rather than assumed. |
| `check-cut-edges.mjs` | Verifies **by audio energy** that no cut boundary lands inside a word. Run after every cut pass. |

## Two layers, kept apart

| Layer | What it holds | Where it lives |
|---|---|---|
| **Brand** | typeface, colours, language and dialect, lexicon, CTA | `brand.json`, generated per project |
| **Craft** | silence cutting, caption sync, Instagram safe zones, export quality, verification | this skill — marked **CERRADO**, applied unchanged unless the brand owner overrides |

`setup-brand.mjs` writes both `brand.json` and the `brand.css` the composition consumes.
It picks the accent-chip text colour by contrast ratio, and **warns when the accent colour
will not read over dark video** — a warning it reports to the user rather than silently
resolving.

## Contents

```
SKILL.md                          the skill itself: criteria, steps 0→J, the 20 known traps
reference/brand-setup.md          the five brand questions and how each one is applied
reference/pipeline-detallado.md   every command, copy-paste, end to end
reference/composition-template.html   the composition; only {{PLACEHOLDERS}} get replaced
scripts/                          the eight pipeline scripts described above
```

`fonts/` ships empty on purpose — no licensed typefaces are redistributed. The setup
script either pulls a free face from the catalogue or copies in one you own.

## Requirements

ffmpeg, Node 20+, Claude Code, and a [HyperFrames](https://www.npmjs.com/package/hyperframes)
project to render into. `SKILL.md` has the one-time setup.

## Install as a skill

```bash
git clone https://github.com/josemdasilvag-blip/reels-editor.git ~/.claude/skills/reels-editor
```

Claude Code picks it up from `~/.claude/skills/`. Then ask it to edit a reel.

## Licence

MIT — see [LICENSE](LICENSE). The skill text is in Spanish (neutral/Río de la Plata
dialect), which is the language it directs the agent to write captions in by default;
`brand.json` changes that per project.

---

Built by [José Manuel Da Silva González](https://github.com/josemdasilvag-blip) ·
[josempico.com](https://josempico.com)
