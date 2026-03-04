# かんたん動画 Standalone テクニカルガイド

このドキュメントは `simple_video_app` 向けの技術情報に限定しています。

## 1. スコープ

- 対象: Standalone版 `simple_video_app`
- 非対象: distributed構成、マルチユーザー運用、本家Webアプリ全体機能

## 2. 実行構成

- サーバ: `simple_video_app/app.py`（FastAPI）
- フロント: `simple_video_app/static/index.html` + `static/js/*.js`
- 推論実行先: ComfyUI（既定 `127.0.0.1:8188`）

役割:

- フロントは同一オリジンAPIに接続
- サーバはワークフロー適用・ジョブ管理・出力配信を実施
- ComfyUIの `/prompt` `/history` `/progress` を利用

## 3. 主要ディレクトリ

- `simple_video_app/static/`: UI配信ファイル
- `simple_video_app/docs/`: Help/Guide文書
- `simple_video_app/output/`: Standalone側の成果物配置先
- `simple_video_app/data/`: 状態保存・参照画像インデックス
- `workflows/`: 使用ワークフローJSON

## 4. APIの要点（Standalone）

代表的なAPI:

- `GET /api/v1/workflows`: 利用可能ワークフロー一覧
- `POST /api/v1/generate`: 生成ジョブ投入
- `GET /api/v1/status/{job_id}`: ジョブ状態取得
- `GET /api/v1/download/{job_id}/{filename}`: 成果物ダウンロード
- `POST /api/v1/jobs/{job_id}/interrupt`: ジョブ中断
- `DELETE /api/v1/jobs/{job_id}`: ジョブ削除
- `GET /api/v1/simple-video/help/{doc_key}`: Help文書取得

## 5. ヘルプ配信方式

`SIMPLE_VIDEO_HELP_DOCS` に `doc_key -> ファイル` を定義し、
`/api/v1/simple-video/help/{doc_key}` でMarkdownを返却します。

現在のキー:

- `tutorial` → `HELP_JP.md`
- `guide` → `USAGE_JP.md`
- `technical` → `TECHNICAL_JP.md`

フロント側は `static/js/bootstrap.js` のフローティングパネルで表示し、
Help内部リンク（`/api/v1/simple-video/help/...`）は同パネル内で遷移します。

## 6. ワークフロー固定方針

`static/js/simple_video_config.js` で主要ワークフローを固定し、
Standalone利用時の挙動を安定化しています。

例:

- T2I: `qwen_t2i_2512_lightning4`
- I2I: `qwen_i2i_2511_bf16_lightning4`
- T2V: `qwen22_t2v_4step`
- I2V: `wan22_i2v_lightning`
- FLF: `wan22_smooth_first2last`
- 背景削除: `remove_bg_v1_0`（背景削除ON時の前処理）

## 7. エラーハンドリング要点

- `/prompt` 失敗時はレスポンス本文を付加して原因追跡しやすくする
- `node_errors` を含むComfyUIエラーをそのまま確認可能
- 出力探索は複数候補ディレクトリを順に探索
- 古い `job_id` でも実ファイルがあれば後方互換的に配信を試行

## 8. 既知の制約

- single-user前提
- distributed非対応
- Utility機能はStandaloneでは未対応
- ComfyUI側に必要workflowが存在しない場合は実行不可

## 9. 変更時のチェックポイント

1. `app.py` のAPI仕様とフロント呼び出し整合
2. Help文書キーの追加時は `SIMPLE_VIDEO_HELP_DOCS` を更新
3. ワークフローID変更時は `simple_video_config.js` とUI側分岐を同時更新
4. 出力配信パス変更時は `download` の探索ロジックも確認

## 10. 必須モデル（正式ファイル名）

以下は Standalone 既定 workflow で参照されるモデル名です。
ファイル名不一致時はモデルロードに失敗します。

### 10.1 Qwen Image 系（T2I / I2I）

