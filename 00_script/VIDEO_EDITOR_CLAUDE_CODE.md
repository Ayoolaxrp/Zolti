# VIDEO EDITOR GUIDE: Claude Code edits the episode on your own computer

---------------------------------------------------------------------------------------------------
## PART A: FOR YOU (read first, 3 minutes)
---------------------------------------------------------------------------------------------------
**What changed:** before, you used the free Claude website as a guide (file VIDEO_GUIDE_FLOW_RUNWAY.md). Now you have
**Claude Pro**, which includes **Claude Code**: a Claude that works directly on the files on your computer. It can check
your clips, cut them, put them in order, add voice, music and captions, and export the finished episode for you.
You still make the pictures (Google Flow) and the video clips (Runway) by hand, exactly as before.

**One-time setup (about 15 minutes):**
1. Install **ffmpeg** (the video tool Claude uses):
   - Windows: open **PowerShell** and type `winget install Gyan.FFmpeg` then close and reopen PowerShell.
   - Mac: install Homebrew from brew.sh, then in Terminal type `brew install ffmpeg`.
   - Check: type `ffmpeg -version`. If it prints a version, it works.
2. Install **Python** (Claude uses it for small helper scripts):
   - Windows: `winget install Python.Python.3.12` (then reopen PowerShell). Mac: `brew install python`.
3. Install **Claude Code**:
   - Windows (PowerShell): `irm https://claude.ai/install.ps1 | iex`
   - Mac (Terminal): `curl -fsSL https://claude.ai/install.sh | bash`
   - If these commands changed, search "install Claude Code" on docs.claude.com and follow the official steps.
4. Put these 3 files in your project folder, inside `00_script/`:
   - `VIDEO_GUIDE_FLOW_RUNWAY.md` (the first guide)
   - your progress file (the MD file the free Claude made before it hit the limit)
   - this file `VIDEO_EDITOR_CLAUDE_CODE.md`

**Every time you work:**
1. Open PowerShell (Windows) or Terminal (Mac).
2. Go into your project folder, for example: `cd C:\Users\YOURNAME\Videos\MyEpisode`
3. Type `claude` and press Enter. The first time it asks you to log in: use the Claude Pro account.
4. First message (copy this):
   "Read 00_script/VIDEO_EDITOR_CLAUDE_CODE.md, 00_script/VIDEO_GUIDE_FLOW_RUNWAY.md and my progress file in 00_script/.
   Then tell me where I stopped and what to do next. I am a beginner."
5. When Claude wants to run a command or change a file, it asks you. Read the short reason and choose **Yes**.
   It only works inside your project folder.

---------------------------------------------------------------------------------------------------
## PART B: INSTRUCTIONS FOR CLAUDE CODE (follow these for the whole project)
---------------------------------------------------------------------------------------------------
### B1. Situation
- The user is a **beginner** making ONE animated episode, 2 to 3 minutes, from his own script.
- Pictures are made by hand in **Google Flow**, video clips by hand in **Runway** (image-to-video). You cannot generate
  them; you write prompts (using VIDEO_GUIDE_FLOW_RUNWAY.md rules) and he generates and saves the files.
- Earlier work happened in the free Claude website, which stopped at a session limit. His **progress MD file** says how far
  he got (shot list, style line, characters, places, finished files). **That file is the truth about the past.**
  Never redo finished, approved work unless he asks.
- You run on HIS computer (Windows or Mac). Work ONLY inside the project folder. Never delete his files; move rejected
  ones to `99_rejected/` instead. Ask before anything big.
- Talk in **short, simple English**. Explain what a command does in one line before you run it.

### B2. First session: understand and report (do this before anything else)
1. Read the 3 MD files fully.
2. Check the tools: run `ffmpeg -version`, `ffprobe -version` and `python --version` (or `python3`). If one is missing,
   give the exact install step from Part A and stop until it works.
3. Look at the project folder (list every file in 01_ to 06_ folders) and compare with the shot list:
   - which start pictures exist and are approved (`_OK`),
   - which clips exist and are approved,
   - which voice / music files exist.
4. Write `00_script/STATUS.md`: a table `Shot | start picture | clip | status | note`, plus "missing" and "next".
5. Tell the user in 5-8 simple lines: where he stopped, what is missing, and the next step.
If files are named differently from the guide, do NOT rename them yourself: show him the list and suggest names, and rename
only after he says yes (then update STATUS.md).

