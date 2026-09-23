# Kubuntu 25.10 gaming and dev setup

## Phase 0: System Preparation

### 1. First update
Recommended to do periodically.
```bash
sudo apt update && sudo apt upgrade
```

### 2. Remove and Purge Snaps
Before blocking the service via APT, you must remove all existing Snap packages and the daemon itself to reclaim disk space and prevent broken links.

#### 1. List and remove installed packages:
```bash
snap list
# Remove each package listed (e.g., firefox, gnome-46-2404, gtk-common-themes)
sudo snap remove --purge <package-name>
```
#### 2. Uninstall the Snapd daemon and clean system directories:
```bash
sudo apt remove --purge snapd -y
rm -rf ~/snap
sudo rm -rf /var/cache/snapd/
sudo rm -rf /var/snap
sudo rm -rf /var/lib/snapd
```

#### 3. Block Snaps
Create an APT preference file to prevent `snapd` from being reinstalled automatically, then update package list:
```bash
# Block snapd via APT preferences
sudo tee /etc/apt/preferences.d/nosnap.pref <<EOF
Package: snapd
Pin: release a=*
Pin-Priority: -10
EOF

# Update package lists
sudo apt update
```

#### 4. Install Flatpak
```bash
# Update your package index
sudo apt update

# Install the core Flatpak utility and the Discover backend integration
sudo apt install -y flatpak plasma-discover-backend-flatpak

# Add Flathub to the remote list for all users on the system
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Reboot your system to apply the new environment variables
sudo reboot
```

## Phase I: Drivers & Core Dependencies

### 1. Enable 32-bit Architecture & Install Build Tools
Steam and some older games require 32-bit compatibility libraries.
```bash
sudo dpkg --add-architecture i386
sudo apt update && sudo apt upgrade -y
sudo apt install linux-headers-$(uname -r) build-essential dkms mesa-utils mesa-vulkan-drivers mesa-vulkan-drivers:i386 -y
```

### 2. Enable NTSYNC Kernel Module
The `ntsync`module provides advanced synchronization primitives that significantly improve performance for Windows games running through Wine/Proton.
```bash
echo 'ntsync' | sudo tee /etc/modules-load.d/ntsync.conf
sudo modprobe ntsync
```

## Phase II: Applications & Utilities

### 1. Mozilla Firefox (Official APT Repository)
Because Ubuntu/Kubuntu replaces the default `firefox` APT package with a transitional package that installs the Snap version, we must explicitly add Mozilla's repository and set its priority to ensure you receive the native `.deb` package.
```bash
# 1. Create keyring directory and download the Mozilla signing key
sudo install -d -m 0755 /etc/apt/keyrings
wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg -O- | sudo tee /etc/apt/keyrings/packages.mozilla.org.asc > /dev/null

# 2. Add the Mozilla APT repository using the modern deb822 format
cat <<EOF | sudo tee /etc/apt/sources.list.d/mozilla.sources
Types: deb
URIs: https://packages.mozilla.org/apt
Suites: mozilla
Components: main
Signed-By: /etc/apt/keyrings/packages.mozilla.org.asc
EOF

# 3. Configure APT priority to strictly prefer the native Mozilla packages
cat <<EOF | sudo tee /etc/apt/preferences.d/mozilla
Package: *
Pin: origin packages.mozilla.org
Pin-Priority: 1000
EOF

# 4. Update the package index and install native Firefox
sudo apt update && sudo apt install firefox -y
```

### 2. Steam & Gamemode
```bash
sudo apt install steam-installer gamemode -y
```

### 3. Mangohud & Goverlay
```bash
# Update your package index
sudo apt update

# Install the overlay, the GUI manager, and the testing tools
sudo apt install -y mangohud goverlay vulkan-tools mesa-utils
```

### 4. OBS Studio (Official PPA)
```bash
sudo add-apt-repository ppa:obsproject/obs-studio -y
sudo apt update
sudo apt install obs-studio -y
```

### 5. Visual Studio Code (Microsoft APT Repository)
```bash
# Update package index and install dependencies
sudo apt update
sudo apt install software-properties-common wget gpg apt-transport-https

# Import the Microsoft GPG key
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg &&
sudo install -D -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/microsoft.gpg &&
rm -f microsoft.gpg

# Add the VS Code repository to your system
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/usr/share/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'

# Update package cache and install VS Code:
sudo apt update
sudo apt install code
```

