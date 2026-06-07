# Edge AI Final Project: XIAO ESP32S3 Person Detection

本專題使用 Seeed Studio XIAO ESP32S3 Sense 製作邊緣 AI 人員偵測與智慧照明系統。模型在 ESP32S3 板子端執行，不依賴雲端推論；偵測到人時板子上方 LED 亮起，最後一次偵測到人後維持 10 秒，若 10 秒內再次偵測到人則重新計時。

## 專題連結

- HackMD 報告：https://hackmd.io/6zAM0sc-TES9rLbPsSbslw
- GitHub 上傳包：`GitHub_upload_package.zip`

## 使用者需要安裝的軟體

如果只是使用已經燒錄好的板子，不需要重新訓練模型，也不需要安裝 PyTorch / CUDA。

| 軟體 / 套件 | 用途 | 安裝方式 |
|---|---|---|
| Python 3.10 以上 | 執行 USB 影像顯示 viewer | 安裝 Python，並勾選 Add Python to PATH |
| pyserial | 讀取 ESP32S3 COM port | `py -m pip install pyserial` |
| pillow | 顯示 ESP32 傳來的 JPEG 影像 | `py -m pip install pillow` |
| esptool | 燒錄模型 partition 到 `0x400000` | `py -m pip install esptool` |
| VS Code | 開 terminal 與執行 viewer | 建議安裝 |
| Arduino IDE 2.x | 重新上傳 `.ino` 時才需要 | 需要 ESP32 board package 與 SSCMA library |

一次安裝 Python 套件：

```bat
py -m pip install pyserial pillow esptool
```

## 如何執行即時影像 Viewer

1. 將 XIAO ESP32S3 Sense 接上 USB。
2. 關閉 Arduino Serial Monitor，避免 COM port 被占用。
3. 解壓縮 `GitHub_upload_package.zip`。
4. 進入專案資料夾：

```bat
cd edgeai-person-xiao-s3
```

5. 執行 Python viewer：

```bat
py pc_viewer\serial_person_viewer.py COM3
```

如果你的板子不是 `COM3`，請改成實際連接埠，例如：

```bat
py pc_viewer\serial_person_viewer.py COM5
```

Viewer 會顯示：

- ESP32 相機即時影像
- 紅框：通過門檻的 person box
- 黃框：模型有輸出但面積太小、被後處理過濾
- `PERSON DETECTED / NO PERSON`
- best score、raw score、valid boxes、LED 狀態

## Arduino IDE 設定

若需要重新上傳 Arduino 程式，請安裝：

- ESP32 Arduino board package
- Seeed SSCMA Micro Core library

建議 board 設定：

```text
Board: XIAO_ESP32S3
USB CDC On Boot: Enabled
CPU Frequency: 240MHz
Flash Mode: QIO 80MHz
Flash Size: 8MB
Partition Scheme: max_app_8MB
PSRAM: OPI PSRAM
USB Mode: Hardware CDC and JTAG
Upload Mode: UART0 / Hardware CDC
Port: 依照電腦顯示選擇，例如 COM3
```

Arduino sketch：

```text
arduino/sscma_person_arduino/sscma_person_arduino.ino
```

## 如何燒錄模型到 0x400000

模型 partition：

```text
models/person_local_focus100_models_partition.bin
```

手動燒錄指令：

```bat
py -m esptool ^
  --chip esp32s3 ^
  --port COM3 ^
  -b 460800 ^
  --before default_reset ^
  --after hard_reset ^
  write_flash ^
  --flash_mode dio ^
  --flash_freq 80m ^
  --flash_size detect ^
  0x400000 models\person_local_focus100_models_partition.bin
```

重點是最後一行：

```text
0x400000 models\person_local_focus100_models_partition.bin
```

`0x400000` 是模型在 ESP32S3 flash 裡的起始位址；不要改成 `0x10000`，因為 `0x10000` 是 Arduino 程式區，不是模型區。

## 最終模型設定

- 採用版本：`local_focus100`
- 模型燒錄位置：ESP32S3 flash `0x400000`
- 觸發門檻：`kTriggerThreshold = 0.75`
- 最小框面積：`kMinBoxArea = 0.35`
- LED 維持時間：`kLightHoldMs = 10000` ms
- USB viewer baud rate：`921600`

## 資料集與訓練流程

資料集包含公開 person 場景資料、GroundingDINO 自動標註 person bounding box、實際使用 XIAO ESP32S3 Sense 拍攝的人員照片，以及房間無人背景照片。

訓練流程：

1. 收集 person / background 圖片
2. 用 GroundingDINO 產生 person 標註
3. 轉成 COCO dataset
4. 使用 SSCMA / ModelAssistant 訓練 YOLOv5 tiny
5. 匯出 int8 TFLite
6. 修正 TFLite concat quantization scale
7. 打包成 ESP32S3 model partition
8. 燒錄到 `0x400000`

## 備註

GitHub 不適合直接存放大量資料集與大型訓練 log，所以 repo 放乾淨交付包與報告用樣本；完整本機資料已放在桌面的 `finnal project` 資料夾。
