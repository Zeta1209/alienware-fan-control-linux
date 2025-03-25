# Alienware Fan Control for Linux

This repository contains scripts and configurations to manage fan speeds on Alienware laptops running Linux. It provides custom fan profiles for different usage scenarios.

## Prerequisites

- An Alienware laptop (tested on M17 R3)
- Linux distribution with `lm-sensors` and `fancontrol` support (tested on Pop!_OS)

## Installation

### 1. Install the required packages

```bash
sudo apt update
sudo apt install lm-sensors fancontrol
```

### 2. Detect and configure your sensors

```bash
sudo sensors-detect --auto
```

### 3. Check if your fans are detected

```bash
sensors
```

Look for `dell_smm-virtual-0` in the output, which should show your fan speeds.

### 4. Configure fancontrol

```bash
sudo pwmconfig
```

Follow the prompts to identify your fans. When asked to select temperature sensors, choose the appropriate CPU and GPU temperature sensors.

## Fan Profiles Setup

### 1. Create a directory for fan profiles

```bash
sudo mkdir -p /etc/fancontrol.d
```

### 2. Create the balanced profile

```bash
sudo nano /etc/fancontrol.d/balanced
```

Add the following content:

```
# Balanced profile - good for everyday use
INTERVAL=10
DEVPATH=hwmon4=devices/platform/PNP0C14:06/wmi_bus/wmi_bus-PNP0C14:06/F1DDEE52-063C-4784-A11E-8A06684B9B01
DEVNAME=hwmon4=dell_smm
FCTEMPS=hwmon4/pwm1=hwmon4/temp1_input hwmon4/pwm3=hwmon4/temp2_input
FCFANS=hwmon4/pwm1=hwmon4/fan1_input hwmon4/pwm3=hwmon4/fan3_input
MINTEMP=hwmon4/pwm1=45 hwmon4/pwm3=45
MAXTEMP=hwmon4/pwm1=85 hwmon4/pwm3=85
MINSTART=hwmon4/pwm1=40 hwmon4/pwm3=40
MINSTOP=hwmon4/pwm1=20 hwmon4/pwm3=20
MINPWM=hwmon4/pwm1=0 hwmon4/pwm3=0
MAXPWM=hwmon4/pwm1=255 hwmon4/pwm3=255
```

### 3. Create the performance profile

```bash
sudo nano /etc/fancontrol.d/performance
```

Add the following content:

```
# Performance profile - for gaming and heavy loads
INTERVAL=10
DEVPATH=hwmon4=devices/platform/PNP0C14:06/wmi_bus/wmi_bus-PNP0C14:06/F1DDEE52-063C-4784-A11E-8A06684B9B01
DEVNAME=hwmon4=dell_smm
FCTEMPS=hwmon4/pwm1=hwmon4/temp1_input hwmon4/pwm3=hwmon4/temp2_input
FCFANS=hwmon4/pwm1=hwmon4/fan1_input hwmon4/pwm3=hwmon4/fan3_input
MINTEMP=hwmon4/pwm1=40 hwmon4/pwm3=40
MAXTEMP=hwmon4/pwm1=80 hwmon4/pwm3=80
MINSTART=hwmon4/pwm1=80 hwmon4/pwm3=80
MINSTOP=hwmon4/pwm1=60 hwmon4/pwm3=60
MINPWM=hwmon4/pwm1=60 hwmon4/pwm3=60
MAXPWM=hwmon4/pwm1=255 hwmon4/pwm3=255
```

### 4. Create the quiet profile

```bash
sudo nano /etc/fancontrol.d/quiet
```

Add the following content:

```
# Quiet profile - minimal fan noise
INTERVAL=10
DEVPATH=hwmon4=devices/platform/PNP0C14:06/wmi_bus/wmi_bus-PNP0C14:06/F1DDEE52-063C-4784-A11E-8A06684B9B01
DEVNAME=hwmon4=dell_smm
FCTEMPS=hwmon4/pwm1=hwmon4/temp1_input hwmon4/pwm3=hwmon4/temp2_input
FCFANS=hwmon4/pwm1=hwmon4/fan1_input hwmon4/pwm3=hwmon4/fan3_input
MINTEMP=hwmon4/pwm1=60 hwmon4/pwm3=60
MAXTEMP=hwmon4/pwm1=85 hwmon4/pwm3=85
MINSTART=hwmon4/pwm1=60 hwmon4/pwm3=60
MINSTOP=hwmon4/pwm1=40 hwmon4/pwm3=40
MINPWM=hwmon4/pwm1=40 hwmon4/pwm3=40
MAXPWM=hwmon4/pwm1=150 hwmon4/pwm3=150
```

### 5. Create the full-speed profile

```bash
sudo nano /etc/fancontrol.d/fullspeed
```

Add the following content:

