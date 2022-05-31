# supercollider-raspberryPi
An instruction of configuration of running supercollider autostart in Raspberry Pi 4B.

Several features it should achive: 
* Using Patchbox OS.
* Autostart with jack and supercollider IDE, sclang get booted successfully with StartupFile Quark.
* Connect to midi controller automatically (tested with Behringer x touch mini).
* Running whether with head or in headless mode, it dose not need to specify again.
* Several Quacks (StartupFile, MTMI...) get installed in supercollider.
## Install PatchBox
>Patchbox OS is a custom Linux distribution specially designed for Raspberry Pi based audio projects. It comes pre-configured for low latency audio performance and pre-installed audio software that will help you get started with your projects in no time!

download and flash it into sd card. (Release date:2022-05-17)  
[PatchBox OS](https://blokas.io/patchbox-os/)
### Initial setup wizard in PatchBox
Make sure the buffer size is bigger than 512 so that it has no distorded sound.
Choosing the right sound interface.

You can configure PatchBox setup by:  
`patchbox`  

or modify:  
`sudo nano /etc/jackdrc/`  

It should looks like this:  
`exec /usr/bin/jackd -t 2000 -R -P 95 -d alsa -d hw:Device -r 44100 -p 512 -n 2 -X seq -s -S`  

Check if the `hw:Device` , `-p 512` and `-X seq` is correct!

**Save it with ctl+x, press y and then enter.**
## Install StartupFile Quark  
## Jack Autostart with [systemd](https://wiki.archlinux.org/title/systemd)

Modify Jack.service:  
`sudo nano /lib/systemd/system/jack.service`

```[Unit]
Description=JACK Server
After=sound.target

[Service]
LimitRTPRIO=95
LimitMEMLOCK=infinity
Environment=JACK_NO_AUDIO_RESERVATION=1
Environment=JACK_PROMISCUOUS_SERVER=jack
ExecStart=/etc/jackdrc
User=jack
Group=jack

[Install]
WantedBy=multi-user.target
```

Reload the the file (make sure to do so after everytime you modify the file)  
`sudo systemctl daemon-reload`  
Enable jack to boot itself whenever it is.  
`sudo systemctl enable jack`

## Supercollider Autostart with systemd

Modify supercollider.service:  
`sudo nano /etc/systemd/system/supercollider.service`  
```
[Unit]
Description=SuperCollider
After=jack.service
After=graphical.target

[Service]
User=patch
Type=simple
ExecStart=/usr/bin/scide
Environment="DISPLAY=:0.0"
Environment="JACK_PROMISCUOUS_SERVER=jack"

[Install]
WantedBy=graphical.target
```
**Here `User = patch`, make sure that user is whatever your username is, in my case it is patch.**

Reload the the file (make sure to do so after everytime you modify the file)   
`sudo systemctl daemon-reload`  
Enable supercollider to boot itself whenever it is:  
`sudo systemctl enable supercollider`
## Connect to midi controller

Disable the preset midi connection in Patchbox:  
`sudo systemctl stop amidiauto`  
`sudo systemctl disable amidiauto` 
## Testing
Reboot the RPI after connect everything (usb audio interface, MIDI controller...)  
`sudo reboot`  
Check if the controller is connected  
`lsusb`  

one can restart the sevice to test it first on the fly.  
`sudo systemctl stop jack`  
`sudo systemctl start jack`  
or simply:  
`sudo systemctl restart jack`  
check it if jack has been booted successfully:  
`sudo systemctl status jack`

same thing as supercollider:  
`sudo systemctl stop supercollider`  
`sudo systemctl start supercollider`...  
make sure that supercollider start after jack is booted.
### Debug the whole systemd service:  
`sudo systemd-analyze critical-chain`
