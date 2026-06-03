# Edge AI Final Project: XIAO ESP32S3 Person Detection

本專題使用 Seeed Studio XIAO ESP32S3 Sense 製作邊緣 AI 人員偵測與智慧照明系統。模型在 ESP32S3 板子端執行，不依賴雲端推論；偵測到人時板子上方 LED 亮起，最後一次偵測到人後維持 10 秒，若 10 秒內再次偵測到人則重新計時。

## 專題連結

- HackMD 報告：https://hackmd.io/6zAM0sc-TES9rLbPsSbslw
- GitHub 上傳包：`GitHub_upload_package.zip`

## 主要內容

`GitHub_upload_package.zip` 內含：

- Arduino ESP32S3 程式
- 最終 TFLite 模型與 ESP32S3 model partition
- 訓練 config
- HackMD 文件與報告圖片
- 不用 Wi-Fi 的 USB VS Code viewer
- 資料集圖片範例

完整資料集與大型訓練 log 已整理在本機：

```text
C:\Users\Rui\Desktop\NTUST\edge ai\finnal project
```

## 最終模型設定

- 採用版本：`local_focus100`
- 模型燒錄位置：ESP32S3 flash `0x400000`
- 觸發門檻：`kTriggerThreshold = 0.75`
- 最小框面積：`kMinBoxArea = 0.35`
- LED 維持時間：`kLightHoldMs = 10000` ms
- USB viewer baud rate：`921600`

## 使用方式

### 1. 燒錄 Arduino 程式

使用 Arduino IDE 開啟：

```text
arduino/sscma_person_arduino/sscma_person_arduino.ino
```

板子選擇 XIAO ESP32S3，Port 選擇對應 COM port 後上傳。

### 2. 燒錄模型

模型 partition：

```text
models/person_local_focus100_models_partition.bin
```

燒錄位址：

```text
0x400000
```

### 3. 不用 Wi-Fi 顯示畫面

電腦安裝 Python 套件：

```bat
py -m pip install pyserial pillow
```

執行 USB viewer：

```bat
py pc_viewer\serial_person_viewer.py COM3
```

注意：Arduino Serial Monitor 要先關掉，因為 COM port 同一時間只能被一個程式使用。

## 資料集與訓練流程

資料集包含 Kaggle/person 場景資料、GroundingDINO 自動標註 person bounding box、實際使用 XIAO ESP32S3 Sense 拍攝的人員照片，以及房間無人背景照片。

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