```
# Full-speed profile - maximum cooling
INTERVAL=10
DEVPATH=hwmon4=devices/platform/PNP0C14:06/wmi_bus/wmi_bus-PNP0C14:06/F1DDEE52-063C-4784-A11E-8A06684B9B01
DEVNAME=hwmon4=dell_smm
FCTEMPS=hwmon4/pwm1=hwmon4/temp1_input hwmon4/pwm3=hwmon4/temp2_input
FCFANS=hwmon4/pwm1=hwmon4/fan1_input hwmon4/pwm3=hwmon4/fan3_input
MINTEMP=hwmon4/pwm1=0 hwmon4/pwm3=0
MAXTEMP=hwmon4/pwm1=30 hwmon4/pwm3=30
MINSTART=hwmon4/pwm1=255 hwmon4/pwm3=255
MINSTOP=hwmon4/pwm1=255 hwmon4/pwm3=255
MINPWM=hwmon4/pwm1=255 hwmon4/pwm3=255
MAXPWM=hwmon4/pwm1=255 hwmon4/pwm3=255
```

## Profile Switching Script

Create a script to easily switch between profiles:

```bash
sudo nano /usr/local/bin/fan-profile
```

Add the following content:

```bash
#!/bin/bash

# Function to detect current profile
get_current_profile() {
    for profile in balanced performance quiet fullspeed; do
        if diff -q "/etc/fancontrol" "/etc/fancontrol.d/$profile" >/dev/null 2>&1; then
            echo "$profile"
            return 0
        fi
    done
    echo "unknown"
}

# If no argument is provided, show current profile
if [ $# -eq 0 ]; then
    current=$(get_current_profile)
    echo "Current fan profile: $current"
    echo "Available profiles: balanced, performance, quiet, fullspeed"
    echo "Use: fan-profile [profile-name] to change profiles"
    exit 0
fi

if [ ! -f /etc/fancontrol.d/$1 ]; then
    echo "Profile not found: $1"
    echo "Available profiles: balanced, performance, quiet, fullspeed"
    exit 1
fi

# Stop fancontrol service
systemctl stop fancontrol

# Apply the selected profile
cp /etc/fancontrol.d/$1 /etc/fancontrol

# Start fancontrol service
systemctl start fancontrol

echo "Fan profile switched to $1"
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/fan-profile
```

## Interactive Profile Selector

Create a simple terminal-based selector:

```bash
sudo nano /usr/local/bin/fan-profile-selector
```

Add the following content:

```bash
#!/bin/bash

# Function to detect current profile
get_current_profile() {
    for profile in balanced performance quiet fullspeed; do
        if diff -q "/etc/fancontrol" "/etc/fancontrol.d/$profile" >/dev/null 2>&1; then
            echo "$profile"
            return 0
        fi
    done
    echo "unknown"
}

current=$(get_current_profile)
echo "==== Alienware Fan Profile Selector ===="
echo "Current profile: $current"
echo ""
echo "Available profiles:"
echo "1) balanced    - Normal everyday use with dynamic fan speed"
echo "2) performance - For gaming and heavy tasks - fans always active"
echo "3) quiet       - For quiet operation - minimal fan usage"
echo "4) fullspeed   - Maximum cooling - fans at full speed"
echo "q) Quit without changing"
echo ""
echo -n "Select profile (1-4 or q): "
read -n 1 choice
echo ""

case $choice in
  1) profile="balanced" ;;
  2) profile="performance" ;;
  3) profile="quiet" ;;
  4) profile="fullspeed" ;;
  q|Q) echo "No changes made."; exit 0 ;;
  *) echo "Invalid selection."; exit 1 ;;
esac

echo "Changing to $profile profile..."
sudo /usr/local/bin/fan-profile $profile
echo "Done!"
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/fan-profile-selector
```

## Desktop Launcher

Create a desktop entry to launch the selector from your applications menu:

```bash
sudo nano /usr/share/applications/fan-profiles.desktop
```

Add the following content:

```
[Desktop Entry]
Type=Application
Name=Fan Profiles
Comment=Switch between fan cooling profiles
Exec=gnome-terminal -- bash -c "/usr/local/bin/fan-profile-selector; echo 'Press Enter to close'; read"
Icon=preferences-system
Terminal=false
Categories=System;Settings;
```

## Running Without Password

To avoid password prompts when changing fan profiles:

```bash
sudo visudo -f /etc/sudoers.d/fancontrol
```

Add this line (replace USERNAME with your actual username):

```
USERNAME ALL=(ALL) NOPASSWD: /usr/local/bin/fan-profile
```

## Starting Fancontrol at Boot

Enable the fancontrol service to start automatically at boot:

```bash
sudo systemctl enable fancontrol
sudo systemctl start fancontrol
```

## Usage

- To check the current profile: `fan-profile`
- To switch profiles from terminal: `fan-profile balanced` (or performance/quiet/fullspeed)
- To use the interactive selector: `fan-profile-selector`
- To launch from applications menu: Search for "Fan Profiles"

## Notes

- The DEVPATH value may differ on your system. Check your actual fancontrol configuration.
- Fan IDs (pwm1, pwm3) may differ depending on your specific hardware.
- Temperature thresholds can be adjusted based on your laptop's thermal characteristics.

## Troubleshooting

If you encounter issues:

1. Check if the dell_smm_hwmon module is loaded:
   ```bash
   lsmod | grep dell
   ```

2. If not loaded, try loading it:
   ```bash
   sudo modprobe dell-smm-hwmon
   ```

3. If you get configuration errors, check the fancontrol service status:
   ```bash
   systemctl status fancontrol.service
   ```

4. For detailed logs:
   ```bash
   journalctl -xeu fancontrol.service
   ```

5. Check that MINSTOP is always greater than or equal to MINPWM in all profiles
