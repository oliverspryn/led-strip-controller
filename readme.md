This LED Strip Controller is designed to function as a simple web application interface to control an RGB LED light strip. Follow these steps to set up and install this system for your own use.

# Prerequisites
In order to use this program, you will need the following in advance:

### Hardware
- Raspberry Pi 1, 2, or 3 Model B
- RGB LED strip
- Breadboard
- Three MOSFETs
- Jumper wires, male-to-male and male-to-female
- 12V DC, 2Amp+ power supply
- Power jack

### Software
- Raspbian latest
- NodeJS & NPM latest
- Git client
- Ngrok with any paid plan (Optional, for control over the internet)

# Set up the Hardware
Follow the steps in this tutorial to set up hardware: http://popoklopsi.github.io/RaspberryPi-LedStrip You can stop at step 5, since the rest of these directions will focus on getting the lights to work over a web interface.

# Software Installation and Set Up
After properly configuring the hardware, the software may be installed.

### Install NodeJS & NPM
Raspbian includes version `0.10.xx` of NodeJS, which is not recent enough to run this program. Obtain and install the latest using these steps:

1. Go to: https://nodejs.org/dist/latest/
2. Find the correct version of Node for your hardware
    * Pi 1: ARM V6
    * Pi 2: ARM V7
    * Pi 3: ARM 64
3. Log into your Raspberry Pi terminal
4. Download NodeJS (e.g. for 6.7.0):
    * Pi 1: `wget https://nodejs.org/dist/latest/node-v6.7.0-linux-armv6l.tar.gz -O node.tar.gz`
    * Pi 2: `wget https://nodejs.org/dist/latest/node-v6.7.0-linux-armv7l.tar.gz -O node.tar.gz`
    * Pi 3: `wget https://nodejs.org/dist/latest/node-v6.7.0-linux-arm64.tar.gz -O node.tar.gz`
5. Extract the archive: `tar -xvf node.tar.gz`
6. Go into the extracted directory: `cd <extracted node directory>`
7. Install NodeJS: `sudo cp -R * /usr/local/`
8. Reboot: `sudo reboot`
9. Log back into your Raspberry Pi terminal
10. Check the installed version of NodeJS: `node -v`
    * This should match the version number you downloaded in step 4
11. Check the installed version of NPM, the NodeJS package manager: `npm -v`
    * This was installed as part of NodeJS
    * Its version number will not match the NodeJS version number, but should be greater than `3.0.0`

### Install the LED Strip Controller
Now that the prerequisite software has been installed, the LED Strip Controller can be installed.

1. Log into your Raspberry Pi terminal
2. Navigate to or create a directory where you want your to be software installed
3. Download the software: `git clone https://github.com/oliver-spryn/led-strip-controller.git`
4. Go into the installation directory: `cd led-strip-controller`
5. Install the project dependencies: `npm install`
6. Test out your application: `sudo node index.js`
    * `sudo` is necessary, since this programs controls the Pi's GPIO pins

### Start Up the Controller on Boot (Optional)
If you would like the controller to start up each time the Raspberry Pi is booted, follow these steps:

1. Log into your Raspberry Pi terminal
2. Edit the `rc.local` file: `sudo nano /etc/rc.local`
3. Add the following lines, **before** the `exit 0` statement (usually the last line):
    * `sudo` is necessary, since this programs controls the Pi's GPIO pins
    * Replace `/full/path/to/index.js` with the full path to your `index.js` file
    ```bash
    # Start up the LED Strip Controller web interface
    sudo /usr/local/bin/node /full/path/to/index.js
    ```
4. Save your file (`Ctrl + O`) and exit (`Ctrl + X`)

