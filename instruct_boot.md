### Hi, this is a note after testing those instructions; sadly it failed. But i keep it for good old memory. Please read it from ##Part 2: The damn working one.
1. Build the key for signing:
   ```bash
imgtool keygen -k mcuboot.pem -t rsa-2048
    ```
2. Create the mcuboot.conf:
    ```conf
CONFIG_BOOT_SIGNATURE_TYPE_RSA=y
CONFIG_BOOT_SIGNATURE_TYPE_RSA_LEN=2048
CONFIG_BOOT_SIGNATURE_KEY_FILE="./mcuboot.pem"
CONFIG_BOOT_VALIDATE_SLOT0=y
CONFIG_MCUBOOT_LOG_LEVEL_INF=y
    ```
3. Build the mcuboot once per project:
    ```bash
west build -b esp32s3_devkitc/esp32s3/procpu ~/zephyrproject/bootloader/mcuboot/boot/zephyr -d build-mcuboot -p always -- -DEXTRA_CONF_FILE=mcuboot.conf
    ```

4. Flash the mcuboot:
    ```bash
west flash -d build-mcuboot
    ```

5. Create a VERSION
    ```md
VERSION_MAJOR = 1
VERSION_MINOR = 0
PATCHLEVEL = 0
VERSION_TWEAK = 0
    ```

6. Build the file:
    ```bash
west build -b esp32s3_devkitc/esp32s3/procpu -- -DCONF_FILE="prj.conf;wifi.conf;http.conf;flash.conf;boot.conf"
    ```

7. Sign the new firmware:
    ```bash
west sign -d build -t imgtool -- --key mcuboot.pem
    ```

8. Flash it:
    ```bash
west flash
    ```

## Part 2: The damn working one:

1. Create a security pem:
   ```bash
imgtool keygen -k mcuboot.pem -t rsa-2048
    ```
2. Create sysbuild.conf:
    ```conf
SB_CONFIG_BOOTLOADER_MCUBOOT=y
SB_CONFIG_BOOT_SIGNATURE_KEY_FILE="\${APP_DIR}/mcuboot.pem"
SB_CONFIG_BOOT_SIGNATURE_TYPE_RSA=y
    ```
3. The flash.conf:
    ```conf
CONFIG_FLASH=y
CONFIG_FLASH_MAP=y
CONFIG_STREAM_FLASH=y
    ```

4. The boot.conf:
    ```conf
CONFIG_BOOTLOADER_MCUBOOT=y
CONFIG_CRC=y
CONFIG_MCUBOOT_IMG_MANAGER=y
CONFIG_IMG_MANAGER=y
CONFIG_REBOOT=y
CONFIG_MCUBOOT_BOOTUTIL_LIB=y
    ```
5. Now for the mock ota test; we will FIRST build the 1.0.0 first;then copy it to home dir, then name it to firmware.bin (this is signed with sysbuild of course). We then calc the crc32; then store it in the firmware.crc. This will have upgrade=false
    ```bash
west build --sysbuild -b esp32s3_devkitc/esp32s3/procpu \
-- -DCONF_FILE="prj.conf;wifi.conf;http.conf;flash.conf;boot.conf"
    ```
    ```bash
cp build/zephyr/zephyr.signed.bin ./firmware.bin
    ```
    ```bash
crc32 firmware.bin > firmware.crc
    ```
    ```bash
    cat VERSION     
VERSION_MAJOR = 1
VERSION_MINOR = 0
PATCHLEVEL = 0
VERSION_TWEAK = 0
    ```
6. We then build the 0.9.0 then; we need to change the VERSION, and set the upgrade to true; before build it; the build command is the same as above.

7. Then we just flash it:
    ```bash
west flash
    ```



