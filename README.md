# Lenovo Battery Conservation for Linux

## Prerequisites
- Lenovo laptop with `ideapad_acpi` driver support
- TLP (optional, for checking support)

## QuickStart
- Check if your device supports conservation mode
- Create the toggle scripts
- Set up systemd service for auto-enable on boot
- (Optional) Add shell aliases for quick toggling

### Firstly, check if your device supports __battery conservation mode__
You can use _**tlp-stat**:'which helps in viewing your power saving info'_
```bash
# u can use tlp-stat command with flag -b for battery data
    sudo tlp-stat -b
## Expected Output
## if u see battery conservation mode on Output 
## Congrats, your device supports battery_conservation feature

--- TLP 1.6.1 --------------------------------------------

+++ Battery Care
Plugin: lenovo
Supported features: charge threshold
Driver usage:
* vendor (ideapad_laptop) = active (charge threshold)
Parameter value range:
* STOP_CHARGE_THRESH_BAT0: 0(off), 1(on) -- battery conservation mode  <---<---<---<--- Here here Here

/sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode = 1 (60%)

+++ Battery Status: BAT0
/sys/class/power_supply/BAT0/manufacturer                   = Sunwoda
/sys/class/power_supply/BAT0/model_name                     = L20D4PC0
/sys/class/power_supply/BAT0/cycle_count                    =    531
/sys/class/power_supply/BAT0/energy_full_design             =  60000 [mWh]
/sys/class/power_supply/BAT0/energy_full                    =  50740 [mWh]
/sys/class/power_supply/BAT0/energy_now                     =  28930 [mWh]
/sys/class/power_supply/BAT0/power_now                      =      0 [mW]
/sys/class/power_supply/BAT0/status                         = Not charging

Charge                                                      =   57.0 [%]
Capacity                                                    =   84.6 [%]
```
----------

### for enable/disable_conservation.sh

Create these files inside `/usr/local/bin`, you can use editor of your choice[like vim, nano, code ...]

```bash
sudo nano /usr/local/bin/enable_conservation.sh

# PASTE this::
#### enable_conservation.sh
#!/bin/bash
echo 1 | sudo tee /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode
echo "Conservation mode enabled (charging capped)."
```

```bash
sudo nano /usr/local/bin/disable_conservation.sh

# PASTE this::
#### disable_conservation.sh
#!/bin/bash
echo 0 | sudo tee /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode
echo "Conservation mode disabled (full charging allowed)."
```

### quick check to see if battery_cap is on/off
```bash
sudo nano /usr/local/bin/battery_percentage.sh

# PASTE this::
#### check battery_percentage.sh
#!/bin/bash
cat /sys/class/power_supply/BAT0/capacity
```

### THEN make each script executable
```bash
# THEN make them Executable
# ---------------------------

sudo chmod +x /usr/local/bin/enable_conservation.sh
sudo chmod +x /usr/local/bin/disable_conservation.sh
sudo chmod +x /usr/local/bin/battery_percentage.sh
```

### for check_battery_conservation.sh

```bash
sudo nano /usr/local/bin/check_battery_conservation.sh
```

```bash
#!/bin/bash

# Path to conservation mode
CONSERVATION_MODE_FILE="/sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode"

# Check if conservation mode file exists
if [[ ! -f $CONSERVATION_MODE_FILE ]]; then
echo "Conservation mode file not found. Ensure this is a Lenovo laptop with the ideapad_acpi driver."
exit 1
fi

# Read the conservation mode status (0 or 1)
CONSERVATION_MODE=$(cat $CONSERVATION_MODE_FILE)

# Check if battery-conservation service is active
SERVICE_STATUS=$(systemctl is-active battery-conservation.service)

# Output the status
echo "Battery Conservation Mode: "
if [[ $CONSERVATION_MODE -eq 1 ]]; then
echo "  - Enabled (Charging capped)"
else
echo "  - Disabled (No charge cap)"
fi

echo "Battery Conservation Service Status: "
if [[ $SERVICE_STATUS == "active" ]]; then
echo "  - Running"
else
echo "  - Stopped"
fi
```

```bash
sudo chmod +x /usr/local/bin/check_battery_conservation.sh
```

### You can automate whole process by creating alias for them services
Add these to your `~/.bashrc`
```bash
## for conservation mode
alias conserve-on='sudo /usr/local/bin/enable_conservation.sh'
alias conserve-off='sudo /usr/local/bin/disable_conservation.sh'
alias conserve-status='/usr/local/bin/check_battery_conservation.sh'
alias battery-status='cat /sys/class/power_supply/BAT0/capacity'
```
Then run `source ~/.bashrc` to reload changes

### To force it into battery_cap mode on every restart/power_on
Create a file called 'battery-conservation.service' inside '/etc/systemd/system/'
```bash
sudo nano /etc/systemd/system/battery-conservation.service
```

```bash
[Unit]
Description=Enable Lenovo Battery Conservation Mode
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo 1 > /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode'

[Install]
WantedBy=multi-user.target
```

Now enable it so it runs on boot:
```bash
sudo systemctl daemon-reload
sudo systemctl enable battery-conservation.service
sudo systemctl start battery-conservation.service
```

### Tested On
- Lenovo Legion 5 15ACH6
- Ubuntu 24.04.3 LTS