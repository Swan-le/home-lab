# How to set up a Layer 3 Wi-Fi Bridge using a Respberry Pi 5

Welcome back to another home lab tutorial, today, our goal is to create a Wi-Fi Bridge to wirelessly connect an old Cisco switches to my home network without needing to cable through my 1902, 2 bedroom apartment. Because, if you’ve ever lived in an old building before, cutting through slats and plaster is not an easy task.

## What you’ll need

Other than the Pi 5 the materials needed for this project are very limited, you’ll only need 2 pieces of equipment to get up and running. For this project we opted for the Pi 5 because… why not… but because it comes with an onboard Wi-Fi card, if you want to save a few bucks, you can downgrade to the pi 4 which also has a built in Wi-Fi Card.

- Raspberry Pi 5 (MicroCenter)
- Micro SD (MicroCenter)
- Ethernet Cable (MicroCenter)
- USB Wi-Fi Adapter - Optional (Amazon)
- Raspberry Pi 5 Case - Optional (Amazon)

I would like to note, for the optional USB Wi-Fi Adapter, if you choose to go down this route, you will need to make sure it has the RT5370 Chipset and compatible with Linux.

## 1: Obtain Raspberry Pi & Flash the OS

Flashing the OS will be the first step in our project, we’ll be running Raspberry Pi OS Lite (Headless). Why headless? For one, it’s a little more difficult to work with, providing a learning challenge. Secondly, this device, once configured, will act solely as a Wi-Fi card for the switches and connected devices, outside of that nothing else will be running on the device. Headless, gives us a little more headroom to allow network traffic to run without other unneeded activities taking up compute space.

I’m using the official Raspberry Pi imager to flash the SD card, you can find that here. Here are some custom settings to be aware of and need to configure.

Customization Settings:

- ***HostName***: This is the name of your device, that shows up on your network once installation is complete.
- ***User***: This is how we will log into our Pi using SSH, or direct connect. Type in your username & password (Don’t forget what your wrote, otherwise you will need to re-flash the drive)
- ***Wi-Fi***: I personally had issues connecting to the Wi-Fi after filling out this information. I’m not exactly sure why this happened, either because my network is set to aWPA3, instead of WPA2, or something else. I’ll discuss how fix this issue later in the article. In the mean time, add your network SSID and Password so the Pi can connect to your LAN during boot.
- ***Remote Access***: Enable SSH. It’s key we ensure SSH Authentication is active. SSH is the only way we’ll be able to access the device wirelessly.

### Network Manager

As of 2022, Pi has introduced Network Manager as an alternative to the old dhcpcd used to configure device networks. Personally, network Manger is a significant increase thanks to its more user friendly experience and added configuration abilities. Gone are the days of editing boot files to add SSID information. NetworkManager now comes preinstalled with the OS Lite version. Access NetworkManager using nmcli.

## 2. Connecting to your device,

Once the OS is flashed, power on your device and let it run for a few minutes before trying to access. I’ve always found this to be helpful the first time to ensure boot up is successful and allowing the network card to connect.

Next we need to find the IP address of the Pi. You can accomplish this by connecting to the device via HDMI or by going to your router. For the router, start by logging into your home network, we are looking for a new connection that matches the HostName you added during the OS Flashing. If you were lucky, and Wi-Fi automatically connected during boot, congrats! For those who’s Pi decided not to connect on boot, like me. We have more work to do before moving on to the next part of the project.

## 2.1 Network Troubleshooting

As previously mentioned, when I input my SSID info and password on the Pi Imager, the device never connected to the network on its own. I had to manually go into NetworkManager and configure to resolve the Wi-Fi Issue.

## 2.2 Accessing through Ethernet or Micro HDMI

I accessed the Pi via SSH/Ethernet at first because I didn’t have a micro-HDMI. To access the SSH, plug the Pi into your home router via Ethernet cable, once booted up, go to your Router and find the IP address. By default the Pi has ETH0 active, which is the Ethernet jack on your Pi. This is a sure proof way of getting access to your Pi. If you opt into using the Micro HDMI, its as straight forward as connecting everything together, and turning on the device.

Once connected and IP in hand, run the ssh command to get access into the Pi. Type `SSH username@IP-address`, you will be prompted for a password and setting up a key for the first time:

```
ssh pi@192.168.1.22
pi@192.168.1.22's Password:
```

If you are getting an error for wrong password, ensure you are using the username of the device, not the HostName. Admittedly, I’ve done this more than a few times myself. Once connected you are good to move forward with the next steps.

## 2.3 NetworkManager SSID Configuration

Next we’ll tackle the pesky Wi-Fi issue we were having. Now that we are connected we need to configure it to the network we want the bridge to run on. Jump over to the Network Manager GUI using:`Sudo nmtui`

