# 人工智慧期中報告  11224204林柏廷  11224210賴宣妤


## 題目 : 基於 TensorFlow 2 視覺物件偵測之水果辨識與定位系統

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

(2) 切換至 `/content/models/research/` 目錄並執行 Protobuf 編譯

(3) 設定系統的 `PYTHONPATH` 環境變數以載入對應的模組路徑

##### 步驟 3：安裝 Object Detection API 套件

(1) 複製 `setup.py` 並執行安裝指令

(2) 針對新版環境移除 `tensorflow_io` 依賴限制以避免報錯

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

圖 4：本專案之資料集與目錄結構採用標準 PASCAL VOC 格式設計，資料與標籤分別獨立管理於 `images/` 與 `data/` 資料夾中，以利後續 TFRecord 的順利打包與轉換   

##### 步驟 8：生成 TFRecord 檔案

(1) 執行轉換程式，將 XML 標註與影像打包為訓練集 (`train.record`) 與測試集 (`test.record`)   

#### 第三階段：下載預訓練模型與系統相容性修補

##### 步驟 9：下載 SSD MobileNet V2 預訓練模型

(1) 使用 `wget` 下載 `ssd_mobilenet_v2_320x320_coco17_tpu-8` 模型並進行解壓縮   

##### 步驟 10：解決新版 Python/TensorFlow 相容性衝突（重要修補）

(1) 針對 `estimator` 載入錯誤進行模組替換   

(2) 建立空殼的 `tensorflow_io` 模組以繞過版本檢查 

(3) 修補 `tf_slim` 底層中過時的 `control_flow_ops.case` 與 `control_flow_ops.cond` 指令，改為指向 `tensorflow.compat.v1`  

#### 第四階段：模型訓練與匯出
##### 步驟 11：執行模型訓練

(1) 設定環境變數 `TF_USE_LEGACY_KERAS=1`，並執行 `model_main_tf2.py` 開始訓練   

(2) 監控 Loss 數值，待穩定迭代或至指定步數後手動停止

##### 步驟 12：匯出推論模型 (Export Inference Graph)

(1) 執行 `exporter_main_v2.py`，將訓練好的 Checkpoint 權重匯出為 SavedModel 格式

##### 步驟 13：打包模型供下載

(1) 將 `exported_model/` 與 `object-detection.pbtxt` 打包成 `my_custom_model.zip` 壓縮檔。

https://drive.google.com/file/d/1wAsSiyefoUxy1QbGUiDWow_MbpOaApey/view?usp=drive_link

### ⚠️ 遭遇問題與解決方案 (Troubleshooting & Challenges)
在本次專題開發與模型訓練過程中，我們遇到了許多由於「環境版本過新」所導致的技術挑戰：

1. TensorFlow 物件偵測 API 與新版 Python/TF 的相容性衝突

(1) 問題描述：由於官方 TensorFlow Object Detection API 年代較久遠，直接在 Python 3.13 與 TensorFlow 2.x 環境下執行時，頻繁出現底層套件依賴報錯（如 `estimator` 模組找不到、`tensorflow_io` 缺失等）

(2) 解決方案：透過撰寫自動化腳本動態替換過時的引用路徑，並建立空殼的 `tensorflow_io` 模組以成功繞過版本檢查

2. `tf_slim` 底層函式過時報錯

(1) 問題描述：訓練讀取資料時，內部 `tf_slim` 套件呼叫了已被 Google 移除的舊版 TensorFlow 指令 (`control_flow_ops.case` 與 `control_flow_ops.cond`)，導致程式崩潰

(2) 解決方案：撰寫Python 字串取代腳本，將底層套件中的舊指令強制導向至新版的相容寫法 (`tensorflow.compat.v1.case` 與 `cond`)

3. 雲端版控遠端連線授權障礙

![03](https://github.com/11224210/tensorfow/blob/d041bba924a4fe9ed45c09170d760da15203195d/5.png)

圖 5：在專案版控與程式碼遷移過程中，曾遭遇遠端 Repository 權限驗證失敗之問題（如圖所示）。透過調整 Git 遠端連線設定與公開庫配置，成功解決程式碼同步障礙   

4. 訓練過程中的 Loss 波動與過擬合風險

(1) 問題描述：在自定義小資料集上訓練時，隨著學習率的爬升，Loss 值偶爾會出現劇烈跳動甚至飆高的現象

(2) 解決方案：透過監控訓練步數（Step），在取得足夠的檢查點（Checkpoint）權重後適時中斷訓練，並透過匯出腳本成功轉出可用的 SavedModel 推論模型

### 專題心得 (Conclusion & Reflections)

透過本次專題，我們實際走過了一遍完整的人工智慧與電腦視覺開發流程。從最初的影像蒐集、LabelImg 標註，到雲端運算環境的建置與模型訓練，每一個環節都充滿了挑戰。

最讓我們深刻體會的是——「理論與實作之間總會存在環境落差」。在這次開發中，我們沒有被過時的官方教學文件或新舊版本套件的報錯擊退，反而透過一步步的除錯、分析堆疊追蹤（Traceback）、修改底層原始碼，培養了極佳的「AI 系統除錯與故障排除能力」。這不僅讓我們對 TensorFlow 2 的底層運作機制（如資料解碼、SavedModel 結構、訓練迴圈）有了更透徹的理解，也為未來將深度學習模型成功落地到實際硬體或邊緣運算裝置打下了扎實的基礎。
