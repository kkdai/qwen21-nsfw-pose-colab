# Qwen-Image 2.1 NSFW / 3D Pose Control — Google Colab 執行版

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/kkdai/qwen21-nsfw-pose-colab/blob/main/qwen21_nsfw_pose_colab.ipynb)

本儲存庫將 [`Arekejoker/qwen21-nsfw-pose-skill`](https://github.com/Arekejoker/qwen21-nsfw-pose-skill) 完整封裝為 **Google Colab Notebook (`qwen21_nsfw_pose_colab.ipynb`)**，讓沒有本地高階顯卡（24GB VRAM）的使用者，也能在雲端一鍵部署 **ComfyUI + Qwen-Image 2.1 (`int8 convrot`) + NSFW LoRAs + 136 個 VNCCS 3D 姿勢庫**。

---

## 📌 專案用途

原專案 `qwen21-nsfw-pose-skill` 是一套經過驗證的 Qwen-Image 2.1 本地寫實風格與精確姿勢控制配方。本 Colab 專案解決了在雲端環境執行的痛點：

1. **高速直連下載**：改用 `huggingface.co` 官方節點搭配 `aria2c` 16 執行緒加速，1~2 分鐘內即可下載完 16.1 GB 底模。
2. **Civitai API 自動對接**：內建經過驗證的 Civitai Model Version ID，只需填入 `CIVITAI_API_TOKEN`（或設定於 Colab Secrets `CIVITAI_TOKEN`）即可自動下載並重新命名為腳本預期的檔名。
3. **支援完整 4 條生成路線**：
   - **路線 ① T2I（文字出圖 + NSFW LoRA）**：內建作者 3 組經典案例（Shower / Massage / Selfie）一鍵復刻與自訂提示詞表單。
   - **路線 ② Edit（角色換姿勢 / 換裝）**：保留原角色臉部與特徵，透過自然語言指令改變姿勢或衣著。
   - **路線 ③ 腳本 3D 人偶姿勢遷移**：輸入角色圖 + 無貼圖 3D 人偶渲染圖，免開瀏覽器完成姿勢替換。
   - **路線 ④ VNCCS PoseStudio（136 個 3D 姿勢庫）**：自動配置 Playwright + Headless Chromium（啟用 SwiftShader 軟體 WebGL 渲染）執行無頭 3D 人偶擷取，亦提供選用的 **Cloudflare Tunnel (`trycloudflare.com`)** 讓你在瀏覽器內手動旋轉 3D 人偶。

---

## 💻 硬體資源需求（Colab 規格建議）

| 項目 | 空間 / 容量需求 | 說明 |
| :--- | :--- | :--- |
| **硬碟空間 (Disk)** | **約 25 ~ 30 GB** | 3 個底模合計約 **16.1 GB**（Diffusion 6.8GB + Text Encoder 8.7GB + VAE 0.6GB），LoRA 約 0.6~1.6 GB，加上 ComfyUI 與節點依賴。Colab 預設 `/content` 空間（約 78~100GB）完全足夠。 |
| **推薦 GPU：Colab L4** | **24 GB VRAM / 53+ GB RAM** | **最佳性價比**：底模與文本編碼器可常駐顯存，不會發生 CPU RAM 卸載崩潰；1152×1728 (50 步) 約 **2 ~ 3.5 分鐘/張**。 |
| **高階 GPU：Colab A100** | **40~80 GB VRAM / 83+ GB RAM** | **極速體驗**：單張 50 步約 **40 ~ 90 秒**。 |
| **免費版 GPU：Colab T4** | **15 GB VRAM / 12.7 GB RAM** ⚠️ | **不建議跑高解析度**：15GB VRAM 雖可分階段載入，但免費版僅有 **12.7 GB 系統 RAM**，模型 Offload 時極易觸發記憶體不足 (OOM)。Notebook 偵測到 RAM < 20GB 時會自動加上 `--lowvram`，建議將尺寸調降至 `1024×1024`。 |

---

## 🚀 快速使用步驟

1. 點擊上方 **[Open in Colab](https://colab.research.google.com/github/kkdai/qwen21-nsfw-pose-colab/blob/main/qwen21_nsfw_pose_colab.ipynb)** 按鈕開啟 Notebook。
2. 於 Colab 選單點選 **「執行階段 (Runtime) > 變更執行階段類型 (Change runtime type)」**，硬體加速器選擇 **L4 GPU**（或 A100 GPU）。
3. 依序執行 **Cell 1 ~ Cell 4**：
   - 檢查 GPU 與記憶體資源
   - 安裝 ComfyUI、`qwen21-nsfw-pose-skill` 與 `ComfyUI_VNCCS_Utils`
   - 填入你的 [Civitai API Token](https://civitai.com/user/account) 下載底模與 LoRA
   - 背景啟動 ComfyUI、同步 136 個 3D 姿勢庫並通過 `scripts/doctor.py` 自檢
4. 依據需求執行 **Cell 5 ~ Cell 8**（路線 ① ~ ④）即可直接在 Notebook 中預覽生成結果，或執行 **Cell 9** 開啟 Cloudflare Tunnel 進入 ComfyUI 網頁介面。

---

## 📚 參考專案與檔案來源（References）

### 1. 核心配方與腳本儲存庫
* **[Arekejoker/qwen21-nsfw-pose-skill](https://github.com/Arekejoker/qwen21-nsfw-pose-skill)**（Apache-2.0 License）
  * `SKILL.md`：完整 7 要素提示詞結構、L0~L3 姿勢大膽度階梯與配方規範
  * `install/setup.md`：ComfyUI 側依賴與模型安裝指南
  * `scripts/qwen_pose_test.py`：路線 ① (T2I) 與路線 ② (Edit) 自動化生成腳本
  * `scripts/run_pose_transfer.py`：路線 ③ 3D 人偶姿勢遷移腳本
  * `scripts/run_vnccs_ui.py` & `scripts/browser_fallback.py`：路線 ④ VNCCS PoseStudio 無頭瀏覽器控制腳本
  * `scripts/doctor.py`：環境完整性唯讀自檢工具
  * `workflows/`：`qwen21_t2i.json`、`qwen21_edit.json`、`VNCCS_PoseStudio_QI2.1.json`
  * `prompts/`：作者驗證提示詞與參數（`taws_v13_shower`、`taws_v13_massage`、`thesealpacas_v2_selfie`）

### 2. ComfyUI 與 3D 姿勢控制節點
* **[comfyanonymous/ComfyUI](https://github.com/comfyanonymous/ComfyUI)**：底層節點式推論引擎
* **[AHEKOT/ComfyUI_VNCCS_Utils](https://github.com/AHEKOT/ComfyUI_VNCCS_Utils)**：提供 `VNCCS_PoseStudio` 3D 人偶骨架控制與渲染節點

### 3. 底模與 LoRA 模型檔案清單
* **Qwen-Image 2.1 底模（HuggingFace [`Comfy-Org/Qwen-Image-2.1`](https://huggingface.co/Comfy-Org/Qwen-Image-2.1)）**：
  * `diffusion_models/qwen_image_2.1_int8_convrot.safetensors` (6.8 GB)
  * `text_encoders/qwen3vl_8b_int8_convrot.safetensors` (8.7 GB)
  * `vae/qwen_image_2.1_vae_bf16.safetensors` (0.6 GB)
* **LoRA 模型**：
  * `qwen2_1-femaledVaginus-nsfw-v1_3-fine_L22-31.safetensors` (608 MB) — [Civitai Model 2986844 (v1.3b, Version ID `3388157`)](https://civitai.com/models/2986844)
  * `qwen2_1-femaledVaginus-nsfw-v1.3-fine.safetensors` (608 MB) — [Civitai Model 2986844 (v1.3, Version ID `3386403`)](https://civitai.com/models/2986844)
  * `thesealpacas-nsfw-v2.safetensors` (79.7 MB) — [Civitai Model 2958918 (v2.0, Version ID `3357315`)](https://civitai.com/models/2958918)
  * `nsfw-posture-v3.safetensors` (84 MB) — [Civitai Model 2988215 (v1.0, Version ID `3388118`)](https://civitai.com/models/2988215)
  * `qwen21-nsfw-image-edit.safetensors` (84 MB) — [HuggingFace `RunningHubAI/rh-qwen-image-2.1-aio-nsfw-lora`](https://huggingface.co/RunningHubAI/rh-qwen-image-2.1-aio-nsfw-lora)
  * `VNCCS_QI2_PoseStudioV1.1.safetensors` (159 MB) — [HuggingFace `MIUProject/VNCCS_PoseStudio_QI2.1`](https://huggingface.co/MIUProject/VNCCS_PoseStudio_QI2.1)
* **3D 姿勢庫來源（共 136 個姿勢）**：
  * [`Totemistyk/General_Poses_PoseStudio`](https://huggingface.co/Totemistyk/General_Poses_PoseStudio)
  * [`Totemistyk/Ballet_PoseStudio`](https://huggingface.co/Totemistyk/Ballet_PoseStudio)
  * [`MIUProject/VNCCS_PoseLibrary_Main`](https://huggingface.co/MIUProject/VNCCS_PoseLibrary_Main)
  * [`MIUProject/MySuperPoseRepo`](https://huggingface.co/MIUProject/MySuperPoseRepo)
  * 原專案 `pose_library/local_user_poses/` 內建之 55 個手工精選姿勢
