# ZMK Keyboard for  Cornix

This community firmwarw has been tested with Cornix using ZMK and provides full split-role configuration, battery power management, and Bluetooth central/peripheral setup per ZMK split guidelines 


![image](images/cornix_with_dongle.png)
![image](images/cornix_layout.png)

## TODO LIST

- [x] 52 keys full layout keymap, since v2.0
- [x] ec11 encoder, since v2.2
- [x] no-SD image, since v2.3
- [x] rgb since v3


### about RGB

Cornix shield has 2 RGB LEDs on each side (4 LEDs total), controlled by WS2812 via SPI.

The RGB LED indicator module is **now enabled by default** in this repository! The LEDs provide visual feedback for:

**Battery Status Indicators:**
- Green: 80-100% battery
- Yellow: 50-79% battery
- Orange: 20-49% battery
- Red: 0-19% battery
- Blue: Currently charging

**Connection Status Indicators:**
- Solid color: Connected to host
- Slow blink: Disconnected/Searching for connection
- Fast blink: Pairing mode

**LED Configuration:**
- Brightness: 64/255 (default, optimized for battery life)
- LED count: 2 per side
- Pin mapping: Left (P0.24), Right (P0.13)

To adjust brightness or disable LEDs, edit `boards/shields/cornix_indicator/cornix_indicator.conf` and modify `CONFIG_RGBLED_WIDGET_BRIGHTNESS` (0-255).

**Note:** LED indicators consume additional power. If you experience reduced battery life, you can disable them by commenting out the `shield: cornix_indicator` lines in `build.yaml`.

## Performance Tuning

This firmware has been optimized for minimal latency. Here's what has been tuned:

### Current Optimizations

**Matrix Scanning:**
- Poll interval: 2ms (5x faster than default 10ms)
- Location: `boards/arm/cornix/cornix.dtsi:26`

**Debouncing:**
- Press debounce: 1ms (3x faster than previous 3ms)
- Release debounce: 5ms (balanced for stability)
- Location: `boards/arm/cornix/Kconfig.defconfig:26-28`

**Bluetooth Connection:**
- Min interval: 6ms (7.5ms actual)
- Max interval: 9ms (11.25ms actual)
- Connection latency: 10 (previously 30)
- Location: `boards/arm/cornix/cornix_*_defconfig`

**Processing:**
- Work queue priority: -2 (higher priority for faster HID)
- BLE buffer sizes: optimized for 251 bytes
- USB HID polling: 1ms (1000Hz)

### Fine-Tuning Options

If you want to experiment further, you can adjust these settings:

**For even lower latency (may reduce stability):**
```kconfig
# In cornix_*_defconfig files:
CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=0    # No debounce (risky - may cause double-taps)
CONFIG_BT_PERIPHERAL_PREF_LATENCY=0      # Zero latency mode (uses more power)
```

**If experiencing key chatter or double-taps:**
```kconfig
# In cornix_*_defconfig files:
CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=2    # Slightly more stable
CONFIG_ZMK_KSCAN_DEBOUNCE_RELEASE_MS=7  # More stable release
```

**For better battery life at cost of latency:**
```kconfig
# In cornix.dtsi:
poll-interval-ms = <5>;  # Less aggressive scanning

# In cornix_*_defconfig:
CONFIG_BT_PERIPHERAL_PREF_MAX_INT=12    # Longer intervals
CONFIG_BT_PERIPHERAL_PREF_LATENCY=30    # More power efficient
```

### Additional Enhancement Suggestions

Consider these ZMK features not yet enabled in this repo:

1. **NKRO (N-Key Rollover)** - For gaming or fast typing
   ```kconfig
   CONFIG_ZMK_HID_REPORT_TYPE_NKRO=y
   ```

2. **Combo performance** - Faster combo detection
   ```kconfig
   CONFIG_ZMK_COMBO_MAX_COMBOS_PER_KEY=16
   CONFIG_ZMK_COMBO_MAX_KEYS_PER_COMBO=4
   ```

3. **Deep sleep optimization** - Disable if you want instant wake
   ```kconfig
   CONFIG_ZMK_SLEEP=n  # Disable sleep for instant response
   ```

## Supported Hardware: Cornix Split Keyboard

Cornix Split Tented Low‑Profile Ergo Keyboard (Jezail Funder)

Cornix is a Corne‑inspired split ergonomic keyboard featuring a compact 3×6 column‑staggered layout with six thumb‑cluster keys (three per half). It offers adjustable tenting angles at 10°, 18°, and 25°, allowing users to reduce wrist strain and find a custom ergonomic alignment

