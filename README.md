# 📡 Arch Linux Mobile Broadband Setup: ThinkPad X13 (Quectel) & Google Fi (h2g2)

This guide documents the configuration steps for establishing a persistent mobile broadband connection on Arch Linux using a ThinkPad X13 with a Quectel MBIM modem, specifically targeting the **Google Fi** service with its required Access Point Name (APN).

This process uses **ModemManager** and **NetworkManager** via the command-line utility `nmcli`, focusing on direct configuration vectors.

## Prerequisites

I recommend setting up NetworkManager with this Guide First:
https://github.com/Zeus-Deus/omarchy-setup/tree/main/NetworkManager

```bash
sudo pacman -S networkmanager modemmanager libmbim
Verify and enable the necessary services:

Bash

sudo systemctl enable --now NetworkManager.service
sudo systemctl enable --now ModemManager.service
1. Diagnostics and Modem Status Check
The most common failure point for Quectel modems is the FCC lock state, which must be addressed before configuration.

A. Verify ModemManager Detection
Check if the modem is detected and its index (usually 0).

Bash

mmcli -L
# Expected output: /org/freedesktop/ModemManager/Modems/0

# Check modem state
mmcli -m 0
If the state is disabled or similar, enable it:

Bash

mmcli -m 0 --enable
B. Check and Resolve FCC Lock State
If your modem is known to enter a radio-disabled state (FCC lock), you must manually unlock it using the mbimcli tool.

Identify the Modem Device Node: Look for a device node like /dev/cdc-wdm0. You can typically find this in the output of mmcli -m 0.

Query Radio State (Checking the Vector): Replace /dev/cdc-wdm0 with your actual device node.

Bash

sudo mbimcli -p -d /dev/cdc-wdm0 --quectel-query-radio-state
Unlock the Modem (If Necessary): If the output reports RadioState = 'fcc-locked', execute the unlock command:

Bash

sudo mbimcli -p -d /dev/cdc-wdm0 --quectel-set-radio-state=on
2. NetworkManager Profile Creation
We use nmcli to inject the precise APN vector required for Google Fi. The APN for Google Fi is h2g2.

A. Create the Mobile Broadband Connection Profile
This command creates the connection profile, setting the connection type to gsm (mobile broadband) and telling NetworkManager to automatically select the available WWAN interface (ifname "*") via ModemManager.

Bash

nmcli connection add type gsm con-name "Google Fi H2G2" ifname "*"
B. Inject the Google Fi APN Vector
This is the critical step that configures the connection to use the Google Fi routing key.

Bash

nmcli connection modify "Google Fi H2G2" gsm.apn "h2g2"
C. Set Autoconnect (Optional)
Configure the connection profile to automatically establish the connection whenever the modem device is available.

Bash

nmcli connection modify "Google Fi H2G2" connection.autoconnect yes
D. Activate the Connection
Bring the new profile up to initiate the connection sequence.

Bash

nmcli connection up "Google Fi H2G2"
If successful, you should see output confirming the connection is activated and a gateway has been assigned. You can verify network connectivity with a simple ping test.

3. Persistence Verification
The configuration is saved permanently in the NetworkManager profiles directory:

Bash

cat /etc/NetworkManager/system-connections/Google\ Fi\ H2G2.nmconnection
This ensures that the APN setting persists across reboots. If the connection fails after a reboot, the most likely culprit is the FCC lock state (Section 1.B). If that occurs, refer to the Arch Wiki on Mobile broadband modem#FCC locking for persistent solutions like a systemd service or udev rule.
