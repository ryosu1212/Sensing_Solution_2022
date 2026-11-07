# Sensing Solution 2022

カメラで撮影した手元の映像から人差し指の位置を検出し、Spresenseのディスプレイ上にマーカーを表示するプロトタイプです。  
「ハッカソン応募書類.pdf」にまとめられているアイデア・実験内容をもとに、Spresense側の撮影とPC側のMediaPipe推論を組み合わせています。

## イラスト・システム図

プロジェクト資料の1ページ目から、イラストとシステム構成図だけを切り出して掲載しています。

### イラスト

![PDF掲載のプロジェクトイラスト](<スクリーンショット 2026-11-08 035234.png>)

### システム構成図

![PDF掲載のシステム構成図](<スクリーンショット 2026-11-08 035313.png>)

元資料は[`ハッカソン応募書類.pdf`](ハッカソン応募書類.pdf)です。

## システム構成

```text
Spresense Camera
    │
    ├─ 320 x 240 のJPEGをSDカードへ保存
    │
    ├─ USB Mass Storage経由でPCから画像を読み込み
    │
PC (Python)
    ├─ OpenCVで画像を読み込み
    ├─ MediaPipe Handsで手のランドマークを推定
    └─ 人差し指の座標・色をシリアル通信でSpresenseへ送信
    │
Spresense Display
    └─ 検出位置に赤い円を表示
```

### 使用技術

- **Spresense**: Camera、SDカード、USB Mass Storage、ILI9341ディスプレイ
- **Python**: OpenCV、MediaPipe、pyserial、Pillow、NumPy
- **Arduino**: Spresense SDK向けカメラ・SDカード・ディスプレイ制御

## リポジトリ構成

| パス | 役割 |
| --- | --- |
| [`spresense_hackathon.py`](spresense_hackathon.py) | PC側の手検出、指先座標の取得、シリアル送信 |
| [`testspre.py`](testspre.py) | PC側の初期検証用スクリプト |
| [`arduino_sketch/arduino_sketch.ino`](arduino_sketch/arduino_sketch.ino) | Spresense側の撮影・保存・表示 |
| [`train_hackathon/`](train_hackathon/) | 実験時に取得した入力画像 |
| [`connect_test.ipynb`](connect_test.ipynb) | MediaPipeのインストール確認と推論時間の検証 |
| [`performance.txt`](performance.txt) | メモリ使用量の計測結果 |
| [`ハッカソン応募書類.pdf`](ハッカソン応募書類.pdf) | プロジェクトの説明資料 |

## セットアップ

### PC側

Python 3環境で、必要なパッケージをインストールします。

```bash
pip install opencv-python mediapipe pillow numpy pyserial
```

`spresense_hackathon.py`ではシリアルポートを`COM3`として開きます。環境に合わせて、次の行を変更してください。

```python
serial.Serial("COM3", 115200)
```

### Spresense側

Arduino IDEで[`arduino_sketch/arduino_sketch.ino`](arduino_sketch/arduino_sketch.ino)を開き、次の構成を用意します。

- Spresense本体とCameraボード
- SDカード
- ILI9341互換TFTディスプレイ
- PCとのUSB接続

Arduino IDEに次のライブラリを追加してください。

- Adafruit GFX Library
- Adafruit ILI9341

## 実行方法

1. SpresenseへArduinoスケッチを書き込みます。
2. SpresenseをPCへ接続し、カメラ画像がSDカードの`input_images`へ保存されることを確認します。
3. PC側でMediaPipeの依存関係をインストールします。
4. `spresense_hackathon.py`の`COM3`と画像パスを環境に合わせます。
5. Pythonスクリプトを実行します。

```bash
python spresense_hackathon.py
```

検出できた場合は、PCから次の形式でSpresenseへ通知します。

```text
detected,<x>,<y>,<color>
```

検出できない場合は、次の文字列を送信します。

```text
not detected
```

## 処理のポイント

- MediaPipe Handsは最大1つの手を対象にします。
- 人差し指の先端に対応するランドマーク（インデックス8）の座標を取得します。
- 指先とその一つ前のランドマークの位置から画素を参照し、B/G/Rの最大値で色を判定します。
- Spresense側ではカメラ画像を`320 x 240`で保存し、検出位置に半径10の赤い円を描画します。
- PC側の画像読み込みとAI推論を分けて処理時間を計測できます。

## 計測結果

検証用ノートブックでは、1枚の画像に対するMediaPipe処理時間として約`0.33秒`が記録されています。  
`performance.txt`には、実験時のメモリ使用量として約`133 MiB`から`202 MiB`の範囲が記録されています。実行環境や画像サイズによって結果は変わるため、参考値として扱ってください。

## 注意事項

- 現在のコードは実験用プロトタイプです。COMポート、画像保存先、SDカードのマウント状態を環境に合わせて設定してください。
- Spresense側とPC側で、シリアル通信の形式を一致させる必要があります。
- カメラ画像の読み込み待ちにビジーウェイトを使用しているため、製品用途ではタイムアウトやスリープを追加することを推奨します。