- **Split, column‑staggered layout** (3×6 + thumb cluster layout).
- **Adjustable tenting support** at 10°, 18°, 25° (hardware‑based, no firmware hacks).
- **Kailh Choc V2 hot‑swap sockets** and support for LAK or LCK low‑profile keycaps.
- **Dual‑mode connectivity**: Wired USB‑C or Bluetooth wireless (left half as master).
- **Firmware**: Fully VIAL‑supported for keymaps and layer customization, stock firmware is RMK.
- Premium **CNC‑machined aluminum chassis**, custom damping foam, and portable storage pouch.

> this project owner is RMK contributor too, support RMK https://rmk.rs/ please 

## Improvements in This Fork

This fork includes several enhancements over the original repository:

### Performance Optimizations (Low Latency)
- **Ultra-fast debouncing** - Reduced press debounce from 3ms to 1ms for instant key response
- **Aggressive matrix polling** - Reduced scan interval from 10ms to 2ms for faster key detection
- **Optimized BLE connection** - Tighter connection intervals (6-9ms) and reduced latency (10 vs 30)
- **Fast HID processing** - Higher priority work queue and optimized BLE buffer sizes
- **1ms USB polling** - When using wired mode, USB HID reports at 1000Hz

**Expected latency improvements:**
- Debounce: 2-3ms faster
- Matrix scan: ~8ms faster
- BLE connection: ~15-20ms lower average latency
- **Total improvement: 25-31ms lower end-to-end latency**

### Bluetooth Connectivity
- **Enhanced BLE transmit power** - Increased to +8dBm for better range and reliability
- **Improved connection stability** - Disabled 2Mbps PHY for better compatibility with various Bluetooth chipsets
- **Optimized connection intervals** - Tuned parameters for more reliable and responsive split keyboard communication

### Keymap Customizations
- **Removed home row mods** - Simplified typing experience by removing home row modifiers
- **Enhanced combos** - Added useful key combinations:
  - Copy/paste combos for faster workflow
  - Mouse button (MB1) combo
  - Caps Word support
  - Common shortcuts: Cmd+W, Cmd+A, Cmd+T
  - Window movement keys
  - Hyper key combinations (Hyper+A/B/C/D)
- **Mouse integration** - Changed right shift to left click for integrated mouse support
- **Optimized key layout** - Refined key positions and fixed space/shift key behavior

### Build System & Code Cleanup
- **Streamlined builds** - Removed unused Cornix 42 configuration and artifacts to speed up CI/CD
- **Cleaner codebase** - Removed unused layers and unnecessary combos
- **Dead code removal** - Eliminated orphaned configuration files:
  - Removed unused `cornix_ph_left` board variant
  - Removed orphaned `cornix.conf` base config
  - Removed legacy `boards/arm/cornix/cornix.keymap` (superseded by `config/cornix.keymap`)

### Documentation
- **English comments** - Translated all Chinese comments to English for better accessibility

## --Bootloader Recovery Instructions--

-- The original RMK firmware removed the SoftDevice, so before flashing `zmk.uf2`, you need to restore the SoftDevice first. For specific steps, please refer to [bootloader/README.md](./bootloader/README.md). --

Since v2.3 this board' flash partitions has updated, removed SD (reducing sd partitionsize size from 150K to 4K), so You can flash firmware directly.

> You may need to reset fw by reset.uf2 from ealier version

> You can rollback to stock firmware by flash orgin uf2 file, backup files under rmkfw/

## 🔰 Easy Method: Clone This Repository and Build with GitHub Actions

If you’re new to ZMK and don’t want to deal with `west.yml` or module management, you can simply use this repository directly to customize your firmware.

### Steps

1. **Fork or Clone This Repository**
   - Click **Fork** in the top right to copy this repo to your GitHub account, or
   - Run `git clone` locally.

   > Forking is recommended, because GitHub Actions will automatically build your firmware.

