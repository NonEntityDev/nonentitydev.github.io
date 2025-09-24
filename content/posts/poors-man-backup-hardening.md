+++
date = '2025-09-24T12:35:40+01:00'
title = 'My poor man backup and NAS: Part 2 - Hardening'
categories = ['Automation']
tags = ['backup', 'linux', 'bash']
summary = 'Now that we have our backukp strategy well defined and planned, lets start the implementation hardening our Raspnerry PI to ensure the minimum security of our data, while keeping it accessible and easy to maintain.'
+++

## TL;DR
* We automated the update of our system using the `unattended-upgrades` package.
* We disabled insecure default configurations, unnecessary services, and hardened privileges to reduce the attack surface of our system.
* A firewall is a must-have for any system that aims to be minimally secure. We blocked all incoming traffic by default, allowed all outgoing traffic, and limited access to our SSH service.
* Finally, we reduced the risk of DDoS and brute-force attacks on our SSH service using `fail2ban`.

## Introduction

In the [first part](https://nonentitydev.github.io/posts/poors-man-backup-introduction/) of this series we planned what we needed to do to have a well-sized and low-cost personal backup solution. From now on, we will start implementing this plan. I won't get into how to install and configure a Raspberry Pi because the internet is already oversaturated with tutorials and videos on how to do that. It is sufficient to say that I have installed a headless Raspberry OS (Lite version without GUI), and I am starting from a point where it is already connected to my local network and SSH is enabled.

This post aims to describe in detail all the steps I took to harden my installation to provide the minimum protection for my data.

## Defining an static IP address

This step is fundamental to save us a lot of trouble. Local domestic networks are usually configured to use DHCP, which is great—but not for a device that we want to connect to remotely on a regular basis. Assuming that you know and have already created a range of reserved IP addresses in your router, let's configure our Raspberry OS to use one of those addresses statically.

In my network, I am using the `192.168.4.*` range and have reserved the first 20 addresses in it. I will be using `192.168.4.3` for my Raspberry Pi.

First, let's discover the names of the networks available on our device:

```bash
nmcli con show
```

This should output something like mine:

```text
NAME                UUID                                  TYPE      DEVICE 
preconfigured       2cf1fc99-38ce-43bf-960b-c89f68742a6f  wifi      wlan0  
lo                  5bad99d3-397d-4e8f-a303-a9a1732a44d6  loopback  lo     
Wired connection 1  05476bf3-cee3-3230-a1c8-1ccf2d92ad74  ethernet  -- 
```

Let's rename our Wi-Fi network to something more intuitive and easier to use than ```preconfigured```:

```bash
sudo nmcli con mod 'preconfigured' connection.id 'wifi'
```

And then, apply the static IP address configuration:

```bash
sudo nmcli con mod 'wifi' ipv4.addresses 192.168.4.3/24
sudo nmcli con mod 'wifi' ipv4.gateway 192.168.4.1
sudo nmcli con mod 'wifi' ipv4.dns '192.168.4.1'
sudo nmcli con mod 'wifi' ipv4.method manual
```

It’s not strictly required, but at this point I like to restart the system to make sure the configuration is not rejected by my Wi-Fi router.

## Updating everyting and keeping it updated

It is very important to always keep our installation and applications updated to apply fixes for recently discovered bugs and vulnerabilities. Let's start by updating everything manually, and then automate the process.

```bash
sudo apt-get -y update && sudo apt-get -y upgrade
sudo apt-get -y autoremove
```

Now, let's install the ```unattended-upgrades``` package to automate updates for us.

```bash
sudo apt-get install -y unattended-upgrades
```

Enable, start and check the unattended-upgrades service:

```bash
sudo systemctl enable unattended-upgrades
sudo systemctl start unattended-upgrades
sudo systemctl status unattended-upgrades
```

This should output something like:

```text
● unattended-upgrades.service - Unattended Upgrades Shutdown
     Loaded: loaded (/lib/systemd/system/unattended-upgrades.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-09-01 21:37:55 IST; 3 weeks 1 day ago
       Docs: man:unattended-upgrade(8)
   Main PID: 625 (unattended-upgr)
      Tasks: 2 (limit: 1581)
        CPU: 243ms
     CGroup: /system.slice/unattended-upgrades.service
             └─625 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal

Sep 01 21:37:55 trantor systemd[1]: Started unattended-upgrades.service - Unattended Upgrades Shutdown.
```

Next, let's do a dry-run just to be sure that everything is working properly.

```bash
sudo unattended-upgrade -d --dry-run
```

Finally, to inspect it execution we can check the logs:

```bash
sudo tail -f /var/log/unattended-upgrades/unattended-upgrades.log
```

By default, unattended-upgrades will always updated packages from the **stable** channel.

## Changing SSH port

By default, SSH daemon uses the TCP port 22, which is a low port. We need to change the port to a high port like 2222.

* Use your favorite editor (in my case I'm using ```vim```) to edit the file ```/etc/ssh/sshd.config```.
* Find the line ```#Port 22```.
* Change it to ```Port 2222```.

After saving the file, restart ```sshd``` service.

```bash
sudo systemctl restart sshd
```

## Disabling the default ```pi``` account (if it exists)

```bash
sudo usermod --lock --expiredate 1 pi
```

## Installing a firewal

Our policy to protect our system is to **allow any outgoing traffic by default, denying any incoming traffic by default, and limit access to our ssh service**. I'm using ```ufw``` as firewall, which I installed using the command:

```bash
sudo apt-get install -y ufw
```

Next, let's define our basic rules as described above:

```bash
sudo ufw default deny incoming # Deny all incoming traffic by default.
sudo ufw default allow outgoing # Allow all outgoing traffic by default.
sudo ufw limit 2222/tcp comment 'SSH Port rate limit' # Limit incoming SSH connections.
sudo ufw logging on # Enable logging for troubleshooting.
```

After that, enable ```ufw```:

```bash
sudo ufw enable
sudo systemctl enable ufw
sudo systemctl start ufw
```

Finally, let's inspect our rules:

```bash
sudo ufw status numbered
```

Which should output:

```text
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 2222/tcp                   LIMIT IN    Anywhere                   # SSH port rate limit
[ 2] 2222/tcp (v6)              LIMIT IN    Anywhere (v6)              # SSH port rate limit
```

## Disabling unnecessary services

Bluetooth is enabled by default with Raspberry OS. If you don't need it, disable it with:

```bash
sudo systemctl stop bluetooth
sudo systemctl disable bluetooth
```

Check if there is any other unecessary service enabled in your system using the following command:

```bash
sudo service --status-all
```

And repeat the steps above for any service that you find unecessary. This is important to reduce the atack surface of our system.

## Making ```sudo``` require password

Again, using your favorite text editor, edit the file ```/etc/sudoers.d/010_pi-nopasswd``` and change the following line:

```text
<userid> ALL=(ALL) NOPASSWD: ALL to <userid>
```

To...

```text
<userid> ALL=(ALL) PASSWD: ALL
```

Save the file. This is a good opportunity to restart our system and test everything that we've done so far.

## Avoiding brute-force and DDoS attacks with  ```fail2ban```

So far we've taken actions to keep our system up to date, reduced the attack surface by disabling unnecessary services, configured a firewall with simple but effective rules and disabled insecure default credentials and configurations.

Lastly for this post, we need to prevend DDoS and brute-force attacks, specially against our SSH service which is still using user/password authentication. ```fail2ban``` is a service that helps us to achieve that by dynamically denying access to hosts that failed multiple authentication attempts in our system.

Lets install it using:

```bash
sudo apt-get install -y fail2ban
```

By default, ```fail2ban``` denies access to any host for 1 hour after 5 failed login attempts. We need to enable the default configuration by copying the default configuration file to the proper path:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Edit ```/etc/fail2ban/jail.local```, find the **[sshd]** section, and make it look like this:

```ini
[sshd]
mode = normal
enabled = true
port = 2222
logpath = %(sshd_log)s
backend = systemd
```

After saving the changes, enable, start and inspect the service status:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo systemctl status fail2ban
```

The last command should output something like:

```text
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-08-19 12:15:16 IST; 19s ago
       Docs: man:fail2ban(1)
   Main PID: 2785 (fail2ban-server)
      Tasks: 5 (limit: 1581)
        CPU: 382ms
     CGroup: /system.slice/fail2ban.service
             └─2785 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

Aug 19 12:15:16 trantor systemd[1]: Started fail2ban.service - Fail2Ban Service.
Aug 19 12:15:16 trantor fail2ban-server[2785]: 2025-08-19 12:15:16,853 fail2ban.configreader   [2785]: WARNING 'allowipv6' not defined in 'Definition'. Using default one: 'auto'
Aug 19 12:15:17 trantor fail2ban-server[2785]: Server ready
```

```fail2ban``` generates logs on ```/var/log/fail2ban.log``` and rotates them from time to time. The logs are very useful to troubleshooting and verifyin that the service is working properly.

## Conclusion.

Now we have a system with the minimal security in place to start configuring our NAS and backup solution. In the next post I plan to cover the preparation of my storages units and configure SAMBA to provide a shared folder for my local network.