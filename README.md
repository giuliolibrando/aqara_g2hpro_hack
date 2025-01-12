# aqara_g2hpro_hack
Access the Aqara G2H Pro camera (CH-CO1) with hacked firmware

[Niceboygithub](https://github.com/niceboygithub) created non-official firmware 3.3.4 for the Aqara cameras. This version is older than the current v 4.x but allows older hack to work. [Wh1terat](https://github.com/Wh1terat) figured out a genius QRcode hack.

Full credit to these great people- [buy niceboygithub a coffee](https://www.buymeacoffee.com/niceboygithub). 

Original [instructions here](https://github.com/niceboygithub/AqaraGateway/issues/179#issuecomment-1555901949).

1. format an SD card as FAT32
2. copy firmware 3.3.4_0014.0004 onto SD card `rootfs.sqfs` (This file is from inside the 7zipped firmware and renamed from `rootfs.bin`)
3. unplug camera then insert SD card
4. hold G2H's top button down while inserting the power cable
5. after 3s release the top button and wait so the firmware can be flashed to 3.3.4

6. setup the camera in Homekit or Aqara app to confirm firmware version is 3.3.4 (then insert paperclip 5 times into reset button to clear camera)

7. [wh1terat](https://github.com/Wh1terat/aQRootG3) has created a hack which enables telnet. Run the cmd as `python3 aQRootG3.py dlink_ssid_2g thisisthepassword` (if dependencies like qrcode are missing, `pip3 install qrcode`). The QRcode must be displayed on a larger display (minimum 11x11cm)
8. Open Aqara app- if camera isn't ready for setup, use reset button with 1) hold for 5 seconds or 2) 5 quick presses of reset. Carefully position the created QRcode ~15cm from the phone's camera until you hear the audio announcing it is scanned. This will eventually error out and that is completely normal.
9. Camera reset (hold for 5 seconds, NOT 5x reset presses) and restart the Aqara process, this time setting up with the actual QR code created by the Aqara app. Add to Homekit if required.
10. Find the camera IP from your router or similar
11. Open terminal, `telnet 192.168.1.101`. Use `root` for password. It should return
```
❯ telnet 192.168.1.101
Trying 192.168.1.101...
Connected to 192.168.1.101.
Escape character is '^]'.
Camera-Hub-G2HPro-5114 login: root
login: can't chdir to home directory '/home/root'
ulimit: invalid option -- q
1 ulimit=256
/ #
```
Now, let's enable rtsp on boot.
Edit post_init.sh
```
vi /data/scripts/post_init.sh
```

From this:
```
#!/bin/sh

fw_manager.sh -r
passwd -d $USER
fw_manager.sh -t -k
```

to this:

```
#!/bin/sh

fw_manager.sh -r
passwd -d $USER
fw_manager.sh -t -k

pkill rtsp
rtsp &
```

Exit from Vi and save with "esc" and then  ":wq!"

Reboot the camera with:
```
reboot
```

Login into telnet again and check if rtsp is running with
```
agetprop sys.camera_rtsp_url
```
I should return something like this:
```
/ # agetprop sys.camera_rtsp_url

{"360p":"rtsp:\/\/192.168.1.16:8554\/360p","720p":"rtsp:\/\/192.168.1.16:8554\/720p","1080p":"rtsp:\/\/192.168.1.16:8554\/1080p"}
```
ex. Use "rtsp://192.168.1.16:8554/720p" inside Frigate if you want the 720p rtsp stream
