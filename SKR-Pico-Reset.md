# Work Around: SKR Pico not present after booting or rebooting the Rpi


## Symptom

When booting or rebooting the Raspberry Pi, the SKR Pico is not present on the USB bus. In ssh `dmesg` will report "Unable to enumerate", and `lsusb` will not show the board. Pressing the "RESET" button on the SKR Pico often fixes the issue.

## Workaround

Since no real solution exists so far, we can implement a mechanism to reset the SKR Pico without needing to physically press the RESET button.

- **Connect a dupont-style jumper cable from the SKR Pico's Reset pin on the SWD Header to the Raspberry Pi GPIO 19 (Pin 35).** Refer to the attached picture for details. This allows us to control the RESET mechanism of the SKR Pico remotely.
- **Copy and paste the following commands (right-click in Putty to paste) to create a restart script**:
```
cat << EOT > /usr/local/bin/restart-skr-pico.sh
#!/bin/bash
raspi-gpio set 26 op
raspi-gpio set 26 dl
raspi-gpio set 26 dh
EOT
chmod +x /usr/local/bin/restart-skr-pico.sh
```

- **To manually reset the SKR Pico, execute** `/usr/local/bin/restart-skr-pico.sh`

## Modified Workaround

Some SKR Picos need rebooting multiple times, this script will perform that once per second until it is present. The same dupont connector is needed.

```
#!/bin/bash
port=/dev/serial/by-id/usb-Klipper_rp2040_<<<YOUR_ID_HERE>>>-if00

while ! [ -e "$port" ]
do
   echo "SKR is not found"
   raspi-gpio set 26 op
   raspi-gpio set 26 dl
   raspi-gpio set 26 dh
   sleep 1
done
echo "SKR is found!"
```


- **To automatically reset the SKR Pico prior each Klipper start, copy and paste this command to modify the Klipper service, then restart your Pi:**
```
grep -c "ExecStartPre=/usr/local/bin/restart-skr-pico.sh" /etc/systemd/system/klipper.service || sed -i '/^\[Service\].*/a ExecStartPre=/usr/local/bin/restart-skr-pico.sh' /etc/systemd/system/klipper.service
```

# Wiring

![](img/skr-pico-reset.png)