### 6. Zoom 
```bash
# 1. Download the latest Zoom Debian package directly from their official server
wget https://zoom.us/client/latest/zoom_amd64.deb

# 2. Install the package using APT to handle dependencies natively
sudo apt install -y ./zoom_amd64.deb

# 3. Clean up the downloaded installation file to keep your directory tidy
rm zoom_amd64.deb
```

## Phase III: Development Tools

### 1. C/C++ Build Essentials
This installs the GNU Compiler Collection (GCC), G++, and make, which are fundamental for compiling software from source.

```bash
sudo apt update
sudo apt install -y build-essential gdb cmake
```

### 2. Python & Package Management
Kubuntu 25.10 comes with Python 3, but these tools add flexible environment management.

#### Standard Python Tools
You will need `pip` (package manager) and `venv` for isolated environments. On Debian/Ubuntu-based systems, the `venv` module must be installed explicitly to enable `ensurepip`, which bootstraps the environment.

```bash
# Update package index
sudo apt update

# Install pip and the specific venv module for the system's Python version
# This prevents the "ensurepip is not available" error
sudo apt install -y python3-pip python3.13-venv

# --- Usage Example ---
# 1. Create a virtual environment in the current directory
# python3 -m venv .venv

# 2. Activate the environment
# source .venv/bin/activate
```

#### Miniconda (Conda Environment Manager)
Ideal for data science, machine learning, or projects requiring specific non-Python dependencies.

```bash
# Download the latest Miniconda installer
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# Run the installer (Follow the prompts: press Enter and type 'yes')
bash Miniconda3-latest-Linux-x86_64.sh

# Cleanup installer and refresh shell
rm Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc
```

### 3. Node.js (via fnm)
We use `fnm` (Fast Node Manager) because it’s faster than `nvm`. It keeps your Node.js installs isolated and makes it easy to switch between versions without touching the system package manager.

```bash
# Install fnm (Fast Node Manager)
curl -fsSL https://fnm.vercel.app/install | bash

# Activate fnm (or restart your terminal)
source ~/.bashrc

# Install the latest LTS version of Node
fnm install --lts
fnm use lts-latest
```

### 4. Docker Engine
Official Docker repository setup to avoid the Snap-based version.

```bash
# Add Docker's official GPG key
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Allow running docker without sudo (Requires Logout/Login)
sudo usermod -aG docker $USER
```

## Phase IV: Gaming Setup

### 1. GEProton

#### Install ProtonUp-Qt (Flatpak)
```bash
# Install ProtonUp-Qt from Flathub
flatpak install -y flathub net.davidotek.pupgui2
```

(Note: If the application does not immediately appear in your KDE Plasma application menu, you can launch it manually from the terminal the first time by running `flatpak run net.davidotek.pupgui2`.)

#### Install GE-Proton using the GUI
ProtonUp-Qt provides a straightforward visual interface to handle the downloading and extraction process we previously did via the terminal.

1. Open ProtonUp-Qt from your application launcher.
2. In the main window, ensure the Install for: dropdown menu at the top is set to Steam.
3. Click the Add Version button located at the bottom of the window.
4. In the new popup window:
    - Compatibility Tool: Select GE-Proton.
    - Version: The latest stable release will be selected by default (e.g., GE-Proton-10).
5. Click Install.

#### Activation on Steam
Once the installation inside ProtonUp-Qt is completely finished, you must apply the new tool inside Steam.

1. Fully restart the Steam client. Steam only scans for new compatibility tools during its initial startup.
2. Open your Steam Library.
3. Right-click on the specific game you want to run with GE-Proton and select Properties.
4. Navigate to the Compatibility tab.
5. Check the box that says Force the use of a specific Steam Play compatibility tool.
6. Click the dropdown menu and select the GE-Proton version you just installed.

### 2. Global Environment Variables
Use systemd's `/etc/environment.d/` directory to set variables system-wide. This applies them early, before apps start, so KDE Plasma, Steam, and Proton all inherit the same settings.

```bash
sudo tee /etc/environment.d/gaming-config.conf <<EOF
# Prefer Wayland for Electron apps
ELECTRON_OZONE_PLATFORM_HINT=wayland

# Enable NTSYNC for Wine and Proton
WINE_USE_NTSYNC=1
PROTON_USE_NTSYNC=1

# Enable HDR support in Proton
PROTON_ENABLE_HDR=1

# Enable GE-Proton Wayland support (experimental)
# PROTON_ENABLE_WAYLAND=1

# Optional upscaling features, choose what matches your GPU
# PROTON_FSR4_UPGRADE=1
# PROTON_DLSS_UPGRADE=1
# PROTON_XESS_UPGRADE=1
EOF
```

