# Compiling TaintDroid #

Start with:
```
% mkdir -p ~/tdroid/tdroid-2.1_r2.1p
% cd ~/tdroid/tdroid-2.1_r2.1p
% repo init -u git://android.git.kernel.org/platform/manifest.git -b android-2.1_r2.1p
% repo sync
... wait

% . ./build/envsetup.sh
% lunch 1
% make -j4
... wait
% emulator
... ensure the build works
```

Download the TaintDroid local\_manifest.xml (http://droidbox.googlecode.com/files/local_manifest.xml) and place it in _~/tdroid/tdroid-2.1\_r2.1p/.repo_.

Next, pull the source code and make sure we are working with the right version.

```
% cd ~/tdroid/tdroid-2.1_r2.1p
% repo sync
% cd dalvik
% git branch --track taintdroid-2.1_r2.1p github/taintdroid-2.1_r2.1p
% git checkout taintdroid-2.1_r2.1p
% git pull # (just to be safe)
% cd ..
% cd frameworks/base
% git branch --track taintdroid-2.1_r2.1p github/taintdroid-2.1_r2.1p
% git checkout taintdroid-2.1_r2.1p
% git pull # (just to be safe)
```

ext2 support is not currently enabled for the SDcard; however the code exists in android-2.1\_r2.1p. The following simple patch to the _system/core_ directory enable ext2/ext3 support for the SDcard and ensures it is mounted with user\_xattr. It is probably easiest to make the changes by hand as opposed to applying the change as a patch.

```
diff --git a/vold/volmgr.c b/vold/volmgr.c
index deb680e..a5f5789 100644
--- a/vold/volmgr.c
+++ b/vold/volmgr.c
@@ -43,7 +43,7 @@ static volume_t *vol_root = NULL;
static boolean safe_mode = true;

static struct volmgr_fstable_entry fs_table[] = {
-// { "ext3", ext_identify, ext_check, ext_mount , true },
+ { "ext3", ext_identify, ext_check, ext_mount , true },
    { "vfat", vfat_identify, vfat_check, vfat_mount , false },
    { NULL, NULL, NULL, NULL , false}
};
diff --git a/vold/volmgr_ext3.c b/vold/volmgr_ext3.c
index fe3b2bb..8979c70 100644
--- a/vold/volmgr_ext3.c
+++ b/vold/volmgr_ext3.c
@@ -157,7 +157,8 @@ int ext_mount(blkdev_t *dev, volume_t *vol, boolean safe_mode)

    char **f;
    for (f = fs; *f != NULL; f++) {
- rc = mount(devpath, vol->mount_point, *f, flags, NULL);
+ //rc = mount(devpath, vol->mount_point, *f, flags, NULL);
+ rc = mount(devpath, vol->mount_point, *f, flags, "user_xattr");
        if (rc && errno == EROFS) {
            LOGE("ext_mount(%s, %s): Read only filesystem - retrying mount RO",
    devpath, vol->mount_point);
```

```
% cd ~
% wget http://droidbox.googlecode.com/files/yaffs_xattr.patch
...
% cd ~/tdroid/tdroid-2.1_r2.1p
% git clone git://android.git.kernel.org/kernel/common.git
% cd common
% git branch --track android-gldfish-2.6.29 origin/archive/android-gldfish-2.6.29
% git checkout android-gldfish-2.6.29
% git pull # (just to be safe)
% patch -p1 < ~/yaffs_xattr.patch

...
% cd ~/tdroid/tdroid-2.1_r2.1p
% . ./build/envsetup.sh
% lunch 1 # (lunch 1 is fine for now)

...
% cd common
% export ARCH=arm
% export SUBARCH=arm
% export CROSS_COMPILE=arm-eabi-
% make goldfish_defconfig
% make oldconfig
% make menuconfig
# ... verify YAFFS and EXT2 with XATTR and SECURITY support

% make -j4
% cp arch/arm/boot/zImage ~/ # for later use

% cd ~/tdroid/tdroid-2.1_r2.1p
% edit/create buildspec.mk and copy/paste the following:
# Enable core taint tracking logic (always add this)
WITH_TAINT_TRACKING := true

# Enable taint tracking for ODEX files (always add this)
WITH_TAINT_ODEX := true

# Enable taint tracking in the "fast" (aka ASM) interpreter (recommended)
WITH_TAINT_FAST := true

# Enable addition output for tracking JNI usage (not recommended)
#TAINT_JNI_LOG := true


% . ./build/envsetup.sh
% lunch 1
% make clean
% make -j4
```

Next step is download the latest Android SDK and create an AVD with Android 2.1 and start the emulator with the compiled kernel and TaintDroid system image.

```
% emulator -avd name -kernel ~/zImage -ramdisk ~/tdroid/tdroid-2.1_r2.1p/out/target/product/generic/ramdisk.img -image ~/tdroid/tdroid-2.1_r2.1p/out/target/product/generic/system.img &
```

While booting up, issue:

```
% ../platform-tools/adb shell setprop dalvik.vm.execution-mode int:portable
```

# Applying patch #

Applying the patches on TaintDroid is done by issuing:

```
patch -p0 -i dalvik.patch
patch -p0 -i frameworks.patch
```
in _~/tdroid/tdroid-2.1\_r2.1p/_. It then needs to be compiled once again with:

```
% . ./build/envsetup.sh
% lunch 1
% make -j4
```

Copy the system image and run the emulator as described above.