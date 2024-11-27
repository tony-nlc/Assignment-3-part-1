# Assignment-3-part-1

The goal of this assignment is to setup a Bash script that generates a static index.html file that contains some system information. The script will be configured to
run automatically every day at 05:00 using a systemd service and timer. The HTML document created by this script will be served with an nginx web server that will
run on your Arch Linux droplet along with a firewall setup using ufw to help secure your server.

---

# Task 1
## 1.1 Create User
In this part, We will create a new system user `webgen` with a home directory at /var/lib/webgen and a login shell appropriate for a non-login user. 
```bash
sudo useradd -r -m -d /var/lib/webgen -s /usr/bin/nologin webgen
```
- useradd allows us to add a new user to /etc/passwd [https://ss64.com/bash/useradd.html]
- -r create a system account
- -m specifies create the home directory if it does not exist
- -d specifies the home directory
- /var/lib/webgen is the location we are creating the user's directory
- s specifies the shell
- /usr/bin/nologin is the regular shell to refuse a login [https://wiki.archlinux.org/title/Users_and_groups]
## 1.2 Create Home Directory Structure 
In this part, we will create a system users home directory with the folloinwg structure
```
/var/lib/webgen/
├── bin/
│ └── generate_index
└── HTML/
└── index.html
```
For creating the structure
```bash
sudo mkdir -p /var/lib/webgen/bin
sudo mkdir -p /var/lib/webgen/HTML
sudo chown -R webgen:webgen /var/lib/webgen
sudo chmod -R o+wr /var/lib/webgen 
```
- mkdir create directory
- -p will create directory if parent directories as needed[https://ss64.com/bash/mkdir.html]
- chown -R webgen:webgen /var/lib/webgen change the ownership of /var/lib/webgen to user `webgen` and group `webgen` 
For getting the file, we need to git clone the repository. Then, we will move the generate_index file to bin. Then, we will grant execute access to `generate_index`.
```bash
cd /var/lib/webgen
git clone https://git.sr.ht/~nathan_climbs/2420-as3-p2-start
sudo mv 2420-as3-p2-start/generate_index bin/
sudo rm -rf 2420-as3-p2-start
chmod +x bin/generate_index
```
The reason we use system user instead of regular user or root user are,
- root user has too much privilege. It violates least privilege principle.
- system users are usually created specifically for running system services and applications. [https://www.freecodecamp.org/news/how-to-manage-users-in-linux/#heading-type-of-users-in-linux]
- regular users are created by the administrator and can access the system and its resources based on their permissions. [https://www.freecodecamp.org/news/how-to-manage-users-in-linux/#heading-type-of-users-in-linux]
---
# Task 2
## 2.1 Create Service File
Before we can start a service, we need a service file. At the same time, we need the service run everyday at 05:00. Therefore, we need a timer file. Let's start with creating service file.

```bash
sudo nvim /etc/systemd/system/generate-index.service
```
Then, copy and paste the following into your service file
```bash
# Filename:generate-index.service 
[Unit]
Description=Run generate_index
Wants=network-online.target
After=network-online.target

[Service]
User=webgen
Group=webgen
ExecStart=/var/lib/webgen/bin/generate_index

[Install]
WantedBy=multi-user.target
```
Breakdown of `generate-index.service`:
- Description is a brief explanation for the service
- We will use Wants and After if the dependcy is optional[https://wiki.archlinux.org/title/Systemd]
- User=webgen implies the service only has the permissions granted with the `webgen` user
- Group=webgen implies the service only has the permissions granted with the `webgen` group
- ExecStart implies the script it is running on the start

## 2.2 Create Timer File
We will be creating timer file for our service. Timer files are useful if we need the service to run regularly.
```bash
sudo nvim /etc/systemd/system/generate-index.timer
```
Then, copy and paste the following into your service file
```bash
# Filename:generate-index.service 
[Unit]
Description=Run generate_index every 05:00

[Timer]
OnCalendar=*-*-* 05:00:00
Persistent=true

[Install]
WantedBy=timers.target
```
Breakdown of `generate-index.timer`:
- Oncalendar=*-*-* 05:00:00 specifies for everyday's 05:00:00 in UTC the timer will trigger
- Persistent means if the machine is down at the time, the service will run after it boot
- WantedBy=timers.target implies the timer will be active after boot[https://wiki.archlinux.org/title/Systemd/Timers]

If you want to check the timer is active and the services runs successfully, use the following commands.
```bash
sudo systemctl status generate-index.service
sudo systemctl status generate-index.timer
```
For service,
![alt text](https://github.com/tony-nlc/Assignment-3-part-1/blob/main/assets/service.png)
For timer,
![alt text](https://github.com/tony-nlc/Assignment-3-part-1/blob/main/assets/timer.png)


---