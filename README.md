**Most of this guide also works on other Debian / Ubuntu systems:**
- Most feeds will work on any architecture 
- FA / FR24 feed will only run on a Raspberry Pi (there are workarounds, but this guide does not cover that)
- If you're not using Raspberry Pi OS but have installed Debian / Ubuntu, start with point 2 to install the decoder
- If you don't have the required hardware yet have a look at this shopping list: https://github.com/adsbfi/adsb-wiki/wiki/adsb-receiver-shopping-list
- Join our Discord community if you have any questions: https://discord.gg/jfVRF2XRwF


## 1. Write Raspberry Pi OS Lite / Raspbian Lite image to the SD card

This guide also works with other Raspberry Pi OS images, but there is really no need for a full desktop interface.

Use the Raspberry Pi Imager to install Raspberry Pi OS Lite to the SD card:
- https://www.raspberrypi.com/documentation/computers/getting-started.html#using-raspberry-pi-imager
- Place the SD card in your SD card reader (use MicroSD to SD Card adapter if needed)
- Run the Imager software
- Select Raspberry Pi OS Lite under "Raspberry Pi OS (Other)"
- Click the gear icon
- Enable SSH access to manage your Pi over network
- Add an user account and password
- Configure Wi-Fi if you want to use that
- Click save and choose your SD card as the storage
- Start the write process

