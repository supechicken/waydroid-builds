## Instructions on building LineageOS images for Waydroid

> [!NOTE]
> This instruction is based on https://docs.waydro.id/development/compile-waydroid-lineage-os-based-images
>
> Replace `x86_64` with `arm64` if you want to build for ARM64 platforms

### Requirements
- A Linux-based system (or WSL if you are using Windows)
- Enough storage space (~300GB)
- Time and patience, the whole process can take up ~5 hours depending on your hardware

### Install build dependencies
> [!WARNING]
> If you encounter building errors, check if the error can be solved with `apt install/pacman -S <package>` first, instead of opening an issue here

- Install `lineageos-devel` from AUR if you are using Arch Linux
```shell
yay -S lineageos-devel python-setuptools python-mako python-yaml
```

- For Ubuntu
```
apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git git-lfs gnupg gperf \
    imagemagick lib32readline-dev lib32z1-dev libelf-dev liblz4-tool libsdl1.2-dev libssl-dev libxml2 libxml2-utils \
    lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev lib32ncurses5-dev libncurses5 libncurses5-dev \
    python3-setuptools python3-mako python3-yaml
```

- Enable `ccache` (optional)
```shell
ccache -M 20G

export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
```

### Setup and patch LOS source
- Create a new directory and `cd` into it
```shell
mkdir android
cd android
```

- Setup LOS source **(for A14)**
```shell
repo init -u https://github.com/LineageOS/android.git -b lineage-21.0 --git-lfs
repo sync build/make

wget -O - https://raw.githubusercontent.com/Waydroid-ATV/android_vendor_waydroid/lineage-21/manifest_scripts/generate-manifest.sh | bash
```

- Setup LOS source **(for A15)**
```shell
repo init -u https://github.com/LineageOS/android.git -b lineage-22.2 --git-lfs
repo sync build/make

wget -O - https://raw.githubusercontent.com/WayDroid-ATV/android_vendor_waydroid/lineage-22.2/manifest_scripts/generate-manifest.sh | bash
```

- Sync and patch source
```shell
repo sync -j$(nproc --all)
. build/envsetup.sh

apply-waydroid-patches
```

- Start building

> [!NOTE]
> By default, the build will have these features enabled by default:
>   - Widevine L3 (remove **`ANDROID_USE_WIDEVINE := true`** from `device/waydroid/waydroid/waydroid_x86_64/lineage_waydroid_x86_64.mk` to disable it)
>   - ARM translation layer support (remove **`ANDROID_USE_INTEL_HOUDINI/ANDROID_USE_NDK_TRANSLATION := true`** from `device/waydroid/waydroid/waydroid_x86_64/lineage_waydroid_x86_64.mk` to disable it)
>   - GApps (remove **`$(call inherit-product, vendor/gapps/x86_64/x86_64-vendor.mk)`** from `device/waydroid/waydroid/waydroid_x86_64/lineage_waydroid_x86_64.mk` to disable it)

```shell
# replace `waydroid_x86_64` with appropriate architecture, if needed

lunch lineage_waydroid_x86_64-ap2a-userdebug # FOR A14 ONLY
lunch lineage_waydroid_x86_64-bp1a-userdebug # FOR A15 ONLY

make systemimage -j$(nproc --all)
make vendorimage -j$(nproc --all)
```