```
共通
├── CLIPLoader:      text_encoders/  qwen_2.5_vl_7b_fp8_scaled.safetensors
└── VAELoader:       vae/            qwen_image_vae.safetensors

2512 系（T2I / I2I 画像生成）
└── UNETLoader:      diffusion_models/  qwen_image_2512_fp8_e4m3fn.safetensors        ← ベース
    └── LoraLoader:  loras/             Qwen-Image-Lightning-4steps-V1.0.safetensors ← 4step 高速化

2511 系（I2I 画像編集 / キャラ合成）
└── UNETLoader:      diffusion_models/  qwen_image_edit_2511_bf16.safetensors                    ← ベース
    └── LoraLoader:  loras/             Qwen-Image-Edit-2511-Lightning-4steps-V1.0-bf16.safetensors ← 4step 高速化
```

- 2512 と 2511 の LoRA は各ベース専用（互換性なし）
- `--image-model 2511` 運用時は 2512 系の 2 ファイルが不要（→ 10.4 参照）

### 10.2 Wan2.2 系（T2V / I2V / FLF）

Wan2.2 は 2 段階ノイズ除去のため、高ノイズ用と低ノイズ用のモデルを常にペアでロードします。

```
共通
├── VAELoader:       vae/             wan_2.1_vae.safetensors
└── CLIPLoader:      text_encoders/   umt5_xxl_fp8_e4m3fn_scaled.safetensors
                                      （I2V のみ umt5_xxl_fp16.safetensors も使用）

T2V（テキスト→動画）
├── 高ノイズ段（step 0→2）
│   └── LoaderGGUF:  diffusion_models/  wan2.2_t2v_high_noise_14B_Q4_K_M.gguf                    ← ベース
│       └── LoraLoader:  loras/         wan2.2_t2v_lightx2v_4steps_lora_v1.1_high_noise.safetensors ← 4step
└── 低ノイズ段（step 2→4）
    └── LoaderGGUF:  diffusion_models/  wan2.2_t2v_low_noise_14B_Q5_K_M.gguf                     ← ベース
        └── LoraLoader:  loras/         wan2.2_t2v_lightx2v_4steps_lora_v1.1_low_noise.safetensors  ← 4step

I2V / FLF（画像→動画 / 先頭末尾フレーム補間）  ※Seko-V1 LoRA 使用
├── 高ノイズ段（step 0→2）
│   └── UNETLoader:  diffusion_models/  wan2.2_i2v_high_noise_14B_fp8_scaled.safetensors   ← ベース
│       └── LoraLoader:  loras/         high_noise_model.safetensors                       ← Seko-V1 LoRA
└── 低ノイズ段（step 2→4）
    └── UNETLoader:  diffusion_models/  wan2.2_i2v_low_noise_14B_fp8_scaled.safetensors    ← ベース
        └── LoraLoader:  loras/         low_noise_model.safetensors                        ← Seko-V1 LoRA
```

- T2V と I2V の UNET / LoRA は互換性なし（別モデル）
- FLF は I2V と同じモデルスタックを共有
- `high_noise_model` / `low_noise_model` は Seko-V1 LoRA のワークフロー内参照名。入手した LoRA をこのファイル名にリネームまたはシンボリックリンクして `loras/` に配置すること

### 10.3 ACE-Step（T2A）

- `ace_step_1.5_turbo_aio.safetensors`

### 10.4 モデル代替（起動オプションで切り替え）

`qwen_image_2512_fp8_e4m3fn.safetensors` が入手できない／VRAMに収まらない場合、
起動時に `--image-model 2511` を指定するだけで 2511 系のみの運用に切り替わります。

```bash
# 既定（2512 使用）
./start.sh

# 2511 のみで運用
./start.sh --image-model 2511
```

内部動作:
- `start.sh` が環境変数 `SIMPLE_VIDEO_IMAGE_MODEL=2511` を設定
- `app.py` が `/js/simple_video_config.js` リクエストを `simple_video_config_2511.js` に差し替え
- フロント側は通常どおり設定を読み込む（変更不要）

