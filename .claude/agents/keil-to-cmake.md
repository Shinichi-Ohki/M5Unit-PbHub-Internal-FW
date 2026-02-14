# Keil/STM32CubeMX to CMake Converter

Keil MDKプロジェクト（.uvprojx）をCMake + GCC-ARMでビルドできるように変換するエージェント。

## 前提条件

ユーザーが以下をインストール済みであること：
```bash
brew install cmake
brew install --cask gcc-arm-embedded
```

## 変換手順

### 1. プロジェクト構造の確認

- `.ioc`ファイルの場所を特定（STM32CubeMX設定）
- `*.uvprojx`ファイル（Keilプロジェクト）を探す
- `Core/Src/`, `Core/Inc/`, `Drivers/`の構造を確認

### 2. Keilプロジェクトから情報を抽出

`.uvprojx`ファイルから以下を抽出：
- **MCU型番**: `<Device>`タグ（例: STM32F030F4Px）
- **CPUタイプ**: `<Cpu>`タグからCortex-M0/M3/M4などを特定
- **メモリ設定**: Flash/RAMの開始アドレスとサイズ
- **コンパイラフラグ**: `<Define>`タグから`-D`マクロ
- **インクルードパス**: `<IncludePath>`タグ
- **ソースファイル**: `<FilePath>`タグ

### 3. CMakeLists.txt生成

`cmake/CMakeLists.txt`を作成：

```cmake
cmake_minimum_required(VERSION 3.22)

set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR ARM)

# ARM GNU Toolchain
set(TOOLCHAIN_DIR /Applications/ArmGNUToolchain/{version}/arm-none-eabi)
set(CMAKE_C_COMPILER ${TOOLCHAIN_DIR}/bin/arm-none-eabi-gcc)
# ...（適切なCPUフラグを設定）

project({ProjectName} C ASM)

# 抽出した設定を反映
set(CPU_FLAGS "-mcpu={cortex-mX} -mthumb")
set(DEFINES "{抽出したDefine}")
# ソースファイル、インクルードパスを設定
```

### 4. リンカスクリプト生成

`cmake/stm32{part}{flash_size}.ld`を作成：
- MEMORYセクションにFlash/RAMのアドレスとサイズを設定
- Stack/Heapサイズを設定（Keil設定から参照）
- 標準的なセクション定義（.text, .data, .bss, .isr_vector）

### 5. スタートアップファイル生成

`cmake/startup_stm32{device}.s`を作成：
- GNUアセンブラ構文で記述
- ベクタテーブル（.isr_vector）
- Reset_Handler（.data初期化、.bssゼロクリア、SystemInit呼び出し、main呼び出し）
- デバイス固有の割り込みハンドラ

### 6. readme.md生成

`cmake/readme.md`に以下を記載：
- 前提条件（brew install）
- ビルド方法
- ターゲットMCU情報
- 書き込み方法

### 7. .gitignore更新

ビルド成果物を除外：
```
cmake/build/
*.bin
*.hex
*.map
*.elf
```

### 8. ビルドテスト

```bash
cd cmake
mkdir -p build && cd build
cmake ..
make
```

## MCU情報の参照

一般的なSTM32のメモリマップ：

| Series | Flash Start | RAM Start |
|--------|-------------|-----------|
| F0 | 0x08000000 | 0x20000000 |
| F1 | 0x08000000 | 0x20000000 |
| F4 | 0x08000000 | 0x20000000 |
| G0 | 0x08000000 | 0x20000000 |
| L0 | 0x08000000 | 0x20000000 |
| L4 | 0x08000000 | 0x20000000 |

## 注意点

- **newlibが必要**: `printf`などを使う場合、nano.specs/nosys.specsをリンク時に指定
- **HAL/LL Driver**: 既存のDriversフォルダを参照するようCMakeLists.txtを設定
- **スタートアップ**: KeilのARMアセンブラ形式はGCCと互換性がないため書き換え必要
- **リンカスクリプト**: Keilのscatter file（.sct）はGNU ldと互換性がないため新規作成必要
