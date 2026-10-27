# 人工智慧期中報告  11224204林柏廷  11224210賴宣妤


## 題目: 基於 TensorFlow 2 視覺物件偵測之水果辨識與定位系統

### 專題簡介 (Project Introduction)

本專題旨在實作一套端到端（End-to-End）的自定義物件偵測系統。透過收集特定目標影像（以蘋果 apple 為例），利用 LabelImg 進行標註，並在 Google Colab 雲端平台上運用 TensorFlow 2.x Object Detection API 與 SSD MobileNet V2 輕量化神經網路模型進行訓練。

本系統克服了新版 Python 3.13 與 TensorFlow 2.20+ 之間的版本相容性與底層套件衝突（如 tf_slim、tensorflow_io 缺失等問題），成功完成模型訓練與推論圖（SavedModel）匯出。最終產出完整可用的 AI 模型權重與配置檔，具備極高的邊緣運算與智慧物聯網（IoT）應用潛力。

### 開發環境與技術架構 (Development Environment)

本專案運行於雲端運算環境，詳細規格與技術如下：


一、雲端訓練環境 (Google Colab)
1. 硬體加速 (GPU)：NVIDIA Tesla T4 (15 GB VRAM)
2. 作業系統：Ubuntu 24.04.4 LTS (Noble Numbat)
3. 程式語言：Python 3.13
4. 核心深度學習框架： (1) TensorFlow v2.20.0 / v2.21.0
   (2) Keras / TF-Keras
   (3) TF-Slim (針對新版 TF 進行相容性修補)


二、模型架構
1. 基礎模型 (Base Model)：`SSD MobileNet V2 320x320 (COCO17 TPU-8)`
2. 資料集格式：PASCAL VOC XML ➔ CSV ➔ TensorFlow TFRecord
3. 模型輸出格式：TensorFlow SavedModel (`saved_model.pb` 與 `variables/`)

### 完整實作步驟說明(Step-by-Step Implementation)
#### 第一階段：Google Colab 雲端環境建置與依賴安裝
##### 步驟 1：確認系統與 GPU 狀態

(1) 檢查 Python 版本與 Ubuntu 系統版本資訊 

(2) 檢查 TensorFlow 版本並確認 GPU（如 Tesla T4）是否成功掛載   

##### 步驟 2：下載 TensorFlow Official Models 與編譯 Protobuf

(1) 複製官方 TensorFlow Models 專案庫 

`!git clone [https://github.com/tensorflow/models.git]`

`(https://github.com/tensorflow/models.git)`   

(2) 切換至 `/content/models/research/ 目錄並執行 Protobuf 編譯

(3) 設定系統的 `PYTHONPATH 環境變數以載入對應的模組路徑

##### 步驟 3：安裝 Object Detection API 套件

(1) 複製 `setup.py 並執行安裝指令

(2) 針對新版環境移除 `tensorflow_io 依賴限制以避免報錯

#### 第二階段：資料集準備與標註
##### 步驟 4：本機端前置作業與工具部署

