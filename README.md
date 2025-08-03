# Wireless-Hacking-101
Understanding Wireless Attack
Before we get deep into the wireless attack itself, there are some key terms that we need to understand:

SSID: The network “name” that you see when you try and connect
ESSID: An SSID that may apply to multiple access points, eg a company office, normally forming a bigger network. For Aircrack they normally refer to the network you’re attacking
ESSID + Password = PSK (Wireless Key)
BSSID : An access point MAC (hardware) address
WPA2-PSK: Wifi networks that you connect to by providing a password that’s the same for everyone
WPA2-EAP: Wifi networks that you authenticate to by providing a username and password, which is sent to a RADIUS server
RADIUS: A server for authenticating clients, not just for wifi
Zoom image will be displayed

Image Credit : https://www.airtel.in/blog/broadband/how-to-change-wifi-password-and-name/
The core of WPA authentication is the 4-way handshake

The 4-way handshake is the process of exchanging 4 messages between an access point (authenticator) and the client device (supplicant) to generate some encryption keys which can be used to encrypt the data that is being sent over the wireless medium. This is the main target of a wireless penetration testing.

Preparing The Network Adapter Device & Driver
The product I am using is AWUS036NHA. The official page only has drivers that supports Linux & Windows up to 8. So, if you are using Win 10/11 then go visit the blog.

AWUS036NHA - ALFA Network Docs

Using Alfa AWUS036NHA on Microsoft Windows 10 & Windows 11
<img width="720" height="402" alt="image" src="https://github.com/user-attachments/assets/d16d8117-0c97-4669-ad02-d6636fcfb983" />

Zoom image will be displayed

Starting The Network Interface
First you need to find out the name of your wireless network adapter, it will probably be wlan0, but to check it run ifconfig and then to double check, run iwconfig

Zoom image will be displayed
<img width="720" height="865" alt="image" src="https://github.com/user-attachments/assets/f33fa739-d648-4da4-8585-7c4830a1974e" />

Zoom image will be displayed
<img width="720" height="318" alt="image" src="https://github.com/user-attachments/assets/652dae90-3e39-4201-8964-45f297ffdd40" />

Normally a Wi-Fi adapter is set into “managed” mode which means it just acts as a client and connects to a single Wi-Fi router for access to the Internet. However, some Wi-Fi adapters can be set into other modes. In monitor mode the Wi-fi interface can capture packets without even being connected to any access point (router), it is a free agent, sniffing and snooping at all the data in the air!

Next to put the card into monitor mode we can use airmon-ng.

sudo airmon-ng start [network-interface]
As a disclaimer, not all adapters/cards supports this, so you must make sure you are using a compatible adapter.

Zoom image will be displayed
<img width="720" height="309" alt="image" src="https://github.com/user-attachments/assets/185b9f84-9e5f-4b73-bbca-bd7b33a06c7a" />

This will create a new virtual interface called wlan0mon. You can double check it using the iwconfig

Zoom image will be displayed
<img width="720" height="291" alt="image" src="https://github.com/user-attachments/assets/a7975cde-c03b-45e0-a404-344cdbbd6e3c" />

Kill any processes that might interfere with the network adapter.

sudo airmon-ng check kill
Zoom image will be displayed
<img width="720" height="324" alt="image" src="https://github.com/user-attachments/assets/6cbaad81-8002-45f7-8a30-c6ce4b63a92f" />

Capture Traffic & Nearby Wifi Networks
Wi-Fi uses radio and like any radio it needs to be set to a certain frequency. Wi-Fi uses 2.4GHz and 5GHz (depending on which variation you are using). The 2.4GHz range is split into a number of channels which are 5MHz apart. To get two channels which don’t overlap at all they need to be spaced around 22MHz apart (but that also depends on which variation of the Wi-Fi standard is being used). That is why channels 1, 6 and 11 are the most common channels as they are far enough apart so that they don’t overlap.

To capture data via a Wi-Fi adapter in “monitor” mode you need to tell the adapter which frequency to tune into, i.e. which channel to use. To see which channels are in use around you and which channel is being used by the Wi-Fi service you wish to test then use the airodump-ng command:

// airodump-ng will display a list of detected access points
// and also a list of connected clients (“stations”).
// Use this to find the bssid & the channel of the target network
sudo airodump-ng [network-interface]
Zoom image will be displayed
<img width="720" height="693" alt="image" src="https://github.com/user-attachments/assets/97318efa-631b-4f62-a1cd-536195b9b6a2" />