2511 運用時に**不要**になるモデル:
- `qwen_image_2512_fp8_e4m3fn.safetensors`
- `Qwen-Image-Lightning-4steps-V1.0.safetensors`

2511 運用時に**必須**のモデル:
- `qwen_image_edit_2511_bf16.safetensors`
- `Qwen-Image-Edit-2511-Lightning-4steps-V1.0-bf16.safetensors`

> **注意:** 2511 は Edit（指示編集）モデルのため、プロンプトは内部で
> Edit指示形式に自動変換されます。純粋な T2I と比べ生成傾向が異なる場合があります。

手動で切り替えたい場合は `simple_video_config.js` を直接編集することも可能です。
設定項目の `initialImage` と `t2i` を `'qwen_i2i_2511_bf16_lightning4'` に変更してください。

### 10.5 運用補足

- workflow定義上のファイル名と実ファイル名を一致させること
- 名前が異なる配布物は、workflow JSON 側のモデル指定名を実環境に合わせること

### 10.6 特殊カスタムノード確認（既定 workflow）

既定 workflow では、以下の特殊ノードを使用します（ComfyUI標準外を含む）。

- Qwen系: `CFGNorm`, `TextEncodeQwenImageEditPlus`, `FluxKontextMultiReferenceLatentMethod`
- Wan系: `LoaderGGUF`, `WanImageToVideo`, `WanFirstLastFrameToVideo`, `CreateVideo`, `SaveVideo`
- ACE-Step系: `TextEncodeAceStepAudio1.5`, `EmptyAceStep1.5LatentAudio`, `VAEDecodeAudio`, `SaveAudioMP3`
- 背景削除系: `LayerMask: RemBgUltra`

具体的な導入先:

- ComfyUI本体（最新版）に含まれるノード
    - `CFGNorm`, `TextEncodeQwenImageEditPlus`, `FluxKontextMultiReferenceLatentMethod`
    - `WanImageToVideo`, `WanFirstLastFrameToVideo`, `CreateVideo`, `SaveVideo`
    - `TextEncodeAceStepAudio1.5`, `EmptyAceStep1.5LatentAudio`, `VAEDecodeAudio`, `SaveAudioMP3`
- 追加導入が必要なノード
    - `LoaderGGUF` → `calcuis/gguf`（https://github.com/calcuis/gguf）
    - `LayerMask: RemBgUltra` → ComfyUI-Manager で `RemBgUltra` または `LayerMask` を検索して導入

※ `LoaderGGUF` が見つからない場合は、まず `calcuis/gguf` を導入してください。

ロード状況の確認例（ComfyUI起動中）:

```bash
python3 - <<'PY'
import json, urllib.request
url='http://127.0.0.1:8188/object_info'
required=[
    'CFGNorm','TextEncodeQwenImageEditPlus','FluxKontextMultiReferenceLatentMethod',
    'LoaderGGUF','WanImageToVideo','WanFirstLastFrameToVideo','CreateVideo','SaveVideo',
    'TextEncodeAceStepAudio1.5','EmptyAceStep1.5LatentAudio','VAEDecodeAudio','SaveAudioMP3',
    'LayerMask: RemBgUltra'
]
with urllib.request.urlopen(url, timeout=8) as r:
    obj=json.loads(r.read().decode('utf-8'))
missing=[name for name in required if name not in obj]
print('missing:', missing if missing else 'none')
PY
```

### 10.7 ダウンロード先候補（検索リンク）

モデルは公開場所・ファイル名が更新されることがあるため、以下の検索リンクから取得し、workflow 記載名と一致させて配置してください。

