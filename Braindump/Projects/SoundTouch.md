Our friends at Bose decided the stop supporting their  SoundTouch Cloud, basically making a lot of speakers completely useless. I didn't pay all that money just to have a stupid Bluetooth speaker.

Luckily, opensource saves the day! With this project you can restore functionality.
https://github.com/gesellix/Bose-SoundTouch

In these notes we will go through the steps to get it working on your speaker.

Reference: https://github.com/gesellix/Bose-SoundTouch/blob/main/docs/content/docs/guides/ON-DEVICE-INSTALL-WALKTHROUGH.md
## Connecting to your speaker

First step is connect to your Bose speaker.
* Format an USB as FAT
* Mount it and create an empty `remote_services` file
* Insert it to your speaker and power cycle your speaker
* Figure out the IP address of your speaker. You can press the keys '5' and '-' at the same time to get to a display menu that can show you your IP address.
* SSH to your speaker, no password is required.

```bash
# My speaker used settings that were older as a dinosaur, which prevented me from logging in. I've had to do the following stuff to be able to connect to it.
$ sudo update-crypto-policies --set LEGACY
$ sudo systemctl restart sshd
$ ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.0.XX
```

Make sure to revert this LEGACY policy after you are done with the following command.
```bash
sudo update-crypto-policies --set DEFAULT 
```

## Installing AfterTouch

Once connected via SSH to your device, do the following steps.
Download the latest version of AfterTouch.

```bash
# Assuming you are logged in via SSH to your speaker
$ rw && curl -sSL https://raw.githubusercontent.com/gesellix/Bose-SoundTouch/main/scripts/on-device-install/install.sh | sh
```

Verify that the endpoint is up.
```bash
$ wget -qO- http://localhost:8000/health

{"status":"up","timestamp":"2026-06-26T15:23:26+02:00","vcs_modified":"false","vcs_revision":"9331e63b2da54814a0cc8507d6e7c2072f199078","vcs_time":"2026-06-22T19:23:15Z","version":"v0.0.0-20260622192315-9331e63b2da5"}
```

Run sync and reboot.

```bash
$ sync
$ reboot
```

Wait for your speaker to reboot and connect again via SSH.
```bash
$ ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.0.XX
```

Open a new terminal window and connect via SSH and  port forwarding.

```bash
$ ssh -oHostKeyAlgorithms=+ssh-rsa -L 8000:localhost:8000 root@192.168.0.XX
```

Navigate to http://localhost:8000/ in your browser.

In the AfterTouch UI:
(I did not have to do this step as I did not have this warning.)
1. Open the **Health** tab.
2. Run or refresh the health checks.
3. Look for the warning:
    
    > _Speaker reports an empty `<margeAccountUUID>`_
    
1. Click the **QuickFix** button (labelled "Fix", "Pair account", or "Apply QuickFix" depending on the version) and confirm.

My margeAccount was actually empty, but I did not see any 'QuickFix'. Instead I had to click on the button shown (Complete pairing) in the following entry. It was on the Health tab.
```
### Speaker reports an empty <margeAccountUUID>
Complete pairing
```

Add your hostname to your /etc/hosts file. The migration does not let you point to localhost.

```bash
root@spotty:~# cat /etc/hosts
127.0.0.1	localhost.localdomain		localhost
127.0.0.1    spotty
```

Navigate to the '4.Migration' tab and change all the URLs so they point to 'https://spotty:8080/xxx'.

Click on 'Apply Suggested Plan' at the bottom. Your speaker will now migrate.

Then reboot again to let the pairing take effect:

```shell
$ sync
$ reboot
```

Connect to your speaker again.
```bash
$ ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.0.XX
```

If you want to program preset buttons from the command line, download the CLI binary to the speaker's `/tmp` (tmpfs, so it survives only until the next reboot — which is fine for a one-time setup run):

```shell
$ cd /tmp

$ curl -L --fail -o soundtouch-cli https://github.com/gesellix/Bose-SoundTouch/releases/download/v0.111.3/soundtouch-cli-v0.111.3-linux-armv7

$ chmod +x soundtouch-cli

$ /tmp/soundtouch-cli --version
soundtouch-cli version v0.111.3
```

Replace `v0.111.3` with the version you installed.

Each station must be playing before it can be saved. The `sleep 5` gives the speaker time to buffer and confirm the stream before storing.

> **Press preset buttons briefly.** A long press on the physical hardware overwrites the stored preset.

```shell
# Preset 1 — Q-Music
./soundtouch-cli --host 127.0.0.1 source custom-radio --url https://streams.radio.dpgmedia.cloud/redirect/qmusic_be/mp3 --name Q-Music --service-url "http://localhost:8000"
sleep 5
./soundtouch-cli --host 127.0.0.1 preset store-current --slot 1

# Preset 2 - MNM
./soundtouch-cli --host 127.0.0.1 source custom-radio --url http://icecast.vrtcdn.be/mnm-high.mp3  --name MNM --service-url "http://localhost:8000" 
sleep 5
./soundtouch-cli --host 127.0.0.1 preset store-current --slot 2
```

Okay, this was a shit show. It did not work at all and I always got the following error:
```bash
✗ Cannot select custom radio: Internet Radio service is not available
⚠️  💡 Available streaming services: Amazon Music, Deezer, Spotify
⚠️  To bypass this check, set SOUNDTOUCH_SKIP_AVAILABILITY_CHECK=true
2026/06/26 16:58:47 LOCAL_INTERNET_RADIO is not available
```

Trial and error and https://github.com/gesellix/Bose-SoundTouch/issues/521 helped me solved. I basically did a factory reset (1 + 'VOLUME DOWN' button), connect the speaker back to Wifi and attempt to do the migration again in the UI and for some reason it suddenly worked.

**The important thing was that I needed to do the migration first, then do a factory reset and then fix the empty margeAccount by clicking on the 'pair' button in the Health tab.**