### B3. If pictures or clips are still missing
Continue exactly like VIDEO_GUIDE_FLOW_RUNWAY.md phases 5 and 6: one shot at a time, the same STYLE LINE and LOOK LINES
from his progress file, the start picture for each Runway clip, the save-as name. Save every prompt you give into
`00_script/prompts_log.md` so nothing is lost if the session ends.

### B4. Checking clips (you can look at pictures; use that)
For every new clip in `04_clips/`:
1. `ffprobe` it: length, resolution, frame rate. Note anything different from the others.
2. Extract still frames (e.g. 4 per second) into a temp folder:
   `ffmpeg -i clip.mp4 -vf fps=4,scale=640:-1 tmp/clip_%03d.jpg`
   and LOOK at them (contact sheets of 12-16 frames are fine for a first pass; open single frames at full size where
   something looks wrong: faces, hands, glasses, clothes, objects).
3. Check: same face as the character master picture, anchors present, hands OK, nothing melting or appearing from
   nowhere, the action happens once and at the right time, no text or watermark.
4. Write the verdict in STATUS.md: `OK` with the usable seconds (e.g. "use 0.4-4.2 s"), or `REDO` with the reason and a
   new prompt. Show him the 1-2 frames that prove a problem.

### B5. The edit plan (before any cutting)
Make `00_script/EDIT_PLAN.md`: one row per shot in story order:
`Shot | file | in (s) | out (s) | voice line that plays | music / sound | transition | caption`
Rules:
- Most shots 2-5 seconds; cut away before any bad second. Total length 2-3 minutes.
- The first 3 seconds must hook (strongest image + first line).
- Hard cuts by default; a short crossfade (0.3-0.5 s) only for time jumps or mood changes. No flashy transitions.
- Voice drives timing: each shot covers its line; reaction shots fill gaps.
Show him the plan in short form and get a yes before building.

### B6. Building the video (reproducible, never by hand)
Write ONE Python script `06_edit/build_edit.py` that reads EDIT_PLAN (or a small `edit_plan.json` you keep in sync) and
uses ffmpeg to:
1. Trim each clip to its in/out, scale/pad all to the SAME size and frame rate (1080x1920 for vertical or 1920x1080
   for wide; 30 fps), same pixel format (yuv420p). Save trimmed parts to `06_edit/parts/`.
2. Join the parts in order (concat demuxer for hard cuts; xfade only where the plan says crossfade).
3. Audio: voice track(s) from `05_audio/` placed at their times; music under the voice and lowered while voice plays
   (sidechain/ducking or simple volume curve, music about -18 to -22 dB under the voice); sound effects where planned.
   Normalize the final mix to about -14 LUFS (`loudnorm`) so it sounds right on YouTube/TikTok.
4. Captions: make an `.srt` (or `.ass` for nicer style) from the script lines and their times; burn them in
   (`subtitles=` filter) with a large, bold, readable font, 2 lines max, inside the safe area (not at the very bottom on
   vertical videos, where TikTok/YouTube buttons sit). Ask him if he wants captions at all.
5. Export `06_edit/exports/EPISODE_v<N>.mp4`: H.264, CRF 18-20, AAC 192k, `-movflags +faststart`.
Every change = edit the plan or the script, run it again, new version number. Never overwrite an older export.

### B7. Checking the finished export (before showing him)
- Run `ffprobe` (length, size, fps), check for black or frozen frames (`blackdetect`, `freezedetect`), check loudness.
- Extract one frame per second and look through all of them, plus every cut point (the frame before and after each cut).
- Check: story order right, no bad seconds left, captions readable and in sync, no caption covering faces, music never
  louder than voice, the ending does not cut off abruptly.
- Write the results in STATUS.md, fix what you found, rebuild, and only then tell him "v<N> is ready, please watch".

### B8. Feedback loop
He watches and tells you what he wants changed ("cut shot 7 earlier", "music too loud", "slower at the end").
Change the plan/script, rebuild a new version, check again (B7), tell him what changed in 2-3 lines.

### B9. If he asks for CapCut instead
If he prefers to do the final touch in CapCut, give him the trimmed parts in `06_edit/parts/` in order plus the voice
and music files, and a numbered list of where each goes. You can still prepare everything up to that point.

### B10. Saving progress (sessions can end)
After every work block update `00_script/STATUS.md` (what is done, what is next). If a new session starts, read the 3 MD
files + STATUS.md + EDIT_PLAN.md and continue from "next". Never rely on remembering the old conversation.
