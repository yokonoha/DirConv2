#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# DirConv2 Made by Y.Yokoha / Licensed under MIT License

import argparse
import concurrent.futures
import subprocess
from pathlib import Path
import os
import sys
import shutil

SUPPORTED_EXTS = {
    ".3g2",
    ".3ga",
    ".ac3",
    ".aif",
    ".aifc",
    ".aiff",
    ".alac",
    ".amr",
    ".ape",
    ".caf",
    ".dff",
    ".dts",
    ".dsf",
    ".mp3",
    ".flac",
    ".m4a",
    ".m4b",
    ".m4p",
    ".m4r",
    ".aac",
    ".wav",
    ".ogg",
    ".oga",
    ".opus",
    ".wma",
    ".mka",
    ".mpc",
    ".ra",
    ".ram",
    ".spx",
    ".tta",
    ".wv",
}


CODEC_SETTINGS = {
    "mp3": {
        "ext": ".mp3",
        "args": ["-c:a", "libmp3lame"],
    },
    "aac": {
        "ext": ".m4a",
        "args": ["-c:a", "aac"],
    },
    "m4a": {
        "ext": ".m4a",
        "args": ["-c:a", "aac"],
    },
    "opus": {
        "ext": ".opus",
        "args": ["-c:a", "libopus"],
    },
    "flac": {
        "ext": ".flac",
        "args": ["-c:a", "flac"],
    },
    "wav": {
    "ext": ".wav",
    "args": ["-c:a", "pcm_s16le"],
},
"wav24": {
    "ext": ".wav",
    "args": ["-c:a", "pcm_s24le"],
},
"wma": {
    "ext": ".wma",
    "args": ["-c:a", "wmav2"],
},
}


def check_ffmpeg():
    try:
        subprocess.run(
            ["ffmpeg", "-version"],
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL,
            check=True,
        )
    except Exception:
        print("[DirConv2 ERR]ffmpeg が見つかりません")
        sys.exit(1)


def collect_files(input_dir: Path):
    files = []

    for path in input_dir.rglob("*"):
        if path.is_file() and path.suffix.lower() in SUPPORTED_EXTS:
            files.append(path)

    return files


def build_output_path(input_file, input_root, output_root, target_ext):
    rel = input_file.relative_to(input_root)
    rel = rel.with_suffix(target_ext)

    out_path = output_root / rel
    out_path.parent.mkdir(parents=True, exist_ok=True)

    return out_path


def convert_file(
    input_file: Path,
    output_file: Path,
    codec: str,
    bitrate: str | None,
    overwrite: bool,
):
    codec_conf = CODEC_SETTINGS[codec]

 
    if input_file.suffix.lower() == codec_conf["ext"]:
        try:
            if input_file.resolve() == output_file.resolve():
                print(f"[DirConv2 SKP] {output_file} は入力ファイルと同じため、コピーをスキップします。")
                return

            if output_file.exists():
                if overwrite:
                    shutil.copy2(input_file, output_file)
                else:
                    print(f"[DirConv2 SKP] {output_file} は既に存在しているため、スキップします。")
                    return
            else:
                shutil.copy2(input_file, output_file)

            print(f"[DirConv2 CPY] {input_file} -> {output_file}")
            return

        except OSError as e:
            print(f"[DirConv2 ERR] {input_file}")
            print(e)
            return


    cmd = [
        "ffmpeg",
        "-hide_banner",
        "-loglevel",
        "error",
    ]

    if overwrite:
        cmd.append("-y")
    else:
        cmd.append("-n")

    cmd += [
        "-i",
        str(input_file),

        "-map_metadata",
        "0",

        "-map",
        "0",

        "-c:v",
        "copy",
    ]

    cmd += codec_conf["args"]

    if bitrate and codec not in ("flac", "wav","wav24"):
        cmd += ["-b:a", bitrate]

    cmd.append(str(output_file))

    try:
        subprocess.run(
            cmd,
            stdout=subprocess.DEVNULL,
            stderr=subprocess.PIPE,
            check=True,
        )

        print(f"[DirConv2 RES OK] {input_file} -> {output_file} (変換が完了しました)")

    except subprocess.CalledProcessError as e:
        print(f"[DirConv2 ERR] {input_file} (変換できませんでした)")
        print("[DirConv2 ERR]"+e.stderr.decode(errors="ignore"))


def main():
    parser = argparse.ArgumentParser()

    parser.add_argument(
        "-i",
        required=True,
        help="入力ディレクトリ",
    )

    parser.add_argument(
        "-o",
        default="here",
        help="出力ディレクトリ (省略時または here でカレントディレクトリ)",
    )

    parser.add_argument(
        "-c",
        "--codec",
        choices=CODEC_SETTINGS.keys(),
        default="mp3",
        help="出力コーデック",
    )

    parser.add_argument(
        "--kbps",
        default="320k",
        help="ビットレート (例: 320k)",
    )

    parser.add_argument(
        "--threads",
        type=int,
        default=os.cpu_count(),
        help="並列スレッド数",
    )

    parser.add_argument(
         "--overwrite",
        action="store_true",
        help="既存ファイルを上書き",
    )

    parser.add_argument(
        "--mk",
        action="store_true",
        help="出力先に入力ディレクトリ名のフォルダーを作成",
    )

    args = parser.parse_args()

    check_ffmpeg()

    input_dir = Path.cwd() if args.i.lower() == "here" else Path(args.i).expanduser().resolve()
    output_dir = Path.cwd() if args.o.lower() == "here" else Path(args.o).expanduser().resolve()

    if not input_dir.is_dir():
        print("[DirConv2 IO ERR]入力ディレクトリが存在しないか、ディレクトリではありません")
        return

    files = collect_files(input_dir)

    print(f"[DirConv2 Info] {len(files)} 件の対象ファイルが見つかりました。処理を開始しています...")
    print(f"[DirConv2 Info]設定スレッド数: {args.threads}")

    tasks = []
    output_root = output_dir / input_dir.name if args.mk else output_dir

    for file in files:
        out_path = build_output_path(
            file,
            input_dir,
            output_root,
            CODEC_SETTINGS[args.codec]["ext"],
        )

        tasks.append((file, out_path))

    with concurrent.futures.ThreadPoolExecutor(
        max_workers=args.threads
    ) as executor:

        futures = []

        for in_file, out_file in tasks:
            futures.append(
                executor.submit(
                    convert_file,
                    in_file,
                    out_file,
                    args.codec,
                    args.kbps,
                    args.overwrite,
                )
            )

        for completed, future in enumerate(concurrent.futures.as_completed(futures), start=1):
            future.result()
            percentage = completed * 100 // len(futures)
            print(f"[DirConv2 Progress] {completed}/{len(futures)} ({percentage}%)")

    print("[DirConv2 Info]変換が完了しました。")


if __name__ == "__main__":
    main()