- Qwen Image 2512 base: https://huggingface.co/models?search=qwen_image_2512_fp8_e4m3fn
- Qwen Image Edit 2511 base: https://huggingface.co/models?search=qwen_image_edit_2511_bf16
- Qwen Image Lightning LoRA: https://huggingface.co/models?search=Qwen-Image-Lightning-4steps
- Qwen Image Edit Lightning LoRA: https://huggingface.co/models?search=Qwen-Image-Edit-2511-Lightning-4steps
- Wan2.2 I2V base（high/low noise）: https://huggingface.co/models?search=wan2.2_i2v_high_noise_14B_fp8_scaled
- Wan2.2 T2V GGUF base（high noise）: https://huggingface.co/models?search=wan2.2_t2v_high_noise_14B_Q4_K_M.gguf
- Wan2.2 T2V GGUF base（low noise）: https://huggingface.co/models?search=wan2.2_t2v_low_noise_14B_Q5_K_M.gguf
- Wan2.2 LightX2V LoRA: https://huggingface.co/models?search=wan2.2_lightx2v_4steps_lora
- ACE-Step 1.5 turbo: https://huggingface.co/models?search=ace_step_1.5_turbo_aio

確認ポイント:

- ダウンロード後、workflow JSON の `unet_name` / `lora_name` / `clip_name` / `vae_name` / `ckpt_name` と実ファイル名を一致させる
- 一致しない場合は、(a) 実ファイルをリネーム、または (b) workflow JSON の指定名を実ファイル名に変更
- `high_noise_model.safetensors` / `low_noise_model.safetensors` は I2V(Seko) workflow の LoRA 参照名なので、同名で `ComfyUI/models/loras/` に置く

### 10.8 初回導入チェック（初心者向け）

ComfyUI を初回導入する場合:

- 公式: https://github.com/comfyanonymous/ComfyUI
- インストール手順は公式 README の Install セクションを参照

最短確認フロー（1本化）:

1. ComfyUI health check

```bash
curl -s http://127.0.0.1:8188/system_stats | head
```

2. simple_video_app 起動

```bash
./start.sh --no-reload
```

3. アプリ health check

```bash
curl -s http://127.0.0.1:8090/api/v1/workflows | head
```

4. 1ジョブ実行（最小テスト: T2I）

```bash
JOB_ID=$(curl -s -X POST http://127.0.0.1:8090/api/v1/generate \
    -H 'Content-Type: application/json' \
    -d '{"workflow":"qwen_t2i_2512_lightning4","prompt":"a simple landscape"}' \
    | python3 -c 'import sys,json; print(json.load(sys.stdin).get("job_id",""))')
echo "job_id=$JOB_ID"
curl -s "http://127.0.0.1:8090/api/v1/status/$JOB_ID"
```

GPU目安（環境差あり）:

- 画像中心（T2I/I2I）: 12GB 以上推奨
- 動画中心（Wan2.2 T2V/I2V）: 16GB 以上推奨
- 快適運用: 24GB 以上推奨

### 10.9 最終検証（ComfyUI GUIワークフロー）

モデル/ノードの導入確認は、ComfyUI GUIで対応ワークフローを直接開いて実行する方法が最も確実です。

本リポジトリには確認用GUIワークフローを同梱しています:

- `workflows/gui_validation/image_qwen_Image_2512.json`（T2I）
- `workflows/gui_validation/qwen_image_edit_2511.json`（I2I Edit）
- `workflows/gui_validation/video_wan2_2_14B_t2v_RTX3060_v1_linux.json`（T2V GGUF）
- `workflows/gui_validation/Wan2.2-I2V-A14B-4steps-lora-rank64-Seko-V1-NativeComfy_linux.json`（I2V）
- `workflows/gui_validation/video_wan2_2_14B_flf2v_s_linux.json`（FLF）
- `workflows/gui_validation/ace-step-v1-t2a_linux.json`（T2A）

検証手順:

1. ComfyUI GUIで workflow JSON を読み込み
2. 各 workflow を 1 回実行
3. `Node class not found` / `Cannot import` / `model not found` が出ないことを確認

エラーが出た場合は、不足したノード名/モデル名をそのまま控えて、README の「AIに聞くテンプレート」に貼り付けてください。
