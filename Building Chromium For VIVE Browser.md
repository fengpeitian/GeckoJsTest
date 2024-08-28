# Building Chromium For VIVE Browser

### Preface

Chromium is an open-source browser project that aims to build a safer, faster, and more stable way for all users to experience the web.

The project's web site is <u>[https://www.chromium.org](https://www.chromium.org)</u>.

> [!TIP]
> To check out the source code locally, don't use `git clone`! instead.

### Compilation Environment

Android builds are only supported on Linux: **Ubuntu** ([22.04.4 LTS, Jammy Jellyfish](https://releases.ubuntu.com/22.04.4/))

- A x86-64 machine, with more than 16GB of RAM
- At least 100GB of free disk space
- Software requirements

  - git
  - python3
  - curl
  - ninja-build
  - lsb_release
  - pkg-config

### Install depot_tools

1. Clone the `depot_tools` repository:
   ```bash
   ```

git clone [https://chromium.googlesource.com/chromium/tools/depot_tools.git](https://chromium.googlesource.com/chromium/tools/depot_tools.git)

```

2. Add `depot_tools` to the end of your environment PATH (must use the absolute path):
	```bash
export PATH="$PATH:/path/to/depot_tools"
```

### Get the code

1. Create a directory for checkout, which can be named as desired, such as `chromium`:
   ```bash
   ```

$ mkdir chromium && cd chromium

```

2. Run the fetch tool from `depot_tools` to check out the code (this might take a lot of time). If you don't need the full repo history, you can save time by using `--no-history` arguments.
	```bash
fetch --nohooks --no-history chromium
```

when fetch completes, it will have created a hidden `.gclient` file and a directory called `src` in the working directory.

1. Edit the file `.gclient` like this:
   ```
   ```

solutions = [
{
"name": "src",
"url": "[https://chromium.googlesource.com/chromium/src.git"](https://chromium.googlesource.com/chromium/src.git%22),
"managed": False,
"custom_deps": {},
"custom_vars": {
"checkout_pgo_profiles": True,
},
},
]
target_os = ["android"]

```

2. If errors occur while fetching sub-repos then you may be able to correct them by running this command:
	```bash
gclient sync
```

3. Switched to the `src` directory:
   ```bash
   ```

cd src

```

4. Install additional build dependencies:
	```bash
./build/install-build-deps.sh --android
```

5. Run the Chromium-specific hooks, which will download additional binaries and other things you might need:
   ```bash
   ```

gclient runhooks

```

6. Add `wolvic-chromium` as a new git-remote to your existing checkout:
	```bash
git remote add wolvic-chromium https://github.com/Igalia/wolvic-chromium
#If fetch always fails, you can try SSH
#git remote add wolvic-chromium git@github.com:Igalia/wolvic-chromium.git
git fetch --depth=1 wolvic-chromium
```

7. switch to the `wolvic` branch:
   ```bash
   ```

git switch wolvic

```

8. If errors occur while fetching sub-repos, you need to run this command for force sync:
	```bash
gclient sync -D
```

### Build Chromium

1. Prepare a build directory, for example `out/Default`:
   ```bash
   ```

gn gen out/Default

```

2. Edit the file `./out/Default/args.gn` like this:
	- For a debug build
		```bash
target_os = "android"
target_cpu = "arm64"
is_debug = true
is_component_build = false
use_allocator_shim = true
blink_symbol_level = 0
ffmpeg_branding="Chrome"
proprietary_codecs=true
```

```
- For a release build
	```bash
```

target_os = "android"
target_cpu = "arm64"
is_official_build = true
ffmpeg_branding="Chrome"
proprietary_codecs=true

# exclude_unwind_tables = false # Optional, allows to get stack traces on release builds.

```

3. To resolve the `fatal error: 'content/shell/grit/shell_resources.h' file not found` error, you need to execute:
	```bash
autoninja -C out/Default content/shell:content_shell_resources_grit
```

4. Build Command (this might take a lot of time):
   ```bash
   ```

autoninja -C out/Default content_aar ui_aar

```

5. There are known issues regarding using AARs solely. so please do: 
	```bash
./fix_aar.sh out/Default/Content.aar && ./fix_aar.sh out/Default/ChromiumUi.aar
```

This will generate the new correct AAR files in your `chromium/src` root directory (**not in** **out/Default**).

1. Copy resources to `chromium/src/chromium_aar`:
   ```bash
   ```

mkdir -p chromium_aar/assets/
cp Content.aar chromium_aar/
cp ChromiumUi.aar chromium_aar/
cp out/Default/snapshot_blob.bin chromium_aar/assets/snapshot_blob_64.bin
cp out/Default/icudtl.dat chromium_aar/assets/
cp out/Default/wolvic.pak chromium_aar/assets/

```

```
