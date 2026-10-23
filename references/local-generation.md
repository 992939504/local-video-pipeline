# 本地出图 / 出视频：两个工作流

默认不装。用户明确要「本地出图」或「本地出视频」时，**就用这里的两个工作流**——别自己搭，也别换别的模型。

工作流都是 ComfyUI 的 JSON：拖进 ComfyUI 界面，或放进 `user/default/workflows/`，就能打开。

## 一、出图（ComfyUI ＋ 千问 Image 2.1 快版）

文件：

- `assets/workflows/qwen-image-t2i.json` —— 文生图
- `assets/workflows/qwen-image-edit.json` —— 垫图改图（保留构图）

模型（放 ComfyUI 对应目录，**文件名别改**——工作流按名字找文件）：

| 文件 | 放哪 |
|---|---|
| `Qwen-Image-2.1-viggle-turbo-v0.3-6step-Q5_K_M.gguf` | unet（或 diffusion_models，看你 ComfyUI 版本；要 ComfyUI-GGUF 的 `UnetLoaderGGUF` 节点） |
| `qwen3vl_8b_w4a8.safetensors` | text_encoders |
| `qwen_image_2.1_vae_bf16.safetensors` | vae |

参数与节奏：

- 竖屏 768×1344；热机 20 秒出头一张，首张 30 秒左右。
- 6 步蒸馏快版；采样 euler。

坑（真踩过）：

- **快版没有 cfg 通道，负面词完全不参与计算**——别指望"不要 XX"能压住它。
- 要出带文字的信息图，走 25 步旧引擎 ＋ cfg 3.0 ＋ 真负面词，别拿快版返工。
- **先出一张**给用户看，确认了再批量。

## 二、出视频（MiniMax H3 lite）

文件：

- `assets/workflows/h3lite-t2v.json` —— 文生视频
- `assets/workflows/h3lite-i2v.json` —— 图生视频

模型（放 ComfyUI 对应目录，**文件名别改**）：

| 文件 | 放哪 |
|---|---|
| `minimax_h3_fl2va_pruned_w4a8_mixed_ax1y2jp.safetensors` | unet（或 diffusion_models） |
| `minimax_h3_fl2v_lightx2v_turbo_4step_v0.1_comfy_resized_avg_rank_21_bf16.safetensors` | loras |
| `minimax_h3_video_vae_fp16.safetensors` | vae |
| `minimax_h3_audio_vae_fp32.safetensors` | vae |
| `qwen3vl_4b_int4_convrot.safetensors` | text_encoders |
| `mmh3-4b-ClipProj-celeb-mlp.safetensors` | text_encoders |

参数（工作流里已经写好，一般不用改）：

- 640×352 · 124 帧 @ 24fps ≈ 5.2 秒 · 4 步 · simple 调度 · res_multistep 采样
- 带音频（AAC 32kHz）
- 一条 1~2 分钟

三条硬规矩：

1. **提示词用英文**——中文基本不生效（同一个 seed，英文描述才照做；中文写了也白写）。
2. **默认只有环境音，不会主动说话**；要人声，得在提示词里写英文台词。
3. 出的是几秒的短镜头，**是素材不是成片**——拼成片子还得回主流程 ④⑤⑥。

## 装之前

- 两样都吃显卡；没显卡别碰。
- 先问用户：装什么、多大、放哪儿，别默默装。
- 视频工作流要用支持 H3 的 ComfyUI（节点 `MiniMaxH3ImageToVideo`、`ClipProjLoader`）——版本太旧会缺节点。
- 工作流认模型文件名——改了名，工作流就断。
