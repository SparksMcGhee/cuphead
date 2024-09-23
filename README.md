# "cuphead" print server
Setup process for an Ubuntu VM hosting DyeSub printers like my Mitsubishi CP-K60DW-S using the selphy_print repository

## References: 
Initial setup followed: 
https://forums.linuxmint.com/viewtopic.php?t=391775 

with modifications sourced directly from Selphy_Print's documentation: 
https://git.shaftnet.org/gitea/slp/selphy_print/src/branch/master/docs/compiling.txt 

## Assumption: 
- I set this up on Ubuntu 24, virtualized via Proxmox. I don't think the setup process is meaningfully dependant on the exact kernel or virtualization mechanism though. 

## Setup process
- Make an Ubuntu VM. Ensure you have root. 

- Ensure the VM has access to a dye sublimation printer supported by selphy_print. I used ```lsusb```

### Set up CUPS 
- Install CUPS: 
```
sudo apt install cups
```

- Start editing the cups configuration file, subsequent instructions will be for editing this file
```
sudo nano /etc/cups/cupsd.conf
```

- Make CUPS be an actual network service
    - Because that's apparantly **NOT** the default for some reason!
    - Change ```Listen localhost:631``` to ```Listen 0.0.0.0:631```

- Enable sharing the printer on the network: 
    - Change ```Browsing No``` to ```Browser Yes```

- Set CUPS to permit users to connect to the web interface 
    -  If you don't do this it will just reject all attempts to reach its web interfaces. 
    - Add the subnets you want to manage and print from to the ```/``` AND ```/admin``` ```Location``` blocks:
    - For instance, I want to print from the 10.10.10.0/24 CIDR block, or ```10.10.10.*```, so my blocks look like this: 

```
    # Restrict access to the admin pages...
    <Location /admin>
    AuthType Default
    Require user @SYSTEM
    Order allow,deny
    Allow 10.10.10.*
    </Location>
```

```
    # Restrict access to the admin pages...
    <Location /admin>
    AuthType Default
    Require user @SYSTEM
    Order allow,deny
    Allow 10.10.10.*
    </Location>
```

- Restart CUPS to apply the new configuration file 
```
sudo systemctl restart cups
```

- Create a cups admin user, for setting up printers later
```
sudo adduser printadmin
sudo usermod -aG lpadmin printadmin
```

 - Ensure you can access your CUPS server 
    - Navigate to https://[yourServerName]:631 in a browser 

> [!WARNING]
> You MUST use HTTPS as modern browsers won't permit basic authentication prompts over HTTP! 

### Install Gutenprint
 You get a minimal and old version of gutenprint and some other related tools on modern Ubuntu. This all needs to go as selphy_print requires the full and up-to-date versions. 

These instructions are lifted from a forum post by MikeNovember: https://forums.linuxmint.com/viewtopic.php?t=391775

- Remove the minmal/old versions of these packages: 
```
sudo apt remove *gutenprint* ipp-usb sane-airscan
```

- Install support utilities 
```
sudo apt install libusb-1.0-0-dev libcups2-dev git-lfs
```

- Download latest gutenprint snapshot from sourceforge
```
wget https://sourceforge.net/projects/gimp-print/files/gutenprint-5.3/5.3.4/gutenprint-5.3.4.tar.xz
```

- Decompress & Extract
```
tar -xJf gutenprint-5.3.4.tar.xz
```

- Compile gutenprint
```
cd gutenprint-5.3.4
./configure --without-doc
make -j4
sudo make install
cd ..

# Refresh PPDs
sudo cups-genppdupdate
```

- Restart CUPS
```
sudo systemctl restart cups
```

### Install Selphy_Print
- Install prerequisites 
```
apt-get install libusb-1.0-0-dev libcups2-dev libgutenprint-dev
```

- Get the latest selphy_print code
```
git clone https://git.shaftnet.org/gitea/slp/selphy_print.git
```

- Compile selphy_print
```
cd selphy_print
make -j4 
sudo make install
```

- Set up library include path
```
sudo echo "/usr/local/lib" > /etc/ld.so.conf.d/usr-local.conf
sudo ldconfig
```

