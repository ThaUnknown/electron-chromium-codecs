# electron-chromium-codecs
This repository contains the necessary git patches to build a custom electron dist with bundled HEVC, AC3 and E-AC3 codecs support for chromium.

## Compiling Electron
Follow the electron [build instructions](https://www.electronjs.org/docs/latest/development/build-instructions-gn).
### Linux
On Ubuntu >= 20.04, install the following libraries:

```bash
$ sudo apt-get install build-essential clang libdbus-1-dev libgtk-3-dev \
                       libnotify-dev libasound2-dev libcap-dev \
                       libcups2-dev libxtst-dev \
                       libxss1 libnss3-dev gcc-multilib g++-multilib curl \
                       gperf bison python3-dbusmock openjdk-8-jre \
                       git npm
```

Update node and npm to the latest version using your preferred method.

Clone [depot_tools](https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up) and add it to PATH.

```bash
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
$ export PATH=/path/to/depot_tools:$PATH
```
Get Electron

```bash
$ mkdir electron && cd electron
$ gclient config --name "src/electron" --unmanaged https://github.com/electron/electron
$ gclient sync --with_branch_heads --with_tags
# This will take a while, go get a coffee.
```

Move patches to respective directories
```bash
$ mv look_chromium_hevc_ac3.patch electron/src/
$ mv look_electron_hevc_ac3.patch electron/src/electron/
$ mv look_ffmpeg_hevc_ac3.patch electron/src/third_party/ffmpeg/
```

Apply the patches

```bash
$ cd src/ && git apply look_chromium_hevc_ac3.patch
$ cd electron/ && git apply look_electron_hevc_ac3.patch
$ cd .. && cd third_party/ffmpeg/ && git apply look_ffmpeg_hevc_ac3.patch
```

Build Electron (step 5)

```bash
$ cd src
$ export CHROMIUM_BUILDTOOLS_PATH=`pwd`/buildtools
$ gn gen out/Release --args="import(\"//electron/build/args/release.gn\")"
$ ninja -C out/Release electron
```

Strip the binaries and pack the distribution (step 6)

```bash
$ electron/script/strip-binaries.py -d out/Release
$ ninja -C out/Release electron:electron_dist_zip
```
Result dist will be inside src/out/Release/dist.zip
