# 1. Set up Build Environment For MasOS
### 1.1 Select and Update OS 

On macOS Mojave or later, select **0System Preferences > Software Update**. Click Update Now if necessary

### 1.2 Install dependencies

- Open **Terminal** and run the following command to install Homebrew
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
- After the Homebrew installation script completes, follow the on-screen instructions to add the Homebrew installation to the path.
    
-   On macOS running on Apple Silicon, this is achieved with:
```
(echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> ~/.zprofile
source ~/.zprofile
```
- On macOS running on Intel, use the command for Apple Silicon, but replace **/opt/homebrew/** with **/usr/local/**.
- Use **brew** to install the required dependencies:
```
brew install cmake ninja gperf python3 python-tk ccache qemu dtc libmagic wget openocd
```
- Add the Homebrew Python folder to the path, in order to be able to execute **python** and **pip** as well **python3** and **pip3**
```
(echo; echo 'export PATH="'$(brew --prefix)'/opt/python/libexec/bin:$PATH"') >> ~/.zprofile
source ~/.zprofile0
```


### 1.3 Install Python3.10

```
brew install python@3.10
```

### 1.4 Install West
- Create a new virtual environment with **python3.10**:
```
cd $HOME
python3.10 -m venv ~/zephyrproject/.venv
```
- Activate the virtual environment (**Remember to activate the virtual environment every time you start working**)
```
source ~/zephyrproject/.venv/bin/activate
```
- Install West:
```
pip install west
pip install --upgrade pyyaml
```


### 1.5 Install nRF Command Line Tools

Download and install tools for MacOS following link:  **https://www.nordicsemi.com/Products/Development-tools/nRF-Command-Line-Tools/Download**

### 1.6 Get Zephyr source code
```
west init ~/zephyrproject
cd ~/zephyrproject
west update
```

### 1.7 Export a Zephyr CMake package
```
west zephyr-export
west packages pip --install
```

### 1.8 Install the Zephyr SDK
```
cd ~/zephyrproject/zephyr
west sdk install
```

# 2. Set up the Project 
- First, open a **terminal**  and activate the build environment:
```
cd $HOME
source ~/zephyrproject/.venv/bin/activate
```
- Create a `project` directory inside `zephyrproject`:
```
cd $HOME/zephyrproject/
mkdir project 
```
- Clone the project repository into the `project` directory
```
cd project
git clone git@github.com:fieldlabsinc/firmware-phamvanthanh.git
```
- Replace the following files with the ones from the repository:
    - `zephyrproject\zephyr\boards\seeed\xiao_ble\xiao_ble_common.dtsi` to `doc/xiao_ble_common.dtsi`
    - `zephyrproject\zephyr\boards\seeed\xiao_ble\xiao_ble.dts` to `doc/xiao_ble.dts`
    - `zephyrproject\zephyr\boards\seeed\xiao_ble\xiao_ble_defconfig` to `doc/xiao_ble_defconfig` 


# 3. Build and Flash
- Build the project (with MCUboot)
```
cd firmware-phamvanthanh
west build -p always --pristine -b xiao_ble/nrf52840 --sysbuild
```
- For the initial flashing, use J-Link and connect the debugger to the required pins (or use an nRF5340 DK board):

![Alt text](doc/SWD1.png)

- Run the following command to flash bootloader and application (install `nrfjprog` if necessary)
 - Check version nrfjprog
```
nrfjprog --version
```
 - Flash bootloader and application by nrfjprog
```
west flash --runner nrfjprog --erase 
```



# 4. Device Firmware Upgrade (OTA)
- Use the `nRF Device Manager` application to flash the firmware over the air.
- Connect to  the device named `Compass` in `nRF Device Manager`

![Alt text](doc/ota1.png)

- Select the firmware image file from build directory: `zephyrproject\project\firmware-phamvanthanh\build\firmware-phamvanthanh\zephyr\zephyr.signed.bin` and press **Start**

![Alt text](doc/ota2.png)

**Firmware Upgrade Modes**

- **Test Only**: Flashes the new firmware, resets the device, and runs the new firmware. However, after a reset, the device will revert to the old firmware.

- **Test and Confirm**: Flashes the new firmware, resets the device, and runs the new firmware permanently.

![Alt text](doc/ota3.png)

- After uploading, wait **30 seconds to 1 minute** for the firmware to reset and swap image slots.

# 5. Swapping Image Slots (A/B FOTA)

- To swap firmware images, open `nRF Device Manager`, navigate to **Advanced > Read Images > Confirm**.

![Alt text](doc/ota4.png)

- Press the reset button **OR** long-press the user button for 10 seconds to complete the image swap.

