# RcloneBrowser v2.0.2 — Fork Changelog

基於 [kapitainsky/RcloneBrowser](https://github.com/kapitainsky/RcloneBrowser) v1.8.0 的修改版本，主要目標是支援 Apple Silicon 並適配當前版本的 rclone。

## 修改總覽

### 1. Apple Silicon (arm64) 原生支援

原版僅提供 Intel (x86_64) 建構，無法在 M 系列晶片上原生運行。

- `CMakeLists.txt`：最低 CMake 版本提升至 3.5
- `src/CMakeLists.txt`：
  - `CMAKE_OSX_DEPLOYMENT_TARGET` 從 `10.9` 改為 `11.0`（Big Sur，Apple Silicon 最低要求）
  - 新增 `CMAKE_OSX_ARCHITECTURES "arm64"`
  - 移除 `-Werror` 編譯旗標，避免 Qt 5.15 deprecation 警告導致編譯失敗

### 2. 適配 rclone v1.73+ 進度輸出格式

原版的正則表達式僅支援到 rclone ~1.43 的輸出格式，導致使用新版 rclone 時進度資訊完全無法顯示。

**`src/job_widget.cpp`** — 新增三組正則表達式，保留舊版相容：

| 項目 | 舊格式 (≤1.43) | 新格式 (v1.73+) |
|------|----------------|-----------------|
| 傳輸大小 | `Transferred: 1.2G (500 MByte/s)` | `Transferred:\t4.883 MiB / 9.766 MiB, 50%, 1.234 MiB/s, ETA 3s` |
| Checks | `Checks: 10 / 20, 50%` | `Checks: 10 / 20, 50%, Listed 100` |
| 單檔進度 | `*file.bin: 50% /1.2G, 500M/s, 3s` | ` * file.bin: 50% /4.883Mi, 2.5Mi/s, 3s` |

**`src/job_options.cpp`** — 新增 `--progress` 旗標，讓 rclone 輸出每個檔案的即時進度（`Transferring:` 區段），使單檔進度條能正常運作。

### 3. Upload 模式新增目的地資料夾選擇按鈕

原版在 Upload 畫面中，目的地（Destination）只能手動輸入路徑，沒有資料夾選擇按鈕。這在用於本地端互相複製備份時非常不便。

**`src/transfer_dialog.cpp`** — 將 `buttonDest`（選擇資料夾）和 `buttonDefaultDest`（預設資料夾）改為在 Upload 和 Download 模式下都顯示，方便直接選擇本地目的地資料夾。

### 4. 修復 Qt 5.15 Deprecation 警告

**`src/main_window.cpp`**：
- `QString::split()` 改用 `Qt::SkipEmptyParts` 取代已棄用的 `QString::SkipEmptyParts`
- `QProcess::start(QString)` 改用 `QProcess::splitCommand()` + `start(program, args)` 取代已棄用的單字串重載

### 5. 修復大量小檔案上傳速度過慢的問題（v2.0.2）

v2.0.1 在 `src/job_options.cpp` 中新增了 `--progress` 旗標，原意是讓 rclone v1.73+ 的單檔進度條能正常運作。但 `--progress` 會讓 rclone 進入互動式進度輸出模式，每秒重繪所有活躍檔案的狀態，導致：

1. rclone 輸出量暴增，大量 I/O 灌入 QProcess 的 stdout
2. 密集觸發 `readyRead` 信號，每次都要跑多組正則表達式匹配
3. 頻繁建立／銷毀 UI 元件（QLabel、QProgressBar）

在上傳大量小檔案時，速度從原本的每秒十幾個檔案降至每秒約 1 個。

**修復方式：** 移除 `--progress` 旗標。`--stats 1s` 已足夠提供整體傳輸進度資訊（已傳大小、速度、ETA），不需要 `--progress` 的互動式輸出。

### 6. 版本與署名更新

- 版本號：`1.8.0` → `2.0.1` → `2.0.2`
- Copyright 年份：`2019` → `2026`
- About 對話框新增 `mod by JasonChen`

## 建構方式

```bash
# 需要 Homebrew 的 Qt5 和 CMake
brew install cmake qt@5

cd RcloneBrowser-1.8.0
mkdir build && cd build
cmake .. -DCMAKE_PREFIX_PATH=$(brew --prefix qt@5)
make -j$(sysctl -n hw.ncpu)

# 打包 .app
macdeployqt build/rclone-browser.app
codesign --force --deep --sign - build/rclone-browser.app
```

## 環境需求

- macOS 11.0 (Big Sur) 或更新版本
- Apple Silicon (M1/M2/M3/M4) 原生支援
- 已測試搭配 rclone v1.73.5
