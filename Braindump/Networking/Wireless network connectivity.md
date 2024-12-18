---
tags:
  - networking
  - wifi
---

List wireless interfaces

```bash
iw dev
```

List all known access points

```bash
# Note the 'beacon loss: 2' this indicates we lost connectivity with this AP 2 times already
# Note the 'connected time' which indicates how long the connection with the AP is established
iw dev wlp3s0 station dump

Station 31:6c:c0:xx:yy:z0 (on wlp3s0)
 inactive time: 200 ms
 rx bytes: 140367
 rx packets: 568
 tx bytes: 6789530
 tx packets: 6728
 tx retries: 50
 tx failed: 0
 beacon loss: 2
 beacon rx: 370
 rx drop misc: 1
 signal:   -48 [-48, -53] dBm
 signal avg: -46 [-46, -50] dBm
 beacon signal avg: -46 dBm
 tx bitrate: 400.0 MBit/s VHT-MCS 9 40MHz short GI VHT-NSS 2
 tx duration: 0 us
 rx bitrate: 300.0 MBit/s VHT-MCS 7 40MHz short GI VHT-NSS 2
 rx duration: 0 us
 authorized: yes
 authenticated: yes
 associated: yes
 preamble: long
 WMM/WME: yes
 MFP:  no
 TDLS peer: no
 DTIM period: 1
 beacon interval:100
 short preamble: yes
 short slot time:yes
 connected time: 32 seconds
 associated at [boottime]: 4923.921s
 associated at: 1624608068803 ms
 current time: 1624608100434 ms
```

List all available Frequencies[channel]

```bash
iw list

# Notice there can be multiple Bands indiciating 2Ghz and 5Ghz
...
# The [xx] indicates the channel number
  Frequencies:
   * 5180 MHz [36] (22.0 dBm) (no IR)
   * 5200 MHz [40] (22.0 dBm) (no IR)
   * 5220 MHz [44] (22.0 dBm) (no IR)
   * 5240 MHz [48] (22.0 dBm) (no IR)
   * 5260 MHz [52] (22.0 dBm) (no IR, radar detection)
   * 5280 MHz [56] (22.0 dBm) (no IR, radar detection)
   * 5300 MHz [60] (22.0 dBm) (no IR, radar detection)
   * 5320 MHz [64] (22.0 dBm) (no IR, radar detection)
   * 5340 MHz [68] (disabled)
   * 5360 MHz [72] (disabled)
   * 5380 MHz [76] (disabled)
   * 5400 MHz [80] (disabled)
   * 5420 MHz [84] (disabled)
   * 5440 MHz [88] (disabled)
   * 5460 MHz [92] (disabled)
   * 5480 MHz [96] (disabled)
   * 5500 MHz [100] (22.0 dBm) (no IR, radar detection)
   * 5520 MHz [104] (22.0 dBm) (no IR, radar detection)
   * 5540 MHz [108] (22.0 dBm) (no IR, radar detection)
   * 5560 MHz [112] (22.0 dBm) (no IR, radar detection)
   * 5580 MHz [116] (22.0 dBm) (no IR, radar detection)
   * 5600 MHz [120] (22.0 dBm) (no IR, radar detection)
   * 5620 MHz [124] (22.0 dBm) (no IR, radar detection)
   * 5640 MHz [128] (22.0 dBm) (no IR, radar detection)
   * 5660 MHz [132] (22.0 dBm) (no IR, radar detection)
   * 5680 MHz [136] (22.0 dBm) (no IR, radar detection)
   * 5700 MHz [140] (22.0 dBm) (no IR, radar detection)
   * 5720 MHz [144] (22.0 dBm) (no IR, radar detection)
   * 5745 MHz [149] (22.0 dBm) (no IR)
   * 5765 MHz [153] (22.0 dBm) (no IR)
   * 5785 MHz [157] (22.0 dBm) (no IR)
   * 5805 MHz [161] (22.0 dBm) (no IR)
   * 5825 MHz [165] (22.0 dBm) (no IR)
   * 5845 MHz [169] (disabled)
   * 5865 MHz [173] (disabled)
   * 5885 MHz [177] (disabled)
   * 5905 MHz [181] (disabled)
...
```

## Specify a single AP to be used

I recently had an issue where I noticed that by `NetworkManager` would lose connection. After some investigation it seemed because it was regularly switching between two access points that had similar signal strength.

```bash
Dec 05 10:38:41 viper wpa_supplicant[1883]: wlp0s20f3: SME: Trying to authenticate with 34:2c:c4:e8:ab:cd (SSID='My-Home-ID' freq=5200 MHz)
Dec 05 10:38:41 viper kernel: wlp0s20f3: disconnect from AP 34:2c:c4:e8:ba:dc for new auth to 34:2c:c4:e8:ab:cd
Dec 05 10:38:41 viper kernel: wlp0s20f3: authenticate with 34:2c:c4:e8:ab:cd (local address=92:fa:ba:a7:aa:bb)
Dec 05 10:38:41 viper kernel: wlp0s20f3: send auth to 34:2c:c4:e8:ab:cd (try 1/3)
```

You can list your currently detected access points and see their strength.

```bash
 ❯ nmcli dev wifi list                                                              
IN-USE  BSSID              SSID                              MODE   CHAN  RATE        SIGNAL  BARS  SECURITY    
*       34:2c:c4:e8:ba:dc  My-Home-ID                   Infra  44    405 Mbit/s  66      ▂▄▆_  WPA2        
        34:2c:c4:e8:ab:cd  My-Home-ID                   Infra  40    405 Mbit/s  59      ▂▄▆_  WPA2     
```

Configure `NetworkManager` to always use a specific access point. This disables roaming.

```bash
nmcli connection modify "My-Home-ID" 802-11-wireless.bssid 34:2c:c4:e8:ba:dc
```