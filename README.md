# Pi Kiosk

<p align="center"><img alt="Raspberry Pi Touch Display 2 with HA Kiosk" src="/resources/pi-touch-display-kiosk.jpeg" height="auto" width="600"></p>

This project contains the simplest form of a persistent Raspberry Pi-based browser kiosk.

Kiosks are useful if you want to build a web dashboard for something and have the Pi display it, or have a custom web page or online video play automatically when your Pi boots up, and take over the full screen.

This configuration is not meant to be highly secure, or extremely robust, I just use it when I want to pop a web page up on one of my Pis full-screen, like for a Home Assistant control panel.

For my own Kiosk, I'm using the following hardware to display a Home Assistant dashboard:

  - Raspberry Pi 5 (2GB)
  - Raspberry Pi Touch Display 2
  - [Touch Display 2 3D Printed Stand](https://www.printables.com/model/1062445-raspberry-pi-touch-display-2-stand)

## Setup

Ensure you're running Pi OS with a graphical interface (the 'full install'), and Chromium is installed (it should be, by default).

Then open up Terminal and install a prerequisite:

```
sudo apt install unclutter
```

Create the script that will run the Kiosk:

```
mkdir -p /home/pi/kiosk
cp kiosk.sh /home/pi/kiosk/kiosk.sh
```

Copy over the SystemD unit file to run the `kiosk` service:

```
sudo cp kiosk.service /lib/systemd/system/kiosk.service
```

Enable the Systemd `kiosk` service so it will automatically run at system boot:

```
sudo systemctl enable kiosk.service
```

(Optional) Fire up the `kiosk` service immediately:

```
sudo systemctl start kiosk
```

## Rotating the Screen

To rotate the Touch Display immediately, you can run:

```
wlr-randr --output DSI-1 --transform 90
```

This orientation assumes the Pi is oriented so the USB-C plug is facing towards the bottom of the Touch Display. If you have the display plugged into the DSI-2 port, change the `--output` accordingly.

To persist the rotation setting, either configure the screen rotation on the display using the "Screen Configuration" utility, or edit the `kanshi` config file at `~/.config/kanshi/config`.

### Rotating the screen during boot (pre-GUI)

You can also set the display to be rotated during boot (either with the Pi Desktop splash screen or console output) by editing `/boot/firmware/cmdline.txt`. Add the following before the last parameter and reboot:

```
video=DSI-1:panel_orientation=left_side_up
```
Options are "normal", "upside_down", "left_side_up", or "right_side_up".

## Power Savings

Since a Kiosk will likely run quite frequently (or all day), there are a few things you can do to reduce its power consumption:

### Reduce shutdown power consumption by 140x

Following the [guide in my blog post](https://www.jeffgeerling.com/blog/2023/reducing-raspberry-pi-5s-power-consumption-140x), edit your EEPROM file (`sudo rpi-eeprom-config -e`) and set the following settings:

```
[all]
BOOT_UART=1
WAKE_ON_GPIO=0
POWER_OFF_ON_HALT=1
```

(Don't worry about changing any other values.) Save the changes and wait for the EEPROM to be updated. Once it is, the Pi 5's power consumption while shut down will be less than 0.01W (with the Pi's default configuration, the shutdown power consumption could be up to 1-2W!).

### Dim the screen while not in use

Running an always-on display at 100% brightness all day is not that helpful.

Right now I'm looking into the best way to dim the screen after a period of time, or configure the 'Screen Blanking' option in Pi OS.

See issue: [Allow for screen to dim after a certain amount of time](https://github.com/geerlingguy/pi-kiosk/issues/2)

## Debugging

You can view the `kiosk` service logs by running the command:

```
journalctl -u kiosk
```

(Follow the logs live by appending `-f` to that command.)

## Stopping the service

If you can log in via SSH in the background, you can stop the service with:

```
sudo systemctl stop kiosk
```

Otherwise, if you have access to the Pi itself, and can plug in a keyboard, you can press `Ctrl` + `F4`, and that _should_ quit the browser.

Another alternative is to set up the Pi with [Pi Connect](https://www.raspberrypi.com/documentation/services/connect.html), and when connected to the Pi remotely, press `Ctrl` + `F4`.

> Note: On a Mac, you need to press `Fn` + `Ctrl` + `F4`.

If you want to get _really_ fancy, you can [use something like `xdotool`](https://unix.stackexchange.com/a/703023/16194) and wire up a button to emulate keypresses to quit the browser.

## License

GPL v3

## Author

This project was created by Jeff Geerling in 2024, loosely following this guide from PiMyLifeUp: [Raspberry Pi Kiosk using Chromium](https://pimylifeup.com/raspberry-pi-kiosk/).
