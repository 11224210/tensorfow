# 人工智慧期中報告  11224204林柏廷  11224210賴宣妤


## 題目:🍎 基於 TensorFlow 2 視覺物件偵測之水果辨識與定位系統

## 批量水印生成工具 

一個 簡單易用的 Python GUI 工具，用於 批量給圖片添加文字水印。支援 JPG、PNG 和 BMP 格式，並提供字體自訂與半透明效果，非常適合作品保護、品牌宣傳或批量處理圖片。
此專案由 Python 與 ChatGPT 協作開發，展示了 AI 輔助程式開發的實際應用。

### 功能特色

批量加水印:
一次處理資料夾內所有圖片，節省手動加水印的時間。

自動調整字體大小:
根據圖片寬度自動計算文字大小，最小字體限制為 20，適合各種尺寸圖片。

半透明水印:
水印文字疊加於圖片上，保持底圖細節清晰。JPG/PNG 都能正確呈現效果。

自選字體:
支援 .ttf TrueType 字體，使用者可選擇喜歡的字體樣式。

輸出管理:
處理後的圖片會自動存放於 output 子資料夾，原始圖片不會被覆蓋。

穩定處理:
自動忽略損壞或非圖片檔案，避免程式中斷。

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