# Accessible Interface from the Web (Optional)
Under most circumstances, the above directions will make the web interface accessible only on a LAN. If you would like to access the interface from the World Wide Web, [Ngrok](https://ngrok.io) can help. Ngrok is a service which can be used to tunnel through a LAN firewall, and expose this website to the outside world with a custom domain name.

### Obtain an Ngrok Account
Open up an account at [ngrok.com](https://ngrok.com) and select any paid plan in order to get a public domain available on the web to access your Raspberry Pi.

### Download and Install Ngrok
After obtaining an Ngrok account, the application will need installed on the Raspberry Pi:

1. In a web browser, go to the [Ngrok download page](https://ngrok.com/download)
2. Copy the link to the Linux ARM download
3. Log into your Raspberry Pi terminal
4. Download the client to the Raspberry Pi: `wget <link you copied> -O ngrok.zip`
5. Unzip your download: `unzip ngrok.zip`
6. Go into the extracted directory
7. In a web browser, log into Ngrok, and go to [dashboard.ngrok.com/auth](https://dashboard.ngrok.com/auth) to obtain your account authentication token
8. Install the token to the Raspberry Pi with: `./ngrok authtoken <your token>`

### Install Ngrok as a Service
When running Ngrok from the Raspberry Pi, it usually must run in the foreground in order to accept requests from the outside world. However, running this a service will allow it to run in the background, forward requests to the LED Strip Controller, and start up automatically.

1. In a web browser, log into Ngrok, and go to [dashboard.ngrok.com/reserved](https://dashboard.ngrok.com/reserved) to reserve a custom subdomain for you to access your Raspberry Pi's web interface
2. Log into your Raspberry Pi terminal
3. Find the `ngrok.service` file, as part of the LED Strip Controller download, and open it in a text editor: `nano ngrok.service`
4. Change the line starting with `ExecStart` and adjust it for your configuration:
    * Change `/path/to/ngrok` to the full path to the Ngrok client you downloaded earlier
    * Change `your-subdomain` to the subdomain you registered in step 1, excluding `ngrok.io`. For example, if you registered `my-custom-domain.ngrok.io`, then enter just `my-custom-domain`.
5. Save your file (`Ctrl + O`) and exit (`Ctrl + X`)
6. Back in the terminal, copy the service to Raspbian's service directory: `sudo cp ngrok.service /lib/systemd/system`
7. Install the service: `sudo systemctl enable /lib/systemd/system/ngrok.service`
8. Start the service: `sudo systemctl start ngrok.service`


### Final Testing
Restart your Raspberry Pi, and wait for it to boot up. Once it is booted, visit the URL you created in your Ngrok Dashboard, and you should see the status of the LED Strip Controller.

# Using the LED Strip Controller
Once the web interface is running, here is how you use the web interface to check the status of your lights, turn them on and off, and change their color. Please note that each time you see `url.com` in the documentation below, this will either be the IP address of your Raspberry Pi, if you are on a LAN, or the Ngrok subdomain. So for example, instead of entering `url.com` to access your Raspberry Pi, use:
* **Local Internet (LAN):** http://<ip address>:3000/
    * Notice the `:3000` in the link. That is important.
* **WWW Access (Ngrok):** http://<ngrok subdomain>.ngrok.io/
    * The `:3000` should **not** be added when using Ngrok

### Understanding JSON and Color Values
When reporing on the status of the lights, the LED Strip Controller will give you JSON, looking something like this:

```json
{"red":100,"green":200,"blue":255,"status":"on"}
```

It is important to know how to read this message to understand the status of your lights. It contains information about the red, blue, and green lights, as well as whether the strip is on or off. The colors will have values between `0` and `255`, inclusive. `0` is off and `255` is full brightness for that color. The status will be either on or off.

### Control the Lights
Here are the pages you can visit to change the status of the lights.

|URL|Description|
|---|-----------|
|`url.com/`|Check the status of the lights, but do not change anything|
|`url.com/off`|Turn off the lights|
|`url.com/on`|Turn on the lights|
|`url.com/toggle`|Toggle the status of the lights|

All of the above links, except for the first one, support changing the color of the lights. After each of the links, add:

* `url.com/on?red=255`: Turn red on full brightness. The settings for green and blue will not be changed.
* `url.com/on?red=255&blue=255&green=255`: Turn all colors on full brightness (white)
* `url.com/on`: Just turn on the light, without changing any colors

### Final Notes
When starting up the controller for the first time, all lights will go on at full brightness (white). Each time the Raspberry Pi is restarted, the colors and state from the previous session will be retained and applied on start up.
