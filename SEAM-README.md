# BalenaOS build from source for Seam Hardware

# Forward Reading
Take a look at this: [https://www.balena.io/os/docs/custom-build/](https://www.balena.io/os/docs/custom-build/)

  
# Hardware Requirements  
- A [compatible](http://www.yoctoproject.org/docs/3.1/ref-manual/ref-manual.html#hardware-build-system-term) Linux distrobution
- At least 50GB of free disk space
- More CPU, more RAM the better. 16GB RAM and 4 CPU cores recommended.
- Network interface to the outside world.

All instructions are known to work with Ubuntu 18.04 LTS. Thus, it is recommended to use that distribution. All instruction below are written in reference to Ubuntu 18.04 LTS. Your mileage may vary with other distros. No full desktop version required, the server version will work fine for the build.

# Build Package Prerequisites
## Docker
Taken from here:
[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

```
sudo apt-get update

sudo apt-get install apt-transport-https ca-certificates curl \

gnupg-agent software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \

"deb [arch=amd64] https://download.docker.com/linux/ubuntu \

$(lsb_release -cs) stable"

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```
  

## Add the Docker user group and add the standard user

```
sudo groupadd docker

sudo usermod -aG docker $USER

newgrp docker
```
  
  
## Install Required Build Packages
```
sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \

build-essential chrpath socat cpio python3 python3-pip python3-pexpect \

xz-utils debianutils iputils-ping python3-git python3-jinja2 \

libegl1-mesa libsdl1.2-dev pylint3 xterm

sudo apt-get install npm jq
```

# Setup Build Area
```
mkdir ~/workspace; cd ~/workspace/ # or where you want the build to live

git clone https://github.com/balena-os/balena-beaglebone

cd balena-beaglebone

git submodule update --init --recursive

git checkout develop/v2.47.1+rev2+seam --recurse-submodules
```

# Build that OS Image
## Shell Setup
### Always Run
`newgrp docker` first.

### First Build
- Run `./balena-yocto-scripts/build/barys -d --dry-run`
-   Edit `build/conf/local.conf` and adjust the following:
	- `MACHINE=beaglebone-green-wifi`
	- `DEVELOPMENT_IMAGE = "1"`
	- `RESIN_RAW_IMG_COMPRESSION = "xz"`
    
### Incremental Builds
- Setup the shell environment with:
	```
	cd ~/workspace/balena-beaglebone/
	source layers/poky/oe-init-build-env
	```
## Invoke the Build Command
From the build directory, run the following. If it's the first build, this can take many hours. Start it before bed.
```
bitbake resin-image-flasher
```
Wait for success.

  
# Installing Images to the Target Device
- The resulting image will reside at: `balena-beaglebone/build/tmp/deploy/images/beaglebone-green-wifi/resin-image-flasher-beaglebone-green-wifi-<timestamp>.rootfs.resinos-img.xz`
- Use Etcher or the Balena CLI to flash the device. CLI example: `sudo balena local flash <path-to-flasher-image>`
- To make the device talk to your Balena Cloud account to install the `config.json` file associated with the Balena account / app and app on the `boot` partition of the SD card. The `config.json` file can be generated with the Balena CLI: `balena config generate --app <balena-cloud-app-name> --version <os-version>`. See [usage]([https://www.balena.io/docs/reference/balena-cli/#config-generate](https://www.balena.io/docs/reference/balena-cli/#config-generate)) for details.

# Useful Tips for the Target Device
##### Editing OS Config Files to Persist on Reboot
-  Remount the running OS root filesystem as read-write `mount -o remount,rw /`
##### Playing with New Device Tree Overlays
- Get a FTDI cable for serial console debug. It plugs directly into the J1 header. If you don't have serial console, then none of this section matters, and don't play with device trees. Console UART settings are `115200 / 8 / N / 1`.
- When testing a new device tree install device tree overlays to `/mnt/sysroot/active/` this will be the root directory when U-boot pulls the device tree overlay file. Thus the corresponding `/mnt/boot/uEnv.txt` should read something like should name your tested device tree file as: `uboot_overlay_addr4=/my-cool-new-device.dtbo`
- Make sure to make a backup of `/mnt/boot/uEnv.txt` when editing it. If there is an issue with your device tree it will not boot to Linux and the board will sit at the U-boot prompt. If this happens update the environment variables to name your backup `uEnv.txt` file, e.g. `uEnv.txt.bak` and tell that puppy to boot. E.g. 
	```
	# sad no linux boot time
	=> setenv bootenv uEnv.txt.bak
	=> setenv bootenvfile uEnv.txt.bak
	=> boot # happy linux boot time!
	```