NetworkManager, as mentioned before is the new configuration tool for Raspberry Pi offering tons of customization and ease of use.

You should see a blue screen with a grey box, as pictured below. Navigate to “Activate a connection” select the Wi-Fi network of your choice, once the connection is complete, head back to “edit a connection”.

Navigate down to Wi-Fi, select the network of choice, and input the credentials as needed. The network device should already be selected, if not, close out of NetworkManager and use Ifconfig to find the network device and its Device ID. This will typically be Wlan0, which indicates the build in Wi-Fi card.

If you choose to use an external USB Wi-Fi card, make sure you change the device to the USB, for example the image below, shows Device wlan1 as my Wi-Fi adapter.

Mode should be set to “Client”, security should be set to whatever your router is most commonly WPA2 or WPA3. After the password is configured save the settings and head out to the main directory,

Pat yourself on the back, now you are fully connected to Wi-Fi and no longer need that Ethernet connection or HDMI cable. To double check your work, remove the Ethernet cable, allow the device to connect to the network and use Ifconfig or look to your home router page to find the new IP address of your device. If you see a wireless client with the HostName up and running you’re good to move onto the next step.

## 3. Updates
Once you are logged into the Pi we need to update the OS to the latest version, this should be a must for every new device you set up. Run the following command: `sudo apt update` `sudo apt upgrade`.

## 4. Configuring the Wi-Fi Bridge

Now that we have done all the prerequisite work on getting our Pi 5 set up it’s finally time to start the Wifi bridge configure. In step 4, we will need to complete to following:

- Identify the Ethernet device name
- Configure the Wifibridge connection
- Start the Wifibridge connection

## 4.1 Identify the Ethernet device name
Before we start configuring the Wifibridge, we first need to identify what connection we want the bridge on. Lets start with ifconfig. The output will be a series of devices that have network capabilities, Eth0, wlan0, wlan1, lo, etc. More often than not we are looking for eth0. Below is an example of the output you should receive.

```
eth0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.15.0.1  netmask 255.255.255.0  broadcast 10.15.0.255
        ether 2a:cf:67:6c:15:6a  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 17 overruns 0  carrier 0  collisions 0
        device interrupt 106  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 16  bytes 2609 (2.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 16  bytes 2609 (2.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 2b:cf:47:6f:15:6b  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
Eth0 is known as Ethernet connection, wlan0 is known as wireless local area network. Since we are passing everything through Ethernet, eth0 is what we want. 

## 4.2 Configure the Wi-Fi Bridge

Now that we’ve identified the connection for our Wifibridge, we will input a command that reconfigures the Wifi card to share all traffic with the Ethernet port, hence giving us the Wifibridge we want. Thanks to NetworkManager, this is done in one line. You will need to change `[interface]` to the actual network device name that you found, in my case that would be “eth0”. 

```
sudo nmcli c add con-name wifibridge type ethernet ifname [INTERFACE] ipv4.method shared ipv6.method ignore
```

Breaking down the Command:
- nmcli - Command line tool to access NetworkManager
- C - Short for Connection, tells nmcli to start a connection profile
- con-name wifibridge - short for connection.name, “Wifibridge” is an arbitrary name humans use to remember what the connection is.
- type ethernet - Specifies the connection type as “Ethernet”
- ifname [Interface] - Ifname, short for Interface Name, identifies what hardware port we want to assign the profile to.
- ipv4.method shared - Configures IPv4 to shared mode. This enables the interface to act as a DHCP/NAT server.
- IPv6.method ignore - Sets IPv6 addressing to Ignore or disabled. Entirely skips IPv6 addressing to avoid potential issues.

Now that the bridge is up, we need to turn it on. Again, done with a one line command: `sudo nmcli con up wifibridge`

## 4.3 Double checking our work

Before we can hi-five each other, lets ensure everything worked out properly. Run the following command to ensure the bridge is up and running with the correct device. `nmcli con show`

If everything worked, you should have an output similar to mine:

```
NAME             UUID                                  TYPE    DEVICE 
wifibridge          93cef4e4-0c3a-428c-cb05-fcdb5c224a4c  ethernet eth0   
Tacos               8c6e2c2f-c01a-4984-ba9c-4e12c06022e1  wifi     wlan1  
lo                  29a61894-7c8d-6f5e-9045-a0377762fe5f  loopback  lo     
Wired connection 1  eabad213-36c0-303c-9ace-714485001b99  ethernet  --
```

Notice, the device associated with each name, this is important, if there is no device name assigned to the wifibridge, you’ll have to go back and associate the device with the bridge.

Thanks for reading, I’ll be doing more of these in the future! 
