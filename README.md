# ai-tutorial-workflow

**English** · [简体中文](README.zh-CN.md)

A template for making software tutorial videos with AI coding agents such as Claude Code or Codex and [Remotion](https://www.remotion.dev/).

The repository includes a working demo and a reusable production workflow. It does not include the original tutorial's product assets, account details, or private conversations. Companion article (Chinese): [Making Video Tutorials with Claude Code and Codex: A Production Retrospective](https://blog.cosine.ren/post/ai-assisted-tutorial-workflow).

All scripts, feedback, IDs, and prompts are generic examples. The demo's episode count, duration, and configuration do not describe the original tutorial.

## What's included

| Directory | Contents |
| --- | --- |
| [`video/`](video/) | A runnable Remotion project with two demo episodes: line-by-line scripts, Edge TTS and caching, an audio-driven timeline, aligned captions, camera moves and spotlights, loudness normalization, delivery checks, local and Feishu transcripts, Bilibili publishing copy, and prompter scripts |
| [`tts-lab/audition/`](tts-lab/audition/) | A TTS comparison page for listening to the same sentences across engines, with normalized loudness and an option to export a single HTML file with embedded audio |
| [`prompts/`](prompts/) | Prompts for starting a project, revising footage, reviewing transcripts, working on narration, handing off work, and publishing |
| [`templates/`](templates/) | Templates for `GOAL.md`, `HANDOFF.md`, and `COORDINATION.md` |
| [`docs/`](docs/) | Workflow notes (Chinese): [production workflow](docs/workflow.md), [transcripts](docs/transcript.md), [pronunciation](docs/pronunciation.md), [TTS selection](docs/tts-selection.md), [render performance](docs/render-performance.md), [multi-agent coordination](docs/multi-agent.md), and [publishing](docs/publishing.md) |

The supporting docs, prompts, and templates are currently in Chinese. This README covers setup and customization in English.

## How it works

**The script is the source of truth. Everything else is derived from it.**

```text
video/src/script/ep*.ts ──► pnpm tts ──► tts-manifest.json
        │                              (line durations + word timestamps)
        └──────────────► timeline.ts ◄─────────┘
                              │
                              ├──► Remotion episodes
                              ├──► Caption chunks
                              ├──► Header and chapter progress
                              ├──► Publishing chapter timestamps
                              └──► Transcript timecodes
```

- Each line has an `id` that is unique across the series. `text` is the caption; `tts` provides a different spoken form when needed.
- Narration duration determines scene length. Change a line and the following timecodes update automatically.
- Scene components use `useScene()` to get each line's start and end frames. `cueTextFrame()` finds the frame when a word is spoken, so visuals can follow the narration.

## Quick start

You need Node 22+, pnpm 10, ffmpeg 5.1+ (`-fps_mode` is used for transcript screenshots; the audition tool also needs libmp3lame), [uv](https://docs.astral.sh/uv/) to run edge-tts, and ImageMagick 7's `magick` for transcript contact sheets. The project uses system fonts: macOS includes PingFang; install Noto Sans SC for consistent output across machines. Remotion downloads Chrome Headless Shell on the first render.

```sh
git clone https://github.com/yusixian/ai-tutorial-workflow.git
cd ai-tutorial-workflow/video
pnpm install                   # Install dependencies and generate demo sound effects/music
pnpm typecheck && pnpm test    # Check types; test captions and the timeline
pnpm studio                    # Preview in Remotion Studio
pnpm timing ep1                # Print scene and line timecodes
pnpm tts                       # Generate narration with Edge TTS; sends demo text to Microsoft
pnpm delivery-check --timing-only
pnpm render --concurrency 4    # Render all episodes to out/raw/<ep>.mp4; or select ep1 ep2
pnpm finalize                  # Two-pass loudnorm to -15 LUFS; writes out/<ep>.mp4
pnpm compilation               # Join episodes in order into out/compilation.mp4
pnpm delivery-check            # Check frames, encoding, color, audio/video duration, decoding
pnpm covers                    # Write covers to out/covers/
pnpm transcript                # Write transcripts and line screenshots to transcript/
pnpm publish-kit               # Write publishing copy to publish/
pnpm prompter                  # Write prompter scripts to voice/; --pdf uses local Chrome
pnpm transcript:lark push      # Optional: push a Feishu transcript; sign in to lark-cli first
```

You can preview and render before running `pnpm tts`. Lines without narration are silent and timed by estimated reading speed. Before rendering, the script checks audio files registered in the manifests and tells you which command to run if any are missing. It also warns about lines without narration. Narration audio is not committed to Git; a fresh clone starts with an empty `{}` narration manifest.

To try the audition tool, run these commands from the repository root:

```sh
cd tts-lab/audition
python3 demo_clips.py          # Generate two demo "engines" using Edge TTS at different speeds
python3 build_page.py --pack   # Write page/index.html and a standalone HTML with embedded audio
```

To compare your own engines, copy `audition.example.json` to `audition.json`. Set the sentences, listening notes, and audio directories, place each engine's `<sentence-id>.wav` files in its directory, then build the page.

## Use your own content

1. `video/src/config.ts`: set the series name, version label, language, and Feishu transcript title.
2. `video/src/script/`: use `ep1.ts` as a starting point for your episodes and register them in `episodes.ts`.
3. `video/src/scenes/`: create one component per scene and register it by scene ID in the episode's `index.ts`. The transcript exporter reads that registry to identify the component and source file for each scene. Add the registry to `src/episode/entries.ts` for Remotion compositions. For UI demos, follow `scenes/ep1/app.tsx`: build a mock interface in JSX and define control coordinates as constants for camera moves, cursors, and spotlights.
4. `video/scripts/spoken-text.ts`: add pronunciation rules for ambiguous words and abbreviations.
5. `video/scripts/kits.ts`: replace publishing titles, descriptions, tags, and links. The example uses `example.com` placeholders; `pnpm publish-kit` warns about any that remain.
6. `video/src/script/music.ts`: set music files and credits. The demo music is generated in code. Put licensed tracks in `public/audio/bgm/`, which is ignored by Git because many music libraries do not allow redistribution.
7. Start with [`prompts/01-kickoff.md`](prompts/01-kickoff.md) and ask your agent to write the requirements and `GOAL.md` first.

## License

The code and documentation in this repository are licensed under [MIT](LICENSE).

Remotion has a separate license. Individuals (including commercial use), for-profit organizations with up to three people, and nonprofits can use it for free. The headcount applies to the whole organization; a three-person team within a larger company does not qualify on its own. Other for-profit organizations need a Company License. See the [Remotion license FAQ](https://www.remotion.dev/docs/license/faq) for the full eligibility rules.

The demo includes no third-party assets; its sound effects and music are generated in code. Images, music, fonts, and voices you add remain subject to their own licenses. Check the terms for your voice and TTS provider as well; see [TTS selection](docs/tts-selection.md#授权和标注) (Chinese).