Once the SD card is ready, get the Pi up and running:
- Place the SD card in the Pi, power up the system and wait few minutes for it to start up
- Find the IP address of the Pi (https://www.raspberrypi.com/documentation/computers/remote-access.html#introduction-to-remote-access)
- Connect using an SSH client and log in with the credentials configured above, see the following links depending on your operating system:
- https://www.raspberrypi.com/documentation/computers/remote-access.html#secure-shell-from-windows-10
- https://www.raspberrypi.com/documentation/computers/remote-access.html#secure-shell-from-linux-or-mac-os

All logs you need should be available via journalctl and are stored in memory / lost on reboot. 
To reduce disk writes to the SD card, it is advisable to remove rsyslog as it is not really needed:

 `sudo apt remove rsyslog -y`

#### Using a different system like a laptop with Ubuntu or pi24 image

- The imaging part, booting the system and connecting via SSH depends on the system
- Feeding sites besides adsb.fi using a laptop is more complicated and is not described here
- graphs1090 and most other feeder utilities should work on all Debian based Linux systems (for example Ubuntu)
- The scripts for installing readsb and feeding adsb.fi should work on CentOS as well


## 2. Decoder to talk to the SDR (readsb and dump1090 are commonly used decoders)

- Script installation: https://github.com/adsbfi/adsb-scripts/wiki/Automatic-installation-for-readsb
- Building from source: https://github.com/adsbfi/adsb-wiki/wiki/Building-readsb-from-source

Try a reboot if it's not working. There can be permission issues with the SDR when librtlsdr is installed for the first time.

### 2.1 Set location

- To ensure the local map and graphs have all functions, it is important to set the location as described in the decoder you are using
- After the script installation of readsb you can use:
```
sudo readsb-set-location 50.12344 10.23429
```

### 2.2 RTL-SDR v3 Bias Tee

- If you need to enable the bias tee on RTL-SDR v3
- https://github.com/adsbfi/adsb-wiki/wiki/RTL%E2%80%90SDR-Bias-Tee


### 2.3 Airspy users only

- Run airspy-conf after installing the decoder of your choice
- https://github.com/adsbfi/airspy-conf#airspy-conf


## 3. Tar1090, webinterface with many more features than the dump1090-fa/SkyView interface:

- Requires decoder like readsb or dump1090-fa
- Automatically installed with the readsb install script linked above

https://github.com/adsbfi/tar1090#tar1090


## 4. Feed adsb.fi with your feeder (recommended)

- https://adsb.fi
- https://github.com/adsbfi/adsb-fi-scripts

You should start seeing MLAT results on your local tar1090 if there are aircraft without ADS-B in your area and the coverage is sufficient.


## 5. Automatic gain optimization (optional, recommended)

If you don't want to optimize gain manually, install this and it will slowly get to the right setting

https://github.com/adsbfi/adsb-scripts/wiki/Automatic-gain-optimization-for-readsb-and-dump1090-fa


## 6. 978 / UAT, used by some general aviation (optional, US only)
For a dual 1090 / 978 system

https://github.com/adsbfi/adsb-wiki/wiki/Dual-1090-978-setup


## 7. Graphs1090 for better statistics (optional)

https://github.com/adsbfi/graphs1090#graphs1090


<details>
 <summary>
<h2>8. Other feed clients (optional)</h2>
</summary>

Specify a network Beast receiver IP 127.0.0.1 port 30005 (beast protocol).
Do not select DVB-T / USB / SDR input. It will interfere with the readsb decoder you already installed.

In case something stops working, just rerun the install script for readsb. It fixes common configuration errors with other feed clients.

### 8.1 Feed Flightaware (optional)

The steps below may look redundant, they are NOT. (there are issues with removing config files and them being not installed)
Execute all of them!
```
URL=""
dpkg --print-architecture | grep -qs -e armhf && URL="https://flightaware.com/adsb/piaware/files/packages/pool/piaware/f/flightaware-apt-repository/flightaware-apt-repository_1.1_all.deb"
wget -O /tmp/piaware-repo.deb "$URL"
sudo apt purge -y piaware-repository &>/dev/null
sudo rm -f /etc/apt/sources.list.d/piaware-*.list
sudo dpkg -i /tmp/piaware-repo.deb
sudo apt update
sudo apt install -y piaware
```


Once you are receiving aircraft on your local map for a couple of minutes, you can link it to your account if you're into that: https://flightaware.com/adsb/piaware/claim

See also: https://flightaware.com/adsb/piaware/install

If you're keen on keeping your stats, you'll need to use the feeder-id from the stats page you wish to retain and configure piaware to use that ID: https://discussions.flightaware.com/t/for-beginners-how-to-get-back-existing-station-number-in-a-fresh-install/30981/

Note that only armhf is supported by Flightaware. (Unsupported: aarch64, x86_64 (amd64) etc.)

### 8.2 Feed FR24 (optional)

- It is recommended to install and update FR24 without using their Debian package
- fr24feed package autoupdate has broken receivers on multiple occasions in the past
- This install script only utilizes the binary from them and removes the auto updater and unnecessary scripts, which are not required when running standalone / readsb decoder.
- FR24 MLAT does not work on Raspberry Pi, they recommend disabling it
```
sudo bash -c "$(wget -O - https://github.com/adsbfi/adsb-scripts/raw/master/fr24-nopackage.sh)"
```

Note that only armhf is supported by FR24. (Unsupported: aarch64, x86_64 (amd64) etc.)

- Configuration can be done using this command (not necessary when updating).
```
sudo fr24feed --signup; sudo systemctl restart fr24feed
```

- Choose the following options (for others it is up to you to decide)
```
Would you like to use autoconfig $:
no

Step 1.3 - Would you like to participate in MLAT calculations? (yes/no)$:
no

Step 4.1 - Receiver selection:
Enter your receiver type (1-7)$:
4 ModeS Beast (USB/Network)

Step 4.2 - Please select connection type:
Enter your connection type (1-2)$:
1 (Network)

Step 4.3A - Please enter your receiver's IP address/hostname (127.0.0.1 is correct for everyone, means same computer)
127.0.0.1

Step 4.3B - Please enter your receiver's data port number
30005

Step 5.1 - Would you like to enable RAW data feed on port 30334 (yes/no)$:
no

Step 5.2 - Would you like to enable Basestation data feed on port 30003 (yes/no)$:
no

Step 6 - Please select desired logfile mode:
0

```
- If your tar1090 / dump1090-fa map is no longer working, it's likely due to a configuration error. The easiest way to fix this is by running the automatic installation script mentioned earlier on this page to install readsb. These scripts will help fix the configuration.

- Check status / logs:
```
sudo fr24feed-status
sudo journalctl -u fr24feed --no-pager
```

- Disabling / removing fr24feed:
```
sudo systemctl disable --now fr24feed
```

</details>

<details>
 <summary>
<h2>9. Miscellaneous fixes and workarounds</h2>
 </summary>
<details>

### 9.1 Potential workaround for losing network connectivity due to dhcpcd / Wi-Fi:
- [[Raspbian dropping off the network potential workaround]]

### 9.2 Static IP address
- [[Static IP]]

### 9.3 Fix for Pihole websites not working:
```
echo 'server.modules += ("mod_alias")' | sudo tee /etc/lighttpd/external.conf
sudo systemctl restart lighttpd
```

### 9.4 Disabling IPv6 (for IPv6 issues)
```
sudo tee /etc/sysctl.d/08-disable-ipv6.conf <<EOF
# disable IPv6
#
net.ipv6.conf.all.disable_ipv6 = 1
EOF
sudo reboot
```
</details>