### 3. Gamemode
To enable gamemode for your Steam games, you need to add a launch option to each game:

**For Individual Games:**
1. Open your Steam Library
2. Right-click on the game you want to optimize
3. Select **Properties**
4. In the **Launch Options** field, add:
   ```
   gamemoderun %command%
   ```
5. Close the properties window and launch the game

### 4. Mangohud

#### Configure using GOverlay
GOverlay provides a graphical interface to customize MangoHud without needing to manually edit configuration files.

1. Open **GOverlay** from your application launcher.
2. Navigate to the **MangoHud** tab.
3. Use the visual interface to toggle the metrics you want to monitor during gameplay (e.g., FPS, Frame Timing, CPU/GPU Temperature, RAM usage).
4. Click the **Save** button at the bottom right to generate your configuration profile.

#### Testing
Before launching a game, you can verify the overlay works using the testing tools we installed in Phase II.
```bash
# Test MangoHud using a basic Vulkan application
mangohud vkcube
```

#### Activation
To enable the overlay in your games, you must add it to the Steam Launch Options. If you are already using Gamemode (configured in Step 3), you can chain these commands together seamlessly.

1. Right-click the game in your Steam Library and select Properties.
2. In the Launch Options field, chain the commands separated by a space:
    ```bash
    mangohud gamemoderun %command%
    ```

## Phase V: KDE Plasma & System Tweaks

### Display
Configuring your display properly in Wayland is critical to eliminate screen tearing and minimize input lag.

1. Open System Settings and navigate to Display & Monitor.
2. **Refresh rate**: Select the absolute highest available option for your monitor (e.g., 144 Hz, 165 Hz).
3. Adaptive sync (FreeSync or Gsync): Select Automatic.
    - *Didactic Note*: In Wayland, "Automatic" enables Variable Refresh Rate (VRR) exclusively when an application is full-screen. This is the optimal setting, as setting it to "Always" forces VRR on standard desktop windows, which can cause erratic screen flickering.
4. **Screen tearing**: Check Allow in fullscreen windows. This permits the Wayland compositor to drop vsync protocols in competitive games for the lowest possible latency.
5. Click **Apply**.

### Mouse
Mouse acceleration dynamically changes your cursor speed based on how fast you move the physical mouse. This ruins muscle memory in first-person shooters. You must enforce raw, 1:1 input.

1. Open **System Settings** and go to **Mouse and Touchpad > Mouse**.
2. Select the **Device** corresponding to your mouse model.
3. Uncheck the **Enable pointer acceleration** box.
4. Click **Apply**.

### Power Management
Modern processors dynamically scale their clock speeds down to save battery. For gaming, you want to ensure the CPU governor is willing to boost to its maximum frequency immediately.

1. Click the **Power Management** icon in your system tray (bottom right).
2. Drag the Power Profile slider all the way to the right to select Performance.
    - **Note**: If you are on a laptop, ensure the device is plugged into wall power, as the Performance profile will drain the battery extremely fast.

### Background Services

#### Disable File Indexing (Baloo)
KDE Plasma uses a background service called Baloo to constantly index your files for rapid searching. This service can unexpectedly spike CPU usage and disk I/O while you are playing a game.

1. Open **System Settings**.
2. Navigate to **File Search**.
3. Uncheck **File Indexing** to completely disable background indexing.
4. Click **Apply**.
    - (Note: You will still be able to search for files manually; it will just take a few seconds longer, which is a worthy trade-off for stable gaming frame rates.)

#### Disable Unnecessary Services
KDE Plasma includes several optional background services that are safe to disable if you do not use them. Turning off the ones you do not need can reduce idle CPU work, background notifications, and small bursts of disk activity while gaming.

Search for `Background Service` in **System Settings > Background Services**, then disable the entries you do not use.

| Service Name | What it does | Should it be disabled? |
| :--- | :--- | :--- |
| **Accent Color** | Applies extracted accent color from wallpaper on request. | **Yes**, unless you want your UI colors to change with your wallpaper. |
| **Application menus daemon** | Transfers application's menu to the desktop. | **Yes**, unless you use a "Global Menu" widget on your panel. |
| **Audio Shortcuts** | Provides shortcuts for audio device and volume control. | **No**, essential for keyboard volume keys. |
| **Automatic Location for Night Light** | Provides location updates for Night Light. | **Yes**, unless you use the Night Light feature and travel across time zones often. |
| **Bluetooth** | Handles Bluetooth events. | **No**, unless you strictly never use Bluetooth devices. |
| **Device Notifications** | Play audible feedback when devices are inserted or removed. | **Yes**, if you find USB plug/unplug sounds annoying. |
| **Donation Request** | Shows a yearly request to donate to KDE. | **Yes**. |
| **Free Space Notifier** | Warns when running out of space on your home folder. | **No**, it uses negligible resources and is a good safety net. |
| **GNOME/GTK Settings Synchronization Service** | Automatically applies settings to GNOME/GTK applications. | **No**, keeps non-KDE apps (like Firefox) matching your system theme. |
| **inotify** | Monitors inotify resources for signs of exhaustion. | **No**, critical for software development tools and IDEs that watch for file changes. |
| **Kameleon** | Synchronizes device LEDs with the Plasma accent color. | **Yes**, unless you want your keyboard RGB to match your wallpaper. |
| **Keyboard Daemon** | Enables switching keyboard layout through shortcuts or system tray. | **No**, essential for stable custom key-repeat rates and layout switching. |
| **KScreen 2** | Screen management. | **No**, critical for external monitors and 2-in-1 screen rotation. |
| **Location-based System Time Zone** | Automatically change the system time zone based on the current location. | **Yes**, unless you travel frequently with the laptop. |
| **Media Controller** | Controls media players using global keyboard shortcuts. | **No**, allows keyboard play/pause/skip keys to work. |
| **Operating System Release Notifier** | Checks for new system releases and notifies when available. | **Yes**, if you prefer to check for OS upgrades manually. |
| **Out of Memory Notifier** | Informs when applications get terminated to prevent out of memory conditions. | **No**, helpful diagnostic tool for heavy workloads. |
| **Plasma Browser Integration Flatpak Integration** | Automatically enables support for Firefox Flatpak. | **Yes**, unless you use the Flatpak version of Firefox. |
| **Plasma Browser Integration Installation Reminder** | Provides a link to the browser extension if the host is installed. | **Yes**, it is just a nag screen. |
| **Plasma Network Management module** | Provides secrets to the NetworkManager daemon. | **No**, essential for remembering Wi-Fi and VPN passwords. |
| **Plasma Vault module** | Provides encrypted vaults. | **Yes**, unless you actively use KDE's encrypted folder feature. |
| **Print Manager** | Notify when a new printer is detected, or there are problems. | **Yes**, if you do not use a printer. |
| **Proxy Verifier** | Verifies proxy setup is performing as expected. | **Yes**, unnecessary for standard home network setups. |
| **Remote URL Change Notifier** | Provides change notification for network folders. | **Yes**, unless you actively work off FTP/SSH network drives. |
| **Removable Device Automounter** | Automatically mounts devices as needed. | **Optional**, yes if you prefer to manually click USB drives to mount them. |
| **Search Folder Updater** | Allows automatic updates of Search Folders. | **Yes**, unless you use saved search queries in the Dolphin file manager. |
| **Session Shortcuts** | Provides shortcuts for common session actions. | **No**, handles standard shortcuts like locking the screen or logging out. |
| **SMART** | Storage device health monitoring. | **No**, essential early warning system for drive failure. |
| **SMB Watcher** | Monitors directories on the smb:/ protocol for changes. | **Yes**, unless you use Windows/Samba shared network folders. |
| **Status Notifier Manager** | Manages services that provide status notifier user interfaces. | **No**, essential for system tray icons to appear and function. |
| **Thunderbolt Device Monitor** | Thunderbolt Device Monitor. | **Yes**, if your hardware does not have a Thunderbolt port. |
| **Time Zone** | Provides the system's time zone to applications. | **No**, essential for accurate application timestamps. |
| **Touchpad** | Enables or disables touchpad. | **No**, required if you use a hardware shortcut/Fn key to toggle the touchpad. |
| **Write Daemon** | Watch for messages from local users sent with write(1) or wall(1). | **Yes**, legacy service for multi-user terminal messaging. |
