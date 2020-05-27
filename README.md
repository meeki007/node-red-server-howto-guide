
## Ubuntu 20.04 - nigix - https - ssl - security - Fail2ban 403 bans, etc

## Table of Contents
* [Introduction](#Introduction)
* [Setup_Server](#Setup_Server)
    * [Creating_a_new_privileged_user_account](#Creating_a_new_privileged_user_account)
    * [Update_Server](#Update_Server)
    * [Enable_a_firewall](#Enable_a_firewall)
    * [Install_Fail2ban](#Install_Fail2ban)
* [Node-red](#Node-red)
    * [build-essential](#build-essential)
    * [nod-red_script](#nod-red_script)
    * [Autostart_on_boot](#Autostart_on_boot)
* [Hosting_to_the_world](#Hosting_to_the_world)
    * [Install_Nginx](#Install_Nginx)
    * [DNS](#DNS)
    * [Reverse_proxy](#Reverse_proxy)
    * [HTTPS_SSL_certbot](#HTTPS_SSL_certbot)
* [Security](#Security)
    * [adminAuth_user_password](#HTTPS_SSL_certbot)
    * [fail2ban_403_protection](#fail2ban_403_protection)
* [Extras](#Extras)
        * [automatic_increasing_ban_times](#automatic_increasing_ban_times)




## Introduction
This Guide is for setting up a <b>secure</b> Dedicated Server.
<br>
Running node-red
<br>
This guide assumes you have a running server with Ubuntu 20.04 and that you have access to terminal on the server. You can host this yourself however this guide does not address port forwarding from routers or DMZ settings and or ipv6 settings.

You need a Domain Name and the ability to point it to your IP address. Let’s Encrypt doesn’t issue certificates for bare IP addresses, only domain names. You’ll need to register a domain name in order to get a Let’s Encrypt certificate.

What your server needs: a visable IP address to the internet to include if you wish a ipv6 address.

I used a VPS (Virtual Private Server) provider. For less than $10 a month, I have a server, and I have complete root access to and can spin up Ubuntu on. I used https://servercheap.net/pricing.php I was not paid or compensated to link them. Its just what I use.
<br>
I suggest you purchase a server close to your geographical location.
<br>
<br>
If all you are going to use this server for is node-red you will only need 1gb of ram and 1 cpu however I recommend 2gb of ram and 2 cpu's
<br>
<br>
MORE! ram and cpu if you plan on running Huge flows with multiple/constantly running injections
<br>
<br>
This Guide is <b>not</b> for individuals who fear the CLI (command line interface). If you have issues your going to need to read and search. You can always open an issue here https://github.com/meeki007/node-red-server-howto-guide/issues and if i have time I can try to help.
<br>

## Setup_Server
Bring up a terminal on the server
<br>

##### Creating_a_new_privileged_user_account
You should never log into your server as root. example ssh root@youripaddress
If you have not or no privileged user account exists lets create one.
<br>
If you have a privileged user account you can skip this step
<br>
<br>
In terminal:
```
$ adduser YourChosenNameHere
```
<br>

Give your new user account sudo rights by appending (-a) the sudo group (-G) to the user’s group membership:
<br>
<br>
In terminal:
```
$ usermod -a -G sudo YourChosenNameHere
```

<br>
Exit root account and login with the new privileged user account you have created
<br>
<br>

##### Update_Server
Update the local repositories and upgrade the operating system and installed applications by applying the latest patches.
<br>
Keep settings or files during the upgrade if asked. Follow the suggestions presented in terminal
<br>
<br>
In terminal:
```
$ sudo apt update && sudo apt upgrade -y
```
<br>
<br>

##### Enable_a_firewall
If you have a firewall setup you can skip this step.
<br>
<br>
Install a firewall, enable it, and configure it only to allow network traffic that you designate
<br>
<br>
In terminal:
```
$ sudo apt install ufw
```
<br>

UFW (universal firewall) denies all incoming connections and allows all outgoing connections
<br>
Enable access to SSH, HTTP, and HTTPS
<br>
<br>
In terminal:
```
$ sudo ufw allow ssh
```
```
$ sudo ufw allow http
```
```
$ sudo ufw allow https
```
<br>

Now lets start UFW
<br>
<br>
In terminal:
```
$ sudo ufw enable
```

<br>

You can see what services are allowed and denied with:
<br>
<br>
In terminal:
```
$ sudo ufw status
```
<br>

If you ever want to disable UFW, you can do so by typing:
<br>
<br>
In terminal:
```
$ sudo ufw disable
```
<br>
<br>

##### Install_Fail2ban
Fail2ban is an application that examines server logs looking for repeated or automated attacks. If any are found, it will alter the firewall to block the attacker’s IP address either permanently or for a specified amount of time.
<br>
<br>
In terminal:
```
$ sudo apt install fail2ban -y
```
<br>

Copy the included configuration file
<br>
<br>
In terminal:

```
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
<br>

And restart Fail2ban
<br>
<br>
In terminal:
```
$ sudo service fail2ban restart
```
<br>
<br>

## Node-red
Install follows the guide @ https://nodered.org/docs/getting-started/raspberrypi
<br>
Using the install script method works for Ubuntu
<br>
##### build-essential
Install possible dependency build-essential first
<br>
<br>
In terminal:

```
$ sudo apt install build-essential git
```
<br>
<br>

##### nod-red_script
Install nod-red and its dependencies
<br>
<br>
In terminal:

```
$ bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```
<br>
<br>

##### Autostart_on_boot
<br>
We want node-red to run automatically every time the server restarts
<br>
<br>
In terminal:
```
$ sudo systemctl enable nodered.service
```
<br>
<br>


## Hosting_to_the_world
We need to setup our DNS for our domain. Install Nginx to reverse proxy so users don't have to enter a port to get to the server. Install Lets encrypt certbot to issue ssl certificates and automate the process so you never have to manually renew ssl certificates; also so users can access the site using https for security and so browsers will not complain and block access. Configure fail2ban so random people on the internet can't mess with your server.

##### Install_Nginx
Nginx is a web server which can also be used as a reverse proxy.
<br>
By default, Nginx is configured to start automatically when the server boots/reboots

In terminal:
```
$ sudo apt install nginx
```
Check to see if Nginx is running

In terminal:
```
$ systemctl status nginx
```
Check to see if Nginx is hosting properly.
<br>
In a web browser type your servers ip address with no port. IE http://server_ip
<br>
You should see a welcome to nginx page


##### DNS
You need to tie your server ipv4 and if you have one ipv6 addresses to you DNS records.
I like to use subdomains for my reverse proxy needs in Nginx.
<br>
<br>
Log into your domain name hosting provider of choice and add some records.
<br>
<br>
DNS advice is outside the scope of this tutorial. You will need to learn and or ask your provider for help if you do not understand the steps below.
<br>
<br>
Create an A record for your ipv4 server address.
<br>
Create an AAAA record for your ipv6 server address.
<br>
Now Create a CNAME record pointing a subdomain to your domain name. IE your_domain_name.com
<br>
<br>
It should look somting like this:
<br>

| Hostname          | Type           | Address / Value      |
| :---------------: |:--------------:|:--------------------:|
|                   | A              | server_ipv4_address  |
|                   | AAAA           | server_ipv6_address  |
| www               | CNAME          | your_domain_name.com |
| node-red        | CNAME          | your_domain_name.com |

The goal of all this is to create the address node-red.your_domain_name.com for users to access
<br>
You can pick a subdomain name of your choice.

Testing that your domain works with Nginx. Enter your domain name. http://your_domain_name.com

You should see a welcome to nginx page

If not DO NOT PASS GO. Fix your domain issues.

##### Reverse_proxy
This is used so when users goto node-red.your_domain_name.com they are directed to the node-red server running on port :1880

First create a server block file for the url , making sure to change:
<br>
<b>server_name node-red.your_domain_name.com;</b>
<br>
To the subdomain.domain.com that you own

In terminal:
```
$ sudo nano /etc/nginx/sites-available/node-red.your_domain_name.com
```
This will open a text editor

Paste the following into the editor, making sure to change one line:
<br>
<b>server_name node-red.your_domain_name.com;</b>
<br>
To the subdomain.domain.com that you own
```
#proxy for node-red @ port :30000
server {
    	listen 80;
	listen [::]:80;

    	server_name node-red.your_domain_name.com;

    	location = /robots.txt {
    		add_header  Content-Type  text/plain;
    		return 200 "User-agent: *\nDisallow: /\n";
    	}
    	location / {
       		proxy_pass http://127.0.0.1:1880;

		#Defines the HTTP protocol version for proxying
		#by default it it set to 1.0.
		#For Websockets and keepalive connections you need to use the version 1.1    
		proxy_http_version  1.1;

		#Sets conditions under which the response will not be taken from a cache.
		proxy_cache_bypass  $http_upgrade;

		#These header fields are required if your application is using Websockets
		proxy_set_header Upgrade $http_upgrade;

		#These header fields are required if your application is using Websockets    
		proxy_set_header Connection "upgrade";

		#The $host variable in the following order of precedence contains:
		#hostname from the request line, or hostname from the Host request header field
		#or the server name matching a request.    
		proxy_set_header Host $host;

		#Forwards the real visitor remote IP address to the proxied server
		proxy_set_header X-Real-IP $remote_addr;

		#A list containing the IP addresses of every server the client has been proxied through    
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

		#When used inside an HTTPS server block, each HTTP response from the proxied server is rewritten to HTTPS.    
		proxy_set_header X-Forwarded-Proto $scheme;

		#Defines the original host requested by the client.    
		proxy_set_header X-Forwarded-Host $host;

		#Defines the original port requested by the client.    
		proxy_set_header X-Forwarded-Port $server_port;

    	}
}
```

Now enable the file by creating a link from it to the sites-enabled directory, which Nginx reads from during startup:
<br>
making sure to change:
<br>
<b>server_name node-red.your_domain_name.com;</b>
<br>
To the subdomain.domain.com that you own

In terminal:
```
$ sudo ln -s /etc/nginx/sites-available/node-red.your_domain_name.com /etc/nginx/sites-enabled/
```


To avoid a possible hash bucket memory problem that can arise from adding additional server names, it is necessary to adjust a single value in the /etc/nginx/nginx.conf file. Open the file:

In terminal:
```
$ sudo nano /etc/nginx/nginx.conf
```
Now uncomment (remove the # symbol) from the http section line <b>server_names_hash_bucket_size 64;</b> so it looks like this:

```
http {
    ...ignore stuff here...
    server_names_hash_bucket_size 64;
    ...ignore stuff here...
}
```

Next, test to make sure that there are no syntax errors in any of your Nginx files:

In terminal:
```
$ sudo nginx -t
```

Restart Nginx to enable your changes:

In terminal:
```
$ sudo systemctl restart nginx
```


<b>Later</b> if we ever need to disable a server block file simply remove the symbolic link.

In terminal:
```
$ sudo rm /etc/nginx/sites-enabled/node-red.your_domain_name.com
```
And don't forget to restart to effect changes

In terminal:
```
$ sudo systemctl restart nginx
```

Test if the reverse proxy is working
<br>
Goto your website, non https, and make sure it works
<br>
http://node-red.your_domain_name.com

The node-red page should load. I know its tempting but do not configure it now. exit the page / close the browser.

##### HTTPS_SSL_certbot
Install certbot to get ssl certificates for domain
<br>
at this time cerbot not relesed for ppa package use snap

In terminal:
```
$ sudo snap install --beta certbot --classic
```
<br>
<br>
Generate the certificate for domain
<br>
making sure to change:
<br>
<b>server_name node-red.your_domain_name.com;</b>
<br>
To the subdomain.domain.com that you own

In terminal:
```
$ sudo certbot --nginx -d node-red.your_domain_name.com
```
If asked to redirect HTTP traffic to HTTPS, removing HTTP access:
<br>
Select 2: Redirect - Make all requests redirect to secure HTTPS access.

Test if https is working
<br>
Goto your website, https, and make sure it works
<br>
https://node-red.your_domain_name.com

The node-red page should load. I know its tempting but do not configure it now. exit the page / close the browser.

Test that lets encrypt cerbot is working and will automatically update certificates before they expire

In terminal:
```
$ sudo certbot renew --dry-run
```
<br>
<br>


## Security

Setup node-reds built in login a password for the admin area, also create a fail2ban filter for 4xx failed requests.

##### adminAuth_user_password

First lets hash a password so we can add it to the settings.js file.

My method of choice for this is to use bcryptjs
<br>
Add to your node-red manage pallet node-red-contrib-bcrypt
<br>
<br>
Now import this flow into node-red
```
[{"id":"6fb8f86c.3dd218","type":"comment","z":"93ccda7b.0f6118","name":"create a key for a password","info":"","x":220,"y":120,"wires":[]},{"id":"2c990cc6.7e76e4","type":"bcrypt","z":"93ccda7b.0f6118","name":"","action":"encrypt","field":"payload","hash":"payload","rounds":"10","x":570,"y":160,"wires":[["480852c7.37515c"]]},{"id":"e203ec3d.eb8938","type":"inject","z":"93ccda7b.0f6118","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":190,"y":160,"wires":[["15919cce.3e10a3"]]},{"id":"15919cce.3e10a3","type":"change","z":"93ccda7b.0f6118","name":"Enter Password Here","rules":[{"t":"set","p":"payload","pt":"msg","to":"MyPassword","tot":"str"}],"action":"","property":"","from":"","to":"","reg":false,"x":380,"y":160,"wires":[["2c990cc6.7e76e4"]]},{"id":"480852c7.37515c","type":"debug","z":"93ccda7b.0f6118","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","x":730,"y":160,"wires":[]}]
```
Replace MyPassword with your password!!!
<br>
Trigger the flow and check the msg.payload
<br>
copy the payload as it is your password hash you will need next
<br>
<br>
We need to setup a user name and password by editing node-red's settings.js file.
<br>
Edit the settings.js file in our current privileged_user_account /home/YourChosenNameHere/.node-red/settings.js
<br>
We can use $HOME variable to accomplish this
<br>
<br>
In terminal:
```
$ nano $HOME/.node-red/settings.js
```
This will open a text editor

Edit the adminAuth section removing forward slashes // so it looks like this:
<br>

```
adminAuth: {
    type: "credentials",
    users: [
        {
            username: "admin",
            password: "$2a$08$zZWtXTja0fB1pzD4sHCMyOCMYz2Z6dNbM6tl8sJogENOMcxWV9DN.",
            permissions: "*"
        }
    ]
}
```
<br>
<br>
Replace $2a$08$zZWtXTja0fB1pzD4sHCMyOCMYz2Z6dNbM6tl8sJogENOMcxWV9DN. within QUOTES with your password hash you created.
<br>
<br>

More information on this can be found at: https://nodered.org/docs/user-guide/runtime/securing-node-red
<br>
<br>

##### fail2ban_403_protection

Every time a user....hacker...etc tries to login to node-red with a bad user name or password it sends a 403 Forbidden error
<br>
This response code indicates that the server understood the request but refuses to authorize it
<br>
<br>
What we would like todo is ban users who have multiple failed attempts to login
<br>
That way we can keep the bad people from password spamming the server trying to gain access.
<br>
<br>
Lets block 404, 444, 403, 400 responses from the server. Feel free to read up on these codes but the gist of it is we will be blocking people trying to gain access to areas they should not be trying to get into.
<br>
<br>
<br>
Create a new fail2ban filter for 404, 444, 403, 400 responses from the server.

In terminal:
```
sudo nano /etc/fail2ban/filter.d/nginx-4xx.conf
```
This will open a text editor

Add the following to the file:

```
[Definition]
failregex = ^<HOST>.*"(GET|POST).*" (404|444|403|400) .*$
ignoreregex =
```
Now, edit your /etc/fail2ban/jail.conf, to use the new filter we created and ban after 10 failed attempts.

In terminal:
```
$ sudo nano /etc/fail2ban/jail.conf
```

This will open a text editor

Add the following to the bottom of the file:

```
[nginx-4xx]
enabled = true
port = http,https
logpath = %(nginx_access_log)s
maxretry = 10
```

Restart fail2ban

In terminal:
```
$ sudo service fail2ban restart
```

Check that the new rule is working

In terminal:
```
sudo fail2ban-client status nginx-4xx
```
Now the user will be banned for the amount of time you have set in fail2ban after 10 failed attempts

you can check this is working with the logs.

In terminal:
```
$ sudo cat /var/log/fail2ban.log | grep Ban
```
and to check in on what the nginx log is producing

In terminal:
```
$ sudo cat /var/log/nginx/access.log
```

## Extras

##### automatic_increasing_ban_times

Many malicious actors will just take note of the fixed ban time and just keep at it every 10min. Instead of increasing the fixed time we want to increase the time for specific bad actors and not every one.

In terminal:
```
$ sudo nano /etc/fail2ban/jail.conf
```

This will open a text editor

search for and remove the # sign in front of the following lines so they look like this:

```
bantime.increment = true

bantime.factor = 1

bantime.formula = ban.Time * (1<<(ban.Count if ban.Count<20 else 20)) * banFact>
```

Restart fail2ban

In terminal:
```
$ sudo service fail2ban restart
```




<b>END OF TUTORIAL</b>
