# RcloneBrowser (Apple Silicon 支援版) 🚀

這是一個基於 [kapitainsky/RcloneBrowser](https://github.com/kapitainsky/RcloneBrowser) (v1.8.0) 的現代化分叉版本（Fork）。

原版的 RcloneBrowser 是一個非常優秀的 rclone GUI 圖形介面工具，但原作者2020年之後就沒更新了，隨著時間推移，原版在最新的 macOS 環境以及新版 rclone 搭配上出現了一些相容性問題。

因為mac之後就不再支援intel核心的軟體，因此我就用claude code寫出這個版本。可能會有一些bug，歡迎提出

本專案的主要目標是 **帶來 Apple Silicon (M1/M2/M3/M4) 的原生支援**，並 **適配當前最新版本 (v1.73+) 的 rclone 進度輸出格式**。

> 💡 **開發筆記**：本專案的修改與重構主要是由 JasonChen 透過 [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) 輔助完成。

## ✨ 核心亮點與修改內容 (v2.0.2)

### 🍏 1. Apple Silicon (arm64) 原生支援

原版僅提供 Intel (x86_64) 建構，在現今的 Mac 上需依賴 Rosetta 轉譯。

- 最低 CMake 版本要求提升至 3.5。
- `CMAKE_OSX_DEPLOYMENT_TARGET` 升級至 `11.0` (Big Sur)。
- 支援編譯 `arm64` 架構，讓程式在 M 系列晶片上以原生效能運行。
- 修復並移除了舊版 Qt 5.15 導致的編譯警告與錯誤。

### 📊 2. 適配最新版 rclone 進度顯示

原版的正則表達式（Regex）僅支援到 rclone `~1.43` 的輸出格式，導致使用新版 rclone 時進度條完全不動。

- 新增多組正則表達式，完美解析 `v1.73+` 的進度輸出格式（同時保持對舊版的相容）。
- 能夠準確抓取並顯示傳輸大小、檔案檢查數 (Checks) 以及單檔傳輸進度。
- **效能優化**：棄用會導致大量小檔案傳輸卡頓的 `--progress` 旗標，改用 `--stats 1s` 提供順暢的整體進度資訊，解決了 I/O 擁塞問題。

### 📁 3. 改善本地端備份體驗 (UI 優化)

- 原版在 Upload (上傳) 畫面中，目的地（Destination）只能手動輸入路徑。
- **改進**：將目的地資料夾的「選擇按鈕」同時開放於 Upload 與 Download 模式。現在要進行本地端硬碟互拷或備份時，可以直接點擊按鈕選取資料夾，大幅提升便利性。

## 🛠️ 環境需求

- **作業系統**：macOS 11.0 (Big Sur) 或更新版本
- **硬體架構**：Apple Silicon (M1/M2/M3/M4)
- **依賴工具**：建議搭配 [rclone](https://rclone.org/) v1.73.5 或更新版本使用。

## 🏗️ 如何自行編譯 (Build)

如果你想自己在 Mac 上編譯這個專案，請先確保你已經安裝了 [Homebrew](https://brew.sh/)。

1. **安裝必要的編譯依賴：**

   ```
   brew install cmake qt@5
   ```

2. **進入原始碼目錄並建立編譯資料夾：**

   ```
   # 假設你在原始碼根目錄
   mkdir build && cd build
   ```

3. **執行 CMake 與編譯：**

   ```
   cmake .. -DCMAKE_PREFIX_PATH=$(brew --prefix qt@5)
   make -j$(sysctl -n hw.ncpu)
   ```

4. **打包為 macOS 應用程式 (.app) 並簽署：**

   ```
   macdeployqt build/rclone-browser.app
   codesign --force --deep --sign - build/rclone-browser.app
   ```

   完成後，你可以在 `build` 資料夾中找到 `rclone-browser.app`，直接拖曳到「應用程式」資料夾即可使用。

## 📜 歷史版本摘要

- **v2.0.2**: 修復大量小檔案上傳時因 QProcess stdout 擁塞導致速度嚴重低落的問題。
- **v2.0.1**: 修正 rclone v1.73+ 進度條解析，新增 arm64 支援與 UI 按鈕優化。
- **v1.8.0**: 基於 [kapitainsky 的最後穩定版本](https://www.google.com/search?q=https://github.com/kapitainsky/RcloneBrowser/releases/tag/1.8.0)。

## 🤝 致謝 (Credits)

- [**mmozeiko/RcloneBrowser**](https://github.com/mmozeiko/RcloneBrowser) - 原始專案的創造者。
- [**kapitainsky/RcloneBrowser**](https://github.com/kapitainsky/RcloneBrowser) - 接手維護並加入諸多實用功能的開發者。
- 本分支由 **JasonChen** 透過 Claude Code 修改維護。

## 📄 授權條款 (License)

本專案採用與原版相同的授權協議進行開源。