2. **Edit Your Keymap**
   - Locate the keymap file in `config/cornix.keymap` (or whichever `.keymap` file you want to customize).
   - You can edit it directly or use the [ZMK Keymap Editor](https://nickcoutsos.github.io/keymap-editor/):
     - Open the editor and load your `.keymap` file.
     - Make changes with the visual editor.
     - Download the updated file and replace it in your repository.
     - Commit and push the changes to GitHub.

3. **Build with GitHub Actions**
   - After pushing, GitHub Actions will automatically run the build.
   - Once the workflow finishes, go to **Actions → your latest run → Artifacts** and download the firmware (`.uf2`) files.

4. **Flash Your Keyboard**
   - Put your board into UF2 bootloader mode (usually by double-tapping the reset button).
   - Drag and drop the `.uf2` file onto the mounted drive.

### Who Is This For?

- Beginners to ZMK  
- Users who only want to customize keymaps  
- Anyone who doesn’t need to modify drivers or hardware definitions

## How to build Cornix Zmk firmware from scratch

This section will guide you through building the Cornix ZMK firmware from scratch using the official ZMK firmware development process.


### Prerequisites

Before starting, ensure you have the following:
- A GitHub account
 Git installed on your system
- Basic understanding of Git and GitHub
- Your Cornix keyboard PCBs ready

### Step 1: Initialize ZMK Config Repository

1. **Create a new repository** using the official ZMK config template:
   - Visit: https://github.com/zmkfirmware/unified-zmk-config-template
   - Click "Use this template" → "Create a new repository"
   - Name your repository (e.g., `cornix-zmk-config`)
   - Choose "Public" or "Private" as preferred
   - Click "Create repository"

2. **Clone your new repository locally**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
   cd YOUR_REPO_NAME
   ```

3. **Initialize ZMK development environment**:
   ```bash
   west init -l config/
   west update
   west zephyr-export
   ```

> **Important**: You should thoroughly read the ZMK documentation before proceeding, as ZMK firmware development has a learning curve.
> - ZMK Customization Guide: https://zmk.dev/docs/customization
> - ZMK Configuration: https://zmk.dev/docs/user-setup

### Step 2: Add Cornix Shield to Your Project

After initializing your zmk-config repository, follow the steps in the next section to integrate the Cornix shield.

## How to Add Cornix Shield to Existing ZMK Project

For users with existing zmk-config, add this repository dependency via west.yml and pull the latest version via west update:

### 1. Modify west.yml

Edit the `config/west.yml` file, add to the `manifest/remotes` section:

```yaml
remotes:
  - name: zmkfirmware
    url-base: https://github.com/zmkfirmware
  - name: cornix-shield
    url-base: https://github.com/hitsmaxft
  - name: urob
    url-base: https://github.com/urob
```

Add to the `manifest/projects` section:

```yaml
projects:
  - name: zmk
    remote: zmkfirmware
    revision: main
    import: app/west.yml
  - name: zmk-keyboard-cornix
    remote: cornix-shield
    revision: main
  - name: zmk-helpers
    remote: urob
    revision: main
```

### 2. Update Dependencies

```bash
west update
```

### 3. Configure Build

Edit the `build.yaml` file, add:

> [!NOTE]
> 1. If you are using (default) cornix without dongle, choose "cornix_left", "cornix_right" and "reset".
> 2. If you are using cornix with dongle, choose "cornix_dongle". "cornix_left_for_dongle", "cornix_right" and "reset".
> 3. Add "cornix_indicator" shield to enable RGB led light. It consumes much more power, use at your own risk.

```yaml
include:
  # Use cornix with dongle
  - board: cornix_dongle
    shield: cornix_dongle_eyelash dongle_display
    snippet: studio-rpc-usb-uart
    artifact-name: cornix_dongle

  - board: cornix_ph_left
    # shield: cornix_indicator
    artifact-name: cornix_left_for_dongle

  # Use cornix without dongle
  - board: cornix_left
    # shield: cornix_indicator
    artifact-name: cornix_left

  - board: cornix_right
    # shield: cornix_indicator
    artifact-name: cornix_right

  - board: cornix_right
    shield: settings_reset
    artifact-name: reset
```

### 4. Build Firmware

Use your preferred method to build

- no need to recovery the sd since 2.3
- falsh reset.uf2 both side of cornix
- flash left and right uf2 files
- reset both side at the same time.

### 5. Flash Firmware

Flash the generated `.uf2` files to the corresponding microcontroller:
- Left half: `build/left/zephyr/zmk.uf2`
- Right half: `build/right/zephyr/zmk.uf2`

## Build This Project Locally (Without west.yaml Dependency)

If you prefer to build this project locally without adding it as a dependency in your west.yaml, you can use the ZMK_EXTRA_MODULES cmake argument.

### Prerequisites

1. Have a working ZMK development environment set up
2. Clone this repository to a local directory

### Build Steps

1. **Clone this repository**:
   ```bash
   git clone https://github.com/hitsmaxft/zmk-keyboard-cornix.git
   ```

2. **Configure your ZMK build with the extra module**:
   
   Edit your `.west/config` file and add the cmake argument under the `[build]` section:
   
   ```ini
   [build]
   cmake-args = -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -DZMK_EXTRA_MODULES=/full/absolute/path/to/zmk-keyboard-cornix
   ```
   
   Replace `/full/absolute/path/to/zmk-keyboard-cornix` with the actual absolute path where you cloned this repository.

3. **Build the firmware**:
   ```bash
   west build -b cornix_e73 -- -DSHIELD=cornix_main_left
   west build -b cornix_e73 -- -DSHIELD=cornix_right
   ```

This method allows you to use the Cornix shield without modifying your existing ZMK configuration's west.yaml file.
