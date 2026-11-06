# DirConv2 (ベータ版)
初代DirConvから大幅に改善した「ファイル構造そのまま 音楽ファイル形式変換」コマンドラインツール  
音声ファイルを一括変換するためのシンプルなPython製コマンドラインツールです。  
FFmpegを利用し、指定したディレクトリ以下にある音声ファイルを再帰的に検索して、指定した形式に変換します。  

## Features  

- ディレクトリ内の音声ファイルを再帰的に検索
- 複数ファイルの一括変換
- マルチスレッドによる並列処理
- 元のディレクトリ構造を維持した出力
- 既存ファイルの上書き設定
- ビットレートの指定
- 変換ロスを発生させないよう、INとOUTが同一フォーマットの場合、コピー処理に自動変更  

## Requirements  

- Python 3.10 以降  
- FFmpeg  

### FFmpegのインストール（Windows）  

本ツールの利用には、別途FFmpegのインストールとPATHの設定が必要です。  
Windowsの場合、コマンドプロンプトまたはPowerShellから以下を実行することでインストールできます。  

```bash
winget install ffmpeg
```
※ FFmpegは本ツールとは別の第三者製ソフトウェアです。インストール前にパッケージ情報や配布元をご確認のうえ、ご自身の判断で実行してください。  

インストール後、新しいコマンドプロンプトを開き、以下のコマンドで動作を確認してください。

```bash
ffmpeg -version
```

FFmpeg公式サイト: https://ffmpeg.org/

## Usage
```bash
python DirConv2.py -i INPUT_DIR [options]
```

### Basic example

入力ディレクトリ内の音声ファイルをMP3（320kbps）に変換します。

```bash
python DirConv2.py -i "C:\Music"
```

### Specify output directory

```bash
python DirConv2.py -i "C:\Music" -o "C:\Converted"
```

### Specify codec

```bash
python DirConv2.py -i "C:\Music" -c flac
```

### Specify bitrate

```bash
python DirConv2.py -i "C:\Music" -c mp3 --kbps 192k
```

### Overwrite existing files

```bash
python DirConv2.py -i "C:\Music" --overwrite
```

### Create an input directory folder in the output directory

```bash
python DirConv2.py -i "C:\Music" -o "C:\Converted" --mk
```

この場合、出力先に `Music` フォルダーが作成されます。

## Options

| Option | Default | Description |
|---|---|---|
| `-i` | 必須 | 入力ディレクトリ |
| `-o` | `here` | 出力ディレクトリ |
| `-c`, `--codec` | `mp3` | 出力形式 |
| `--kbps` | `320k` | 音声ビットレート |
| `--threads` | CPUコア数 | 並列スレッド数 |
| `--overwrite` | 無効 | 既存ファイルを上書き |
| `--mk` | 無効 | 出力先に入力ディレクトリ名のフォルダーを作成 |

### Supported output formats

| Format | Output extension | Codec |
|---|---|---|
| `mp3` | `.mp3` | libmp3lame |
| `aac` | `.m4a` | AAC |
| `m4a` | `.m4a` | AAC |
| `opus` | `.opus` | libopus |
| `flac` | `.flac` | FLAC |
| `wav` | `.wav` | PCM 16-bit |
| `wav24` | `.wav` | PCM 24-bit |
| `wma` | `.wma` | Windows Media Audio 2 |

## Tips

本ツールをPATHに登録し、任意のディレクトリからコマンドとして利用することも可能です。  
その場合は、PyInstallerなどを使用してEXEファイルにパッケージングしてください。  

```bash
pyinstaller --onefile --name DirConv2 DirConv2.py
```

生成されたEXEファイルは `dist` フォルダー内に配置されます。  
なお、EXE化した場合もFFmpegは別途必要です。  

## License
MIT License  

## Third-party software  
本ソフトウェアはFFmpegを外部プログラムとして利用します。FFmpegは本ソフトウェアには同梱されていません。  
- FFmpeg: https://ffmpeg.org/
- PyInstaller（EXE化する場合）: https://pyinstaller.org/
