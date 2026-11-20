# ⚡ GPU Transcription with whisper.cpp and Vulkan (AMD & Intel)

![whisper.cpp 1.9.5](https://img.shields.io/badge/whisper.cpp-1.9.5-6f42c1)
![GPU: AMD | Intel, via Vulkan](https://img.shields.io/badge/GPU-AMD%20%7C%20Intel%20via%20Vulkan-ED1C24)
![Windows 11](https://img.shields.io/badge/Windows-11-0078D4?logo=windows&logoColor=white)
![Runs locally](https://img.shields.io/badge/Runs-locally%2C%20nothing%20uploaded-2ea44f)
[![Last updated](https://img.shields.io/github/last-commit/ImmuneMoon/GPU-Transcription-with-whisper.cpp-and-Vulkan?label=updated)](https://github.com/ImmuneMoon/GPU-Transcription-with-whisper.cpp-and-Vulkan/commits/main)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey)](LICENSE)

**📄 [Download the PDF](GPU-Transcription-with-whisper.cpp-and-Vulkan.pdf)**

## 📖 Overview
**GPU Transcription with whisper.cpp and Vulkan** shows you how to transcribe audio and video on an **AMD or Intel graphics card** on Windows. The [companion guide](https://github.com/ImmuneMoon/Transcribing-Videos-with-yt-dlp-and-faster-whisper) uses faster-whisper, whose GPU mode is for NVIDIA cards. This guide builds **whisper.cpp** with its **Vulkan** backend instead, which runs on the graphics driver you already have.

It covers installing the build tools, compiling whisper.cpp, downloading a model, transcribing one file, and then a whole folder with a script that skips finished files. On an AMD Radeon RX 9070 XT, it transcribed about 30 times faster than real time, with a larger and more accurate model than the CPU route can run at a usable speed.

## ⚠️ Important Notes
* **Transcripts are the speaker's words.** A transcript of someone else's video is still their work. Keep it private, and don't publish it or commit it to a public repository. The companion guide's [Privacy and Copyright](https://github.com/ImmuneMoon/Transcribing-Videos-with-yt-dlp-and-faster-whisper#11-privacy-and-copyright) section explains more.
* **You compile whisper.cpp yourself.** Its official Windows downloads are built for the CPU and for NVIDIA's CUDA, not for Vulkan. The build takes a few minutes, and the tools for it take about 7 GB of disk space.
* **Have an NVIDIA card?** You don't need this guide. Use faster-whisper's GPU mode from the companion guide, or whisper.cpp's ready-made CUDA downloads.
* **The first run after a driver install or update is slow to start,** about 30 seconds, while the driver prepares the GPU code (**Section 6**).

## 🚀 How to Use
Download the PDF above, or read on below. Work through Sections 2 to 6 to transcribe your first file, then use Section 7 for a whole folder. Sections 8 and 9 cover settings and speed.

This guide works alongside these:
* [Transcribing Videos with yt-dlp and faster-whisper](https://github.com/ImmuneMoon/Transcribing-Videos-with-yt-dlp-and-faster-whisper) - Downloading the audio, the CPU route, and the NVIDIA GPU route.
* [Python and Node Tool Environments](https://github.com/ImmuneMoon/Python-and-Node-Tool-Environments) - Installing Python and making virtual environments (venvs).
* [CLI Cheat Sheet](https://github.com/ImmuneMoon/CLI-Cheat-Sheet) - Installing tools with winget, and everyday terminal commands.

---

<!-- guide:start -->

Throughout, replace:

| Placeholder | With |
|---|---|
| `<your-folder>` | the folder that holds your audio or video files |
| `C:\Tools\whisper.cpp` and `C:\Tools\faster-whisper` | wherever you keep your tools, if you prefer somewhere else |

## Contents

1. [Which Route Is Yours](#1-which-route-is-yours)
2. [What You Need](#2-what-you-need)
3. [Install the Build Tools](#3-install-the-build-tools)
4. [Build whisper.cpp](#4-build-whispercpp)
5. [Download a Model](#5-download-a-model)
6. [Transcribe One File](#6-transcribe-one-file)
7. [Transcribe a Whole Folder](#7-transcribe-a-whole-folder)
8. [Settings That Matter](#8-settings-that-matter)
9. [How Fast to Expect](#9-how-fast-to-expect)
10. [Update or Rebuild](#10-update-or-rebuild)
11. [Troubleshooting](#11-troubleshooting)
12. [Sources](#12-sources)

---

## 1. Which Route Is Yours

| Your graphics card | Use |
|---|---|
| **NVIDIA** | faster-whisper on the GPU ([companion guide, Section 8.2](https://github.com/ImmuneMoon/Transcribing-Videos-with-yt-dlp-and-faster-whisper#82-nvidia-gpu)), or whisper.cpp's ready-made CUDA downloads. No compiling. |
| **AMD or Intel** | This guide: whisper.cpp built with Vulkan. |
| **None, or you'd rather not compile** | faster-whisper on the CPU ([companion guide](https://github.com/ImmuneMoon/Transcribing-Videos-with-yt-dlp-and-faster-whisper)). |

**Why faster-whisper doesn't use an AMD card by default:** its engine, CTranslate2, runs on the GPU through NVIDIA's CUDA when you install it with pip. CTranslate2 also publishes separate builds for AMD's ROCm platform, which need AMD's ROCm software. This guide doesn't cover those. It uses Vulkan, a graphics standard that AMD's and Intel's normal drivers already include.

**whisper.cpp** is a C++ version of OpenAI's Whisper speech recognition model. It uses its own model files (`ggml-*.bin`), and its **Vulkan** backend runs the model on any graphics card with a Vulkan driver.

---

## 2. What You Need

- **Windows 11** and an **AMD or Intel graphics card** with its current driver. The driver includes the Vulkan runtime.
- **About 8 GB of free space:** about 3.5 GB for Visual Studio Build Tools plus 1.2 GB for its installer cache, 1.8 GB for the Vulkan SDK, about 400 MB for the build, and 834 MB for the recommended model.
- **Python with PyAV,** to turn your audio into the format whisper.cpp reads. If you followed the [companion guide](https://github.com/ImmuneMoon/Transcribing-Videos-with-yt-dlp-and-faster-whisper#5-install-faster-whisper), its venv in `C:\Tools\faster-whisper` already has PyAV. Otherwise, make a venv in that folder and install `av` into it ([Python and Node Tool Environments](https://github.com/ImmuneMoon/Python-and-Node-Tool-Environments#5-create-a-virtual-environment) shows how).
- **Audio or video you're allowed to use.**

---

## 3. Install the Build Tools
whisper.cpp is compiled from its source code, which needs three free tools. Install them in **PowerShell**.

1. **Visual Studio Build Tools 2026,** with the C++ compiler and CMake. This installs both in one go, without opening the installer's window:
   ```powershell
   $parts = "--add Microsoft.VisualStudio.Workload.VCTools --includeRecommended " +
            "--add Microsoft.VisualStudio.Component.VC.CMake.Project"
   winget install --id Microsoft.VisualStudio.BuildTools --exact `
     --override "--wait --passive $parts"
   ```
   Prefer the installer's window? Install Build Tools, tick **Desktop development with C++**, and make sure **C++ CMake tools for Windows** is ticked on the right.
2. **Git,** to download whisper.cpp's code:
   ```powershell
   winget install --id Git.Git --exact
   ```
3. **The Vulkan SDK** from LunarG, which has the shader compiler and headers the build needs:
   ```powershell
   winget install --id KhronosGroup.VulkanSDK --exact
   ```
   Its installer is about 300 MB. It sets the `VULKAN_SDK` environment variable to where it installed, such as `C:\VulkanSDK\1.4.363.0`.

**Close PowerShell and open a new window,** so it sees the new tools and the `VULKAN_SDK` variable.

---

## 4. Build whisper.cpp

1. Download the code for the latest release. As of November 2026 that's **v1.9.5**. Check the [releases page](https://github.com/ggml-org/whisper.cpp/releases) for a newer one.
   ```powershell
   mkdir C:\Tools -Force
   cd C:\Tools
   git clone --depth 1 --branch v1.9.5 https://github.com/ggml-org/whisper.cpp.git
   cd whisper.cpp
   ```
2. Check that the Vulkan SDK is found:
   ```powershell
   $env:VULKAN_SDK
   ```
   It should print a folder. If it prints nothing, set it for this window, using the version folder you have in `C:\VulkanSDK`:
   ```powershell
   $env:VULKAN_SDK = 'C:\VulkanSDK\1.4.363.0'
   ```
3. Point to the CMake that came with Build Tools, then set up and run the build:
   ```powershell
   $vs = "C:\Program Files (x86)\Microsoft Visual Studio\18\BuildTools"
   $cmake = "$vs\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe"
   & $cmake -B build -DGGML_VULKAN=ON -DWHISPER_BUILD_TESTS=OFF -DWHISPER_BUILD_SERVER=OFF
   & $cmake --build build --config Release -j
   ```

   | Part | What it does |
   |---|---|
   | `-B build` | Puts everything the build makes in a `build` folder, so the source stays clean. |
   | `-DGGML_VULKAN=ON` | Turns on the Vulkan backend. Without it, you get a CPU-only build. |
   | `-DWHISPER_BUILD_TESTS=OFF -DWHISPER_BUILD_SERVER=OFF` | Skips the tests and the web server, which you don't need. |
   | `--config Release -j` | Builds the fast, optimized version, using all your CPU cores. |

   Instead of the full CMake path, you can open **Developer PowerShell for VS** from the Start menu, where plain `cmake` works.
4. When it finishes, the program is `build\bin\Release\whisper-cli.exe`, with `ggml-vulkan.dll` beside it. Warnings that mention **C4267** or **C4244** are harmless.

---

## 5. Download a Model
whisper.cpp comes with a script that downloads its models from the official [whisper.cpp Hugging Face repository](https://huggingface.co/ggerganov/whisper.cpp). In `C:\Tools\whisper.cpp`:

```powershell
.\models\download-ggml-model.cmd large-v3-turbo-q8_0
```

It saves `models\ggml-large-v3-turbo-q8_0.bin`, about 834 MB.

| Model | Size | Notes |
|---|---|---|
| `large-v3-turbo-q8_0` | 834 MB | The recommended one: fast on a GPU, and far more accurate than the small CPU models. |
| `large-v3-turbo-q5_0` | 547 MB | A smaller, more compressed copy of the same model. |
| `large-v3-turbo` | 1.5 GB | The uncompressed model. |

The `q8_0` and `q5_0` endings mean the model's numbers are stored in a compact 8-bit or 5-bit form, which saves memory. The turbo models are multilingual.

**Check the download** against the SHA-256 shown on the file's page in the Hugging Face repository:
```powershell
Get-FileHash .\models\ggml-large-v3-turbo-q8_0.bin
```

---

## 6. Transcribe One File
whisper.cpp's README says `whisper-cli` reads **16-bit WAV** files, so convert your audio to 16 kHz mono WAV first. With [ffmpeg](https://github.com/ImmuneMoon/Transcribing-Videos-with-yt-dlp-and-faster-whisper#44-when-you-need-ffmpeg):

```powershell
ffmpeg -i "<your-folder>\talk.m4a" -ar 16000 -ac 1 -c:a pcm_s16le "<your-folder>\talk.wav"
```

No ffmpeg? The Section 7 script converts the audio for you with PyAV, so you can skip to it.

Then transcribe it, from `C:\Tools\whisper.cpp`:

```powershell
$model = '.\models\ggml-large-v3-turbo-q8_0.bin'
.\build\bin\Release\whisper-cli.exe -m $model -f "<your-folder>\talk.wav" -l en -otxt
```

`-l en` sets the language to English, and `-otxt` saves `talk.wav.txt` beside the audio. For another language, use its code, such as `-l fr`, or `-l auto` to detect it.

**Check it's using the graphics card.** The start of the output should include lines like these:

```
ggml_vulkan: Found 2 Vulkan devices:
ggml_vulkan: 0 = AMD Radeon RX 9070 XT ...
...using Vulkan0 backend
```

If your processor has built-in graphics, it shows up as a second device. Device 0 is used unless you pick another with `-dev 1`, so check that device 0 is your graphics card.

**The first run after a driver install or update** spends about 30 seconds on "load time" while the driver compiles the GPU code (shaders). The driver keeps them, and the next load took under a second.

---

## 7. Transcribe a Whole Folder
This script transcribes every audio and video file in a folder into a `Transcripts` folder beside them, on the GPU. Like the companion guide's folder script, it **skips files that already have a transcript**, so you can stop it and run it again to carry on. It uses PyAV to turn each file into the WAV that whisper.cpp reads, so you don't need ffmpeg.

Save it as `C:\Tools\faster-whisper\transcribe_folder_gpu.py`:

```python
"""Transcribe every audio or video file in a folder on the GPU, with whisper.cpp.

Skips files that already have a transcript. Transcripts are the speaker's own
words: keep them private.
"""
import argparse
import json
import subprocess
import tempfile
import wave
from pathlib import Path

import av

WHISPER = Path(r"C:\Tools\whisper.cpp")
CLI = WHISPER / "build" / "bin" / "Release" / "whisper-cli.exe"
MEDIA = {".m4a", ".mp3", ".wav", ".flac", ".ogg", ".opus", ".webm",
         ".mp4", ".mkv", ".mov", ".avi"}


def write_wav(src, dst, limit=None):
    """Save the first audio track as 16 kHz mono 16-bit WAV. Returns seconds."""
    resampler = av.AudioResampler(format="s16", layout="mono", rate=16000)
    wanted = limit * 16000 if limit else None
    total = 0
    with av.open(str(src)) as container, wave.open(str(dst), "wb") as w:
        w.setnchannels(1)
        w.setsampwidth(2)
        w.setframerate(16000)

        def write(frames):
            nonlocal total
            for out in frames:
                data = out.to_ndarray().reshape(-1)
                if wanted:
                    data = data[:max(0, wanted - total)]
                w.writeframes(data.tobytes())
                total += data.size

        for frame in container.decode(container.streams.audio[0]):
            write(resampler.resample(frame))
            if wanted and total >= wanted:
                break
        else:
            write(resampler.resample(None))
    return total / 16000


def stamp(seconds):
    m, s = divmod(int(seconds), 60)
    h, m = divmod(m, 60)
    return f"{h}:{m:02d}:{s:02d}"


def below_normal():
    """Run at Below normal priority. whisper-cli inherits it."""
    import ctypes
    k32 = ctypes.windll.kernel32
    # GetCurrentProcess returns a special handle, -1. Without these two lines,
    # ctypes cuts it to 32 bits and SetPriorityClass quietly fails.
    k32.GetCurrentProcess.restype = ctypes.c_void_p
    k32.SetPriorityClass.argtypes = (ctypes.c_void_p, ctypes.c_uint32)
    if not k32.SetPriorityClass(k32.GetCurrentProcess(), 0x4000):
        print("Could not lower the priority")


def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("folder")
    ap.add_argument("--out", help="where transcripts go (default: Transcripts)")
    ap.add_argument("--model", default="large-v3-turbo-q8_0")
    ap.add_argument("--only", help="only files whose name contains this")
    ap.add_argument("--test", type=int, metavar="SECONDS",
                    help="only the first SECONDS of each file, into a -test file")
    ap.add_argument("--normal", action="store_true", help="run at normal priority")
    ap.add_argument("--no-context", action="store_true",
                    help="for a file that gets stuck repeating a line")
    args = ap.parse_args()
    if not args.normal:
        below_normal()

    model = WHISPER / "models" / f"ggml-{args.model}.bin"
    for needed in (CLI, model):
        if not needed.exists():
            raise SystemExit(f"Not found: {needed}")
    folder = Path(args.folder)
    out_dir = Path(args.out) if args.out else folder / "Transcripts"
    out_dir.mkdir(parents=True, exist_ok=True)
    files = sorted(p for p in folder.iterdir() if p.suffix.lower() in MEDIA)
    if args.only:
        files = [p for p in files if args.only.lower() in p.name.lower()]
    print(f"{len(files)} file(s) found")

    with tempfile.TemporaryDirectory() as tmp:
        wav = Path(tmp) / "audio.wav"
        result = Path(tmp) / "result"
        for path in files:
            target = out_dir / (path.stem + ("-test" if args.test else "") + ".txt")
            if target.exists() and not args.test:
                print(f"skip (already done): {path.name}")
                continue
            print(f"transcribing: {path.name}")
            try:
                seconds = write_wav(path, wav, args.test)
                cmd = [str(CLI), "-m", str(model), "-f", str(wav), "-l", "en",
                       "-oj", "-of", str(result), "-np"]
                if args.no_context:
                    cmd += ["-mc", "0"]
                run = subprocess.run(cmd, capture_output=True, text=True,
                                     encoding="utf-8", errors="replace")
                if run.returncode != 0:
                    raise RuntimeError(run.stderr.strip()[-500:])
                text = Path(f"{result}.json").read_text(encoding="utf-8")
                segments = json.loads(text).get("transcription", [])
            except Exception as error:
                print(f"FAILED: {path.name}: {error}")
                continue
            lines = [f"# {path.name} (private transcript, not for sharing)"]
            for seg in segments:
                words = seg["text"].strip()
                if words:
                    lines.append(f"[{stamp(seg['offsets']['from'] / 1000)}] {words}")
            part = target.with_suffix(".part")
            part.write_text("\n".join(lines) + "\n", encoding="utf-8")
            part.replace(target)
            print(f"done: {path.name} ({stamp(seconds)} of audio)")


if __name__ == "__main__":
    main()
```

Run it with the faster-whisper venv's Python:

```powershell
cd C:\Tools\faster-whisper
.\.venv\Scripts\python.exe transcribe_folder_gpu.py "<your-folder>"
```

| Option | What it does |
|---|---|
| `--out "<folder>"` | Saves transcripts there instead of a `Transcripts` folder beside the files. |
| `--only "name"` | Only files whose name contains this. |
| `--test 90` | Transcribes only the first 90 seconds of each file, into a `-test` file. Good for checking settings before a long batch. |
| `--model NAME` | Uses `models\ggml-NAME.bin`, such as `large-v3-turbo-q5_0`. |
| `--normal` | Runs at normal priority instead of Below normal (**Section 8**). |
| `--no-context` | Only for a file that gets stuck repeating a line (**Section 8**). |

How it works:
- **It converts each file to a temporary WAV,** runs `whisper-cli` on it with `-oj` (save the result as JSON) and `-np` (print nothing else), and turns each segment's start time, in milliseconds, into a `[0:01:23]` timestamp.
- **It writes to a `.part` file first,** then renames it to `.txt` only when the file is finished, so a stopped run never leaves a half-written transcript that would be skipped next time.
- **A file that fails** is reported and skipped, and the rest carry on. To redo a file, delete its `.txt`.
- **To transcribe another language,** change `"-l", "en"` in the script to the language's code, or `"auto"`.

---

## 8. Settings That Matter

**Keep whisper.cpp's normal context.** By default, whisper.cpp passes the text it just wrote along to the next stretch of audio, which keeps punctuation and capital letters consistent. `-mc 0` (`--max-context 0`) turns that off. It's often suggested as a fix for transcripts that loop, but with it on, punctuation and capitals faded partway through a transcript. So the script leaves it off, and `--no-context` turns it on for a file that actually gets stuck repeating a line.

**Run at Below normal priority, not Idle.** The companion guide runs the CPU version at **Idle** (Low) priority. On the GPU, use **Below normal**: at Idle, a busy processor can starve the thread that keeps the graphics card fed, and the job crawls. The script sets Below normal itself, and `whisper-cli` inherits it, because Windows passes Below normal and Idle on to programs a process starts.

**Why the script declares the types.** `GetCurrentProcess` returns a special handle, -1. Unless the script tells Python's `ctypes` it's a pointer, `ctypes` cuts it to 32 bits, and `SetPriorityClass` fails without an error you'd notice. The two lines in `below_normal()` prevent that. The same fix applies to any Python script that sets its own priority this way.

**Pick the right GPU.** If your processor has built-in graphics, `whisper-cli` lists it as a second Vulkan device. Device 0 is used by default. If it isn't your graphics card, add `"-dev", "1"` to the command in the script.

---

## 9. How Fast to Expect
On an AMD Radeon RX 9070 XT, with ten short videos (about 11 minutes of audio):

| Route | Model | Time | Speed |
|---|---|---|---|
| CPU, faster-whisper | `distil-small.en` | 1 min 45 s | About 8 times real time |
| GPU, whisper.cpp with Vulkan | `large-v3-turbo-q8_0` | 31 s | About 30 times real time |

A single clip of just under 3 minutes took 5.9 seconds once the GPU code was cached.

The GPU route is faster **and** uses a much larger model. On one clip, the small CPU model dropped a 16-second stretch of speech, and the GPU model caught all of it, with better punctuation.

Speed depends on your card, the model, and how much talking there is. Run `--test 90` on a few files before a long batch.

---

## 10. Update or Rebuild

- **After a graphics driver update,** expect the first run to be slow to start again (**Section 6**). Nothing needs rebuilding.
- **For a newer whisper.cpp,** download the new version's code and build it the same way. In `C:\Tools`:
  ```powershell
  Rename-Item whisper.cpp whisper.cpp-old
  git clone --depth 1 --branch <new-version> https://github.com/ggml-org/whisper.cpp.git
  ```
  Build it as in **Section 4**, copy `ggml-large-v3-turbo-q8_0.bin` from the old `models` folder into the new one, and check it works. Then delete `whisper.cpp-old`.
- **After updating the Vulkan SDK or Build Tools,** delete the `build` folder and build again, so CMake picks up the new versions.

---

## 11. Troubleshooting

| What you see | Why | What to do |
|---|---|---|
| CMake says **Could NOT find Vulkan** | `VULKAN_SDK` isn't set in this window | Open a new PowerShell window, or set `$env:VULKAN_SDK` (**Section 4**). Then delete the `build` folder and build again |
| **cmake is not recognized** | Build Tools' CMake isn't on `PATH` | Use the full path as in **Section 4**, or open **Developer PowerShell for VS** |
| CMake can't find a **C++ compiler** (`CMAKE_CXX_COMPILER`) | The C++ workload isn't installed | Add **Desktop development with C++** in the Visual Studio Installer (**Section 3**) |
| No **ggml_vulkan** lines, and it's slow | It was built without Vulkan | Delete `build`, check `$env:VULKAN_SDK`, and build again with `-DGGML_VULKAN=ON` |
| It uses the **built-in graphics** | Device 0 is the processor's graphics | Add `-dev 1` (**Section 8**) |
| The **first run** takes about 30 seconds to start | The driver is compiling GPU code after an install or update | Wait. Later runs start in under a second |
| **Punctuation and capitals fade** partway through | `-mc 0` (no context) is on | Leave it off (**Section 8**) |
| The same line **repeats** over and over | The model got stuck in a loop | Run that file again with `--no-context` |
| The job **crawls** while the PC is busy | It's running at Idle priority | Use Below normal, as the script does (**Section 8**) |
| Much **slower than Section 9** even at Below normal | Another program, such as a game or a 3D app, is using the graphics card heavily | Close it, or let the job run while the card is free |
| **Could not lower the priority** | `ctypes` cut the process handle to 32 bits | Keep the two type lines in `below_normal()` (**Section 8**) |
| **Not found:** `...ggml-NAME.bin` | The model name doesn't match a file in `models` | Check the name, or download it (**Section 5**) |
| **FAILED** on one file | The file has no audio track, is damaged, or uses an unusual format | Check it plays. Other files carry on |
| Wrong words or gibberish on **another language** | The script sets English | Change `"-l", "en"` to the language's code, or `"auto"` (**Section 7**) |

---

## 12. Sources

**whisper.cpp**
- [whisper.cpp README (building, Vulkan, input format)](https://github.com/ggml-org/whisper.cpp/blob/master/README.md)
- [whisper.cpp releases (Windows downloads and versions)](https://github.com/ggml-org/whisper.cpp/releases)
- [whisper-cli options](https://github.com/ggml-org/whisper.cpp/blob/master/examples/cli/README.md)
- [whisper.cpp models on Hugging Face](https://huggingface.co/ggerganov/whisper.cpp)

**Build tools**
- [Vulkan SDK (LunarG)](https://vulkan.lunarg.com/sdk/home)
- [Visual Studio Build Tools workload and component IDs](https://learn.microsoft.com/en-us/visualstudio/install/workload-component-id-vs-build-tools)
- [winget install](https://learn.microsoft.com/en-us/windows/package-manager/winget/install)

**faster-whisper's engine**
- [CTranslate2 installation (CUDA, and ROCm builds)](https://opennmt.net/CTranslate2/installation.html)

**Windows priority**
- [Scheduling priorities (priority classes, and what a started program inherits)](https://learn.microsoft.com/en-us/windows/win32/procthread/scheduling-priorities)
- [SetPriorityClass](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-setpriorityclass)
- [GetCurrentProcess (the -1 pseudo handle)](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-getcurrentprocess)

<!-- guide:end -->

## ☕ Support the Project

If you find this project helpful and want to support further development by Fulllion Creative Works, consider leaving a tip!

* [Donate via PayPal](https://www.paypal.com/donate/?hosted_button_id=LCDZX75HR4CLC)
* [Support on Ko-fi](https://ko-fi.com/fulllion)

---
© 2026 Fulllion Creative Works. All rights reserved.
