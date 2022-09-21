# Pynq2.7OnZedboard
Pynq2.7 image for Zedboard

Not saying that it is not out there, but I was unable to find a working ready-to-run Pynq2.7 image for the Zedboard.
Admitted, it was not a walk in the park but eventually I was able to successfully generate such image.

(workable) Instruction on how to compile are ~~not yet(fully)~~ included...\
Meanwhile, please feel free to download and use my try at PYNQ2.7 for Zedboard.

https://buls.be/public/img/PYNQ/ZED-2.7.0.img

# Instruction on how to compile the PYNQ2.7 for the ZED board

#### OS environment
Download and install Ubuntu 18.04.
(note) The ubuntu website, by default, suggests you to download version 18.04.6.
While not officially supported by Vivado 2020.2, Ubuntu 18.04.6 will work.

I used Ubuntu 18.04.6 within VirtualBox with sufficient RAM (>=12GB), Disk space (>=150GB) and CPU’s (>=4), Video Memory (>=96MB)
The default number of CPUs is 1 and results in a kernel panic when booting the Ubuntu ISO.
I noticed that the default Monitor Memory is too low resulting in a black screen on a secondary monitor after Ubuntu boots.
You can still changes these parameters after the virtual machine is created.
Prepare Ubuntu
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential
sudo apt-get install vim git
```
Install Virtual Box Guest addition software (that is why we needed the build-essential)\
Reboot\
Download Xilinx Unified 2020.2
```
cd ~/Downloads
chmod 775 Xilinx_Unified_2020.2_1118_1232_Lin64.bin
sudo mkdir -R /tools/Xilinx
sudo chmod 775 -R /tools/Xilinx
sudo chown root:<username> -R /tools/
./ Xilinx_Unified_2020.2_1118_1232_Lin64.bin
```
During the installation, do not press ‘Get Latest’.\
On the Welcome screen you’ll be informed that you are running an unsupported version of Ubuntu. You can ignore this and press Next.\
On the Select Install Type,\
Fill in you Xilinx user credentials and press Next.

On the Select Product to Install,\
Select Vitis and press Next.\
On the Vitis Unified Software Platform,\
(optionally) Deselect items/devices that you are not going to use to limit download time and save disk space.\
On the Accept License Agreements,\
Select all ‘I Agree’ and press Next\
Rerun the installation to install Petalinux
```
./Xilinx_Unified_2020.2_1118_1232_Lin64.bin
```
Press Continue when presented for a newer version\
On the Welcome page, Select Next
On the Select Install Type, Fill in your Xilinx user credentials and press Next.\
On the Select Product to install page, Select Petalinux (Linux only) and press Next\
On the Accept License Agreements, Select all ‘I Agree’s and press Next.\
On the Select Destination Directory, Press Next\
On the Installation Summary, Press Install\
After I successful installation, a message pups up referring to a missing requirement.

```
sudo apt-get install gawk
```

Configure/install Petalinux

```
cd /tools/Xilinx/PetaLinux/2020.2/bin/
./petalinux-v2020.2-final-installer.run
```
Run through all the licenses and accept the suggested default installation directory.\
Accept that the installation directory is not empty.

#### Patch Vivado (y2k22)
Download and install the Vivado y2k22 patch: https://support.xilinx.com/s/article/76960?language=en_US\
Follow the instructions provided within the README file.\
You have to provide the python version, which is version 3.\
Execute the following command to successfully execute the patch.

```
Vivado/2020.2/tps/lnx64/python-3.8.3/bin/python3 y2k22_patch/patch.py
```

#### PYNQ environment
Get the PYNQ environment from git
```
cd ~/
git clone https://github.com/Xilinx/PYNQ.git
```
### Board folder/files
Create a dedicated board folder for the Zedbaord under /PYNQ/boards/\
You'll notice that the /PYNQ/boards/ folder already contains supported board folders.\
(note) The makefile will scan the /PYNQ/boards/ folder to enable/set build parameters.\
(optionally) To limit the possibility of something going wrong, I deleted all supported boards folders/files and ending up with only the Zedboard board folders and files.

```
rm -fr ~/PYNQ/boards/Pynq-Z1
rm -fr ~/PYNQ/boards/Pynq-Z2
rm -fr ~/PYNQ/boards/ZCU104
mkdir ~/PYNQ/boards/ZED
```
#### Board Support Package
Download the BSP file for the Zedboard from Xilinx.\
This BSP file should have the same version as your PetaLinux/Vivado environment (2020.2).\
You’ll notice that this version is published under the ‘Archive’ section.\
The BSP for the Zedboard is roughly 100MB.\
You will to provide your Xilinx credentials to start the download.

```
cp ~/Downloads/avnet-digilent-zedboard-v2020.2-final.bsp ~/PYNQ/boards/ZED/
```
#### Root file system
When compiling Pynq, the rootfs will be fetched from different mirror servers.\
I noticed that some packaged and/or dependencies are not found resulting in a failed compilation. As a work around, download the ‘prebuilt board-agnostic image’.\ This image contains all packages/dependencies required to compile the rootfs.\
You can find a link to this image om the pynq.io website (http://www.pynq.io/board.html).\
Download the ‘PYNQ rootfs arm v2.7’ version and copy/move it to the ZED folder.

```
cp ~/Downloads/focal.arm.2.7.0_2021_11_17.tar.gz ~/PYNQ/boards/ZED/
```

#### spec file
Create the ~/PYNQ/boards/ZED/ZED.spec file and copy the following content to it
```
ARCH_ZED := arm
BSP_ZED := avnet-digilent-zedboard-v2020.2-final.bsp
FPGA_MANAGER_ZED := 1
STAGE4_PACKAGES_ZED := xrt pynq ethernet pynq_peripherals
```
#### Dependencies
Install all dependencies by executing the provided setup script
```
./PYNQ/sbuild/scripts/setup_host.sh
````
You also need to have the HDMI IP license! (evaluation is ok)\
Very important, if you don’t have this the compilation will fail!\
In order do receive the HDMI IP you have to launch Vivado, press help and click on the license manager.\
You need to search for the HDMI ip on the webpage that opens and link/bind it to the device that is going to use it. I selected the bind based on MAC address.\
As a result, you will receive an email with a .lic file that you can import in the license manager.\

#### Some additional stuff

There is a check for qemu-arm version in the Makefile, uncommend this line (nasty, I know...)

Install crosstool-ng
```
cd
git clone https://github.com/crosstool-ng/crosstool-ng
cd crosstool-ng
./bootstrap
./configure
make
make install
```
Depending on which bash/shell terminal you use, add the PATH export to either/both .profile and/or .bashrc.\
(example PATH=/opt/qemu/bin:/opt/crosstool-ng/bin:$PATH)

### Build the PYNQ image
Source the appropriate settings for Vitis and PetaLinux.
```
source <path-to-vitis>/Vitis/2020.2/settings64.sh
source <path-to-petalinux>/petalinux-2020.2-final/settings.sh

make PREBUILT=~/PYNQ/boards/ZED/focal.arm.2.7.0_2021_11_17.tar.gz BOARDS=ZED
```

After a couple of hours you will have your ZED.img that you can flash to an SD card!

