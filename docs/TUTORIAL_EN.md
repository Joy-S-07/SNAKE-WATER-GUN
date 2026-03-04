# Simple Video Tutorial

This tutorial provides the shortest path to try each feature.

## 1. Start

```bash
cd simple_video_app
./start.sh
```

Browser: `http://127.0.0.1:8090/`

## 2. Create a Video (T2V)

1. In `⚙️ Generation Sequence`, select a T2V preset
2. In `⚙️ Video Settings`, adjust scene count and scene length
3. In `📜 Scenario Input`, write what you want to generate
4. If needed, click `🎨 Style Preset` to inject style hints (default scene length becomes `5s` when applied)
5. Build scene prompts in two steps: `🧠 Build Scenario` → `🤖 Generate Prompts`
6. Click `Generate Video`
7. Check results in `🎬 Output`

## 3. Create Character Video (I2V / FLF)

1. Add reference images to `📥 Image Drop` (ref1–ref3)
2. Select from `👤 Character List` or register new via `📝 Register Character`
3. Enter initial image prompt in `📝 What do you want to draw?`
4. Click `Generate Initial Image` (auto-reflected as key image)
5. In `⚙️ Generation Sequence`, select a character preset (I2V / FLF)
6. Click `Generate Video`

Notes:

- In `char_edit_i2i_flf`, initial image and character image are especially important
- If intermediate confirmation is enabled, click `CONTINUE` to proceed

## 4. Create Music (T2A)

1. Enter mood and lyric ideas in the music input area
2. Use lyric/tag suggestions if needed
3. Run music generation and check the output audio

## 5. Create Video from Music (M2V)

1. Select generated audio or uploaded audio
2. Enter a video scenario
   - If empty, a confirmation dialog appears
   - Choose either `Generate as-is` or `Input scenario`
3. Run `Generate Video`
4. Final output is saved to `output/movie`

## 6. Create Music from Video (V2M)

1. Select generated video or uploaded video
2. Set music style conditions if needed
3. Run and check the merged result
4. Final output is saved to `output/movie`

## 7. Troubleshooting

- If `ComfyUI /prompt failed` appears, check where it stopped in error details (`node_errors`)
- For reference image errors, re-check `📥 Image Drop` and selected character state
- Old `job_id` 404 errors are often resolved by regenerating