Zoom image will be displayed

The first list shows the Wi-Fi networks within reach of your laptop. The CH tells you which channel number each network is using and the ESSID shows the names of the networks (i.e. the service set identifiers). The ENC column reveals if the network is using encryption and if so, what type of encryption. If you see OPN as the ENC it means it’s a public network.

Get Mario Rufisanto’s stories in your inbox
Join Medium for free to get updates from this writer.

Enter your email
Subscribe
From the results, you can see a wifi hotspot that I have prepared called “POCO X3 Pro”. So, the next step is to capture the packets usign airodump-ng

// Capture the packets and export it to your machine
sudo airodump-ng --bssid [target-bssid] -c [channel-id] --write [filename] [network-adapter]
Zoom image will be displayed
<img width="720" height="184" alt="image" src="https://github.com/user-attachments/assets/3f4448ff-cc3e-4878-85a6-5ba0944c8c23" />

Zoom image will be displayed

There is one device that is connected to the wifi hotspot that I have prepared. So, while that is running, you’re going to run your de-authentication attack against the device connected to make the device re-establish a connection so you can capture the 4-way handshake.

// Deauth the users that are connected to the target network
// User will need to input the wifi password to connect again
sudo aireplay-ng --deauth [number-of-packets(>50)] -a [target-bssid] [network-adapter]
Zoom image will be displayed
<img width="720" height="451" alt="image" src="https://github.com/user-attachments/assets/1caab3ac-9b2f-4dff-9d55-cc526c8f3556" />

Zoom image will be displayed

// If there is an error caused by the channel like the image bellow,
// Restart the interface with the target channel
sudo airmon-ng start [network-adapter] [target-channel]
Zoom image will be displayed
<img width="720" height="410" alt="image" src="https://github.com/user-attachments/assets/1049d0f1-9fdc-41f2-a6d3-035a4b54c70d" />

Zoom image will be displayed

While the DOS attack is underway, check on your airodump scan. You should see at the right top : WPA handshake: <mac address>. Once you have verified that, you can stop the replay attack and the airodump-ng scan.

Zoom image will be displayed
<img width="720" height="173" alt="image" src="https://github.com/user-attachments/assets/d2905894-63ac-491f-ab79-07b8d40812e7" />

Zoom image will be displayed

Crack the Password
In the final steps, we are going to run a bunch of generated Pairwise Master Key (PMK) against the captured packets to brute force the password.

A PKM is an algorithmic combination of a word and the APs name. So, our target is to continuously generating PMKs using a wordlist against the handshake until a valid PMK is found.

// Everytime you restart the capture, another file will be made
// So make sure to choose the right file
Zoom image will be displayed

There are several wordlist that can be used to crack the password and one of the most popular one is rockyou. The rockyou wordlist is a bunch of passwords gotten from one of the most infamous cybersecurity data breaches that affected a company of the same name. It contains approximately 14 million unique passwords that were used in over 32 million accounts and as such, is one of the most dependable wordlists on the planet.

// Crack the wifi password using aircrack
sudo aircrack-ng -b [target-bssid] [packet-file(.cap)] -w [wordlist]
Zoom image will be displayed

Because my wifi hotspot password isn’t in the rockyou wordlist. I created another dummy wordlist that contains my wifi password.
<img width="720" height="59" alt="image" src="https://github.com/user-attachments/assets/37ae4c5a-bdea-4eff-b13a-474dd1e3410d" />

Zoom image will be displayed
<img width="720" height="593" alt="image" src="https://github.com/user-attachments/assets/d0835266-7adc-460c-aa14-dc7720bd44d8" />

Mitigations Againts Wifi Attacks
Basic Wi-Fi security should cover this attack from a defensive perspective. Using WPA3 which is a newer protocol is your best bet against such an attack. To mitigate against de-authentication attacks, use an ethernet connection if possible.

Assuming that option is not on the table, you can use a strong passphrase (not a password) to minimise the attackers chances of getting it. A passphrase is a string of words simply used as a password. Passphrases tend to be longer than passwords, easier to remember, and are a rarer practice. Therefore, they will hardly be found in wordlists.

For example, potato is more likely to be found in a wordlist than bryanlovespotato. The later is a 16-character passphrase and as simple as it is, it would be hard for an attacker to find, guess, or generate.

Another mitigation would be to disable WPS (Wi-Fi Protected Setup) and avoid under any circumstance using a router that uses the WEP protocol. You’d just be asking for unwanted attention as it’s a lot easier to hack both of these than WPA2.
