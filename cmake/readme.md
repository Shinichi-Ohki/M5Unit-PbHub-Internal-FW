# CMake Build for PbHub (GCC-ARM)

STM32CubeMX / Keil を使わずに CMake + GCC-ARM でビルドするためのプロジェクトです。

## 前提条件

```bash
brew install cmake
brew install --cask gcc-arm-embedded
```

## ビルド方法

```bash
cd cmake
mkdir -p build && cd build
cmake ..
make
```

## 生成物

| ファイル | 説明 |
|----------|------|
| `PbHub` | ELF形式 |
| `PbHub.bin` | 書き込み用バイナリ |
| `PbHub.hex` | Intel HEX形式 |
| `PbHub.map` | メモリマップ |

## ターゲット

- MCU: STM32F030F4P6 (Cortex-M0)
- Flash: 16KB
- RAM: 4KB
- Clock: 48MHz

## ファイル構成

```
cmake/
├── CMakeLists.txt        # ビルド設定
├── stm32f030f4.ld        # リンカスクリプト
├── startup_stm32f030x6.s # スタートアップコード (GCC用)
└── readme.md
```

## 元のプロジェクトとの違い

| 項目 | 元 (CubeMX/Keil) | このプロジェクト |
|------|------------------|------------------|
| ビルドシステム | Keil MDK | CMake |
| コンパイラ | ARMCC | GCC-ARM |
| スタートアップ | ARMアセンブラ | GNUアセンブラ |
| リンカスクリプト | scatter file (.sct) | GNU ld (.ld) |

## 書き込み

ST-Linkを使用する場合:

```bash
st-flash write PbHub.bin 0x08000000
```

または OpenOCD:

```bash
openocd -f interface/stlink.cfg -f target/stm32f0x.cfg \
  -c "program PbHub.elf verify reset exit"
```