![03](https://github.com/11224210/tensorfow/blob/d041bba924a4fe9ed45c09170d760da15203195d/4.png)
圖 1：為進行本機端的資料標註前置作業，於 Windows 環境下透過 pip 完成 LabelImg 標註工具及其 GUI 介面依賴套件之部署   

##### 步驟 5：執行 LabelImg 批次載入

![03](https://github.com/11224210/tensorfow/blob/d041bba924a4fe9ed45c09170d760da15203195d/2.png)
圖 2：在資料蒐集與前處理階段，於本機端執行 LabelImg 工具進行圖片路徑與標註檔的對應載入紀錄

##### 步驟 6：目標物件標註實作

![03](https://github.com/11224210/tensorfow/blob/d041bba924a4fe9ed45c09170d760da15203195d/3.png)
圖 3：本專題針對每張訓練圖片中的目標物件進行手動邊界框（Bounding Box）標註，建立高精確度的標籤（apple），作為監督式學習依據   

##### 步驟 7：專案檔案結構建置

![03](https://github.com/11224210/tensorfow/blob/d041bba924a4fe9ed45c09170d760da15203195d/1.png)

圖 4：本專案之資料集與目錄結構採用標準 PASCAL VOC 格式設計，資料與標籤分別獨立管理於 `images/` 與 `data/ 資料夾中，以利後續 TFRecord 的順利打包與轉換   

##### 步驟 8：生成 TFRecord 檔案

(1) 執行轉換程式，將 XML 標註與影像打包為訓練集 (`train.record`) 與測試集 (`test.record`)   

#### 第三階段：下載預訓練模型與系統相容性修補

##### 步驟 9：下載 SSD MobileNet V2 預訓練模型

(1) 使用 `wget 下載 `ssd_mobilenet_v2_320x320_coco17_tpu-8` 模型並進行解壓縮   

##### 步驟 10：解決新版 Python/TensorFlow 相容性衝突（重要修補）

(1) 針對 `estimator` 載入錯誤進行模組替換   

(2) 建立空殼的 `tensorflow_io 模組以繞過版本檢查 

(3) 修補 `tf_slim` 底層中過時的 `control_flow_ops.case` 與 `control_flow_ops.cond` 指令，改為指向 `tensorflow.compat.v1`  

#### 第四階段：模型訓練與匯出
##### 步驟 11：執行模型訓練設定環境變數 TF_USE_LEGACY_KERAS=1，並執行 model_main_tf2.py 開始訓練。   監控 Loss 數值，待穩定迭代或至指定步數後手動停止。   
##### 步驟 12：匯出推論模型 (Export Inference Graph)執行 exporter_main_v2.py，將訓練好的 Checkpoint 權重匯出為 SavedModel 格式。   
##### 步驟 13：打包模型供下載將 exported_model/ 與 object-detection.pbtxt 打包成 my_custom_model.zip 壓縮檔。

## 第一步:提問
![03](https://github.com/11224204lbt/Chatgpt/blob/main/1.png)
## 第二步:安裝pip install pillow
![03](https://github.com/11224204lbt/Chatgpt/blob/main/5.png)

### CHATGPT給的程式
    import os
    from tkinter import *
    from tkinter import filedialog, messagebox
    from PIL import Image, ImageDraw, ImageFont

    # === 加水印函式 ===
    def add_watermark_to_images(folder, font_path, text):
     if not folder or not os.path.exists(folder):
        messagebox.showerror("錯誤", "請選擇有效的圖片資料夾")
        return

    output_folder = os.path.join(folder, "output")
    os.makedirs(output_folder, exist_ok=True)

    # 掃描資料夾內圖片
    images = [f for f in os.listdir(folder) if f.lower().endswith(('.jpg', '.jpeg', '.png'))]
    if not images:
        messagebox.showwarning("提示", "該資料夾中沒有圖片！")
        return

    for img_name in images:
        img_path = os.path.join(folder, img_name)
        img = Image.open(img_path).convert("RGBA")

        # 建立文字圖層
        txt_layer = Image.new("RGBA", img.size, (255, 255, 255, 0))
        draw = ImageDraw.Draw(txt_layer)

        # 設定字體
        try:
            font = ImageFont.truetype(font_path, int(img.width / 20))  # 自動字體大小
        except Exception:
            messagebox.showerror("錯誤", "字體檔載入失敗！請重新選擇字體。")
            return

        text_w, text_h = draw.textsize(text, font=font)
        pos = (img.width - text_w - 20, img.height - text_h - 20)

        # 畫上水印
        draw.text(pos, text, fill=(255, 255, 255, 128), font=font)  # 半透明白色文字

        watermarked = Image.alpha_composite(img, txt_layer)
        watermarked = watermarked.convert("RGB")  # 去除透明通道以存JPG

        out_path = os.path.join(output_folder, img_name)
        watermarked.save(out_path)

    messagebox.showinfo("完成", f"已生成水印圖片！輸出於：\n{output_folder}")

    # === 瀏覽資料夾 ===
    def choose_folder():
      folder = filedialog.askdirectory(title="選擇圖片資料夾")
      folder_entry.delete(0, END)
      folder_entry.insert(0, folder)

    # === 瀏覽字體檔 ===
    def choose_font():
      font_file = filedialog.askopenfilename(
        title="選擇字體檔案 (.ttf)",
        filetypes=[("TrueType 字體", "*.ttf")]
      )
      font_entry.delete(0, END)
      font_entry.insert(0, font_file)

    # === 點擊生成 ===
    def generate():
      folder = folder_entry.get().strip()
      font_path = font_entry.get().strip()
      text = text_entry.get().strip()

      if not folder or not font_path or not text:
        messagebox.showwarning("提示", "請先選擇資料夾、字體並輸入水印內容！")
        return

    add_watermark_to_images(folder, font_path, text)

    # === Tkinter 主介面 ===
    root = Tk()
    root.title("批量水印生成工具")
    root.geometry("480x300")
    root.resizable(False, False)

    Label(root, text="圖片資料夾：").grid(row=0, column=0, padx=10, pady=10, sticky="e")
    folder_entry = Entry(root, width=40)
    folder_entry.grid(row=0, column=1)
    Button(root, text="選擇...", command=choose_folder).grid(row=0, column=2, padx=5)

    Label(root, text="字體檔案：").grid(row=1, column=0, padx=10, pady=10, sticky="e")
    font_entry = Entry(root, width=40)
    font_entry.grid(row=1, column=1)
    Button(root, text="選擇...", command=choose_font).grid(row=1, column=2, padx=5)

    Label(root, text="水印內容：").grid(row=2, column=0, padx=10, pady=10, sticky="e")
    text_entry = Entry(root, width=40)
    text_entry.grid(row=2, column=1, columnspan=2, padx=5, pady=10)

    Button(root, text="生成水印", width=20, bg="#4CAF50", fg="white", command=generate).grid(row=3, column=1, pady=20)

    root.mainloop()

## 第三步:執行程式
![03](https://github.com/11224204lbt/Chatgpt/blob/main/6.png)
### 發現讀不到資料夾裡的圖片
## 第四步:近一步詢問
![03](https://github.com/11224204lbt/Chatgpt/blob/main/7.png)
### CHATGPT給的程式

    import os
    from tkinter import *
    from tkinter import filedialog, messagebox
    from PIL import Image, ImageDraw, ImageFont

    # === 加水印函式 ===
    def add_watermark_to_images(folder, font_path, text):
        if not folder or not os.path.exists(folder):
        messagebox.showerror("錯誤", "請選擇有效的圖片資料夾")
        return

    output_folder = os.path.join(folder, "output")
    os.makedirs(output_folder, exist_ok=True)

    # 支援圖片格式
    images = [f for f in os.listdir(folder) if f.lower().endswith(('.jpg', '.jpeg', '.png', '.bmp'))]
    if not images:
        messagebox.showwarning("提示", "該資料夾中沒有支援的圖片！")
        return

    for img_name in images:
        img_path = os.path.join(folder, img_name)
        try:
            img = Image.open(img_path).convert("RGBA")
        except Exception as e:
            print(f"無法處理圖片 {img_name}: {e}")
            continue

        # 建立文字圖層
        txt_layer = Image.new("RGBA", img.size, (255, 255, 255, 0))
        draw = ImageDraw.Draw(txt_layer)

        # 設定字體大小，最小20
        try:
            font_size = max(20, int(img.width / 20))
            font = ImageFont.truetype(font_path, font_size)
        except Exception:
            messagebox.showerror("錯誤", "字體檔載入失敗！請重新選擇字體。")
            return

        # 計算文字寬高
        bbox = draw.textbbox((0, 0), text, font=font)
        text_w = bbox[2] - bbox[0]
        text_h = bbox[3] - bbox[1]
        pos = (img.width - text_w - 20, img.height - text_h - 20)

        # 畫上半透明文字
        draw.text(pos, text, fill=(255, 255, 255, 128), font=font)

        # 合併文字圖層
        watermarked = Image.alpha_composite(img, txt_layer)
        watermarked = watermarked.convert("RGB")  # 去掉透明通道

        # 儲存
        out_path = os.path.join(output_folder, img_name)
        try:
            watermarked.save(out_path)
        except Exception as e:
            print(f"無法儲存圖片 {img_name}: {e}")
            continue

    messagebox.showinfo("完成", f"已生成水印圖片！輸出於：\n{output_folder}")


    # === 瀏覽資料夾 ===
    def choose_folder():
    folder = filedialog.askdirectory(title="選擇圖片資料夾")
    folder_entry.delete(0, END)
    folder_entry.insert(0, folder)


    # === 瀏覽字體檔 ===
    def choose_font():
    font_file = filedialog.askopenfilename(
        title="選擇字體檔案 (.ttf)",
        filetypes=[("TrueType 字體", "*.ttf")]
    )
    font_entry.delete(0, END)
    font_entry.insert(0, font_file)


    # === 點擊生成 ===
    def generate():
    folder = folder_entry.get().strip()
    font_path = font_entry.get().strip()
    text = text_entry.get().strip()

    if not folder or not font_path or not text:
        messagebox.showwarning("提示", "請先選擇資料夾、字體並輸入水印內容！")
        return

    add_watermark_to_images(folder, font_path, text)


    # === Tkinter 主介面 ===
    root = Tk()
    root.title("批量水印生成工具")
    root.geometry("480x300")
    root.resizable(False, False)

    Label(root, text="圖片資料夾：").grid(row=0, column=0, padx=10, pady=10, sticky="e")
    folder_entry = Entry(root, width=40)
    folder_entry.grid(row=0, column=1)
    Button(root, text="選擇...", command=choose_folder).grid(row=0, column=2, padx=5)

    Label(root, text="字體檔案：").grid(row=1, column=0, padx=10, pady=10, sticky="e")
    font_entry = Entry(root, width=40)
    font_entry.grid(row=1, column=1)
    Button(root, text="選擇...", command=choose_font).grid(row=1, column=2, padx=5)

    Label(root, text="水印內容：").grid(row=2, column=0, padx=10, pady=10, sticky="e")
    text_entry = Entry(root, width=40)
    text_entry.grid(row=2, column=1, columnspan=2, padx=5, pady=10)

    Button(root, text="生成水印", width=20, bg="#4CAF50", fg="white", command=generate).grid(row=3, column=1, pady=20)

    root.mainloop()
### 測試成功!!
# 成果展示
### 視窗
![01](https://github.com/11224204lbt/Chatgpt/blob/main/2.png)

### 選擇圖片所在的資料夾->選擇字體的ttf檔->輸入要水印的內容->按生成水印
![01](https://github.com/11224204lbt/Chatgpt/blob/main/3.png)

### 圖片所在的資料夾會產生一個output資料夾
![01](https://github.com/11224204lbt/Chatgpt/blob/main/4.png)
## 運行結果
### 原圖
![01](https://github.com/11224204lbt/Chatgpt/blob/main/before.bmp)
### 印上水印的圖
![01](https://github.com/11224204lbt/Chatgpt/blob/main/after.bmp)

## 優缺點與更專業性
### 優點
1️⃣ 零程式碼上手

使用者只需要選擇資料夾、輸入文字或圖片水印，就可以批量處理圖片。

GUI 操作簡單直覺，不需要編程知識。

2️⃣ 可批次處理

一次性處理整個資料夾，比手動加水印快很多。

支援多種圖片格式：.jpg, .png, .bmp 等。

3️⃣ 可自訂水印

文字水印：透明度、大小、位置都可調整。

圖片水印（可升級版本）：縮放與透明度控制。

4️⃣ 可作為 GitHub 專案

易於展示、分享，結構清楚。

README 可以附範例圖片或 GIF，對新手友好。

5️⃣ Python 與 Pillow

Python 易上手、跨平台。

Pillow 是成熟影像處理庫，功能穩定。

### 缺點 / 限制
1️⃣ 性能受限

對大量高解析度圖片（如 4K 或以上）處理速度可能較慢。

Pillow 是純 Python 實作，無 GPU 加速。

2️⃣ 功能簡單

現有版本只支援基本文字水印。

進階功能如自動排版、多文字/多水印、多行文字或批量調整透明度未實作。

3️⃣ GUI 限制

使用 tkinter，介面簡單但美觀度有限。

大量圖片處理時無進度條或暫停/取消功能（可升級）。

4️⃣ 依賴字型

字型可能因系統不同而缺失（如 arial.ttf），部分文字水印可能回退到預設字型。

跨平台可能需要自行提供字型檔。

5️⃣ 安全性與穩定性

沒有完整錯誤處理與日誌功能。

當輸入資料夾含非圖片檔案或損壞圖片，程式可能中斷（目前只簡單 print 出錯誤）。
### 更專業性
加進度條與取消按鈕

支援圖片水印、透明度控制、旋轉

增加日誌與錯誤處理

美化 GUI（使用 PyQt、Tkinter ttk 或 web GUI）
