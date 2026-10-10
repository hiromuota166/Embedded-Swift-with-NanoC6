# Embedded Swift with M5Stack NanoC6

M5Stack NanoC6（ESP32-C6）の青色LEDを、Embedded Swiftで点滅させるサンプルです。

Apple公式のサンプル [esp32-led-blink-sdk](https://github.com/swiftlang/swift-embedded-examples/tree/main/esp32-led-blink-sdk) を、NanoC6用に書き換えています（LEDのピンを GPIO8 → GPIO7 に変更）。

## 必要なもの

- Mac（Apple Silicon）
- M5Stack NanoC6
- データ通信ができるUSB-Cケーブル

## 動作確認した環境

| | バージョン |
|---|---|
| macOS | 26.4.1 |
| Swift | 6.4.0（swiftlyでインストール） |
| ESP-IDF | v5.4.4 |
| espressif/idf_swift | 1.0.1 |

## セットアップ（最初の1回だけ）

### 1. Swift 6.4をインストールする

XcodeやCommand Line Toolsに入っているSwiftには、RISC-V（ESP32-C6のCPU）用の部品がないので使えません。swiftlyで別に入れます。

```bash
curl -O https://download.swift.org/swiftly/darwin/swiftly.pkg && installer -pkg swiftly.pkg -target CurrentUserHomeDirectory && ~/.swiftly/bin/swiftly init --quiet-shell-followup && . "${SWIFTLY_HOME_DIR:-$HOME/.swiftly}/env.sh" && hash -r
```

```bash
swiftly install 6.4.0
```

普段使う `swift` コマンドをXcode／Command Line Toolsのものに戻しておきます（組み込み用のSwiftは、あとで `swiftly run ... +6.4.0` で呼び出します）。

```bash
swiftly unlink
```

`Apple Swift version 6.4` と表示されればOKです。

```bash
swiftly run swiftc --version +6.4.0
```

### 2. ビルドに使うツールを入れる

```bash
brew install cmake ninja dfu-util
```

### 3. ESP-IDF v5.4.4を入れる

このサンプルが使う部品 `espressif/idf_swift` は、ESP-IDF 5.4以上が必要です。

```bash
mkdir -p ~/esp && cd ~/esp && git clone -b v5.4.4 --recursive --depth 1 --shallow-submodules https://github.com/espressif/esp-idf.git
```

```bash
cd ~/esp/esp-idf && ./install.sh esp32c6
```

### 4. `get_idf` コマンドを登録する（なくても大丈夫）

`idf.py` を使えるようにする設定を、`get_idf` と打つだけで読み込めるようにします。

```bash
echo "alias get_idf='. \$HOME/esp/esp-idf/export.sh'" >> ~/.zshrc && source ~/.zshrc
```

### 5. このリポジトリをクローンする

```bash
git clone https://github.com/hiromuota166/Embedded-Swift-with-NanoC6.git ~/esp/Embedded-Swift-with-NanoC6
```

## ビルドと書き込み

1. このリポジトリのフォルダに移動する
   ```bash
   cd ~/esp/Embedded-Swift-with-NanoC6
   ```
2. ESP-IDFを読み込む（**ターミナルを開くたびに毎回必要です**）
   ```bash
   get_idf
   ```
   `get_idf` を登録していない場合は `. ~/esp/esp-idf/export.sh` を実行してください。
3. ビルドする（クローンした直後の1回目は、必ず `swiftly run` を付けます）
   ```bash
   swiftly run idf.py build +6.4.0
   ```
4. NanoC6の**ボタンを押しながら**USBケーブルを挿す（書き込みモードで起動します）
5. 書き込む
   ```bash
   idf.py flash
   ```
   `Done` と表示されたら、USBケーブルを抜いてもう一度挿してください。青いLEDが点滅します。

**2回目以降：** コードを書き換えたら、手順4と5だけで大丈夫です。`idf.py flash` がビルドもしてくれます。

ボードの種類（`esp32c6`）は `sdkconfig` に保存済みなので、`idf.py set-target` は要りません。

## うまくいかないとき

| 症状 | 原因と対処 |
|---|---|
| `idf.py: command not found` | そのターミナルでESP-IDFを読み込んでいません。`get_idf` を実行してください |
| `unable to load standard library for target 'riscv32-none-none-eabi'` | Xcode／Command Line ToolsのSwiftでビルドされています。`idf.py fullclean` のあと、`swiftly run idf.py build +6.4.0` を実行してください |
| `TOOLCHAINS=...` でSwiftが切り替わらない | Xcode本体が選ばれていない環境（Command Line Toolsだけの環境）では、`TOOLCHAINS` は無視されます。代わりに `swiftly run ... +6.4.0` を使ってください |
| 書き込みに失敗する／ポートが見つからない | ボタンを押しながらUSBケーブルを挿し直してから、`idf.py flash` を実行してください |

`idf.py set-target` や `idf.py fullclean` を実行したあとの最初のビルドも、`swiftly run idf.py build +6.4.0` で行ってください。最初のビルドのときに、どのSwiftを使うかが記録されるためです。

## ファイル構成

| ファイル | 役割 |
|---|---|
| `main/Main.swift` | 起動時に呼ばれる関数（`app_main`）。LEDのピン番号と点滅の間隔（`blinkDelayMs`）はここで変えます |
| `main/Led.swift` | ESP-IDFのGPIO用のC関数を包んだ構造体 |
| `main/BridgingHeader.h` | SwiftからESP-IDFのC関数を呼ぶためのヘッダ |
| `main/idf_component.yml` | 使う部品の一覧（`espressif/idf_swift`） |
| `sdkconfig` | ESP-IDFの設定（ボードの種類: `esp32c6`） |

## NanoC6のピン

| 機能 | GPIO |
|---|---|
| 青LED | 7 |
| ボタン | 9 |
| RGB LED（WS2812） | 20（電源のオン／オフ: 19） |
| 赤外線の送信 | 3 |
| Grove端子 | 1, 2 |

## 参考

- [Blinking an LED on the ESP32（Swift.org）](https://docs.swift.org/embedded/documentation/embedded/esp32guide)
- [swiftlang/swift-embedded-examples](https://github.com/swiftlang/swift-embedded-examples)（元のサンプル。Apache License 2.0）
- [M5Stack NanoC6 ドキュメント](https://docs.m5stack.com/ja/core/M5NanoC6)
- [Zenn: Embedded Swift 6.3でM5Stack NanoC6をLチカさせる](https://zenn.dev/trickart/articles/c0ac455d1f8a89)
