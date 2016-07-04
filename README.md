# Fail2Ban Blacklist JAIL for Repeat Offenders
### with Perma / Extended Banning Across Reboots

>A customised jail with action and filter file for Fail2Ban. 
This jail is based on the recidive jail but makes use of a simple 
text file to enable extended and permanent bans even across reboots.

>This is intended to replace the recidive filter so make sure that
recidive is set to enabled = false do not have both this jail and
recidive running at the same time

#####Author: Mitchell Krog <mitchellkrog@gmail.com>
#####Version: 1.0
######GitHub: https://github.com/mitchellkrogza/Fail2Ban-Blacklist-JAIL-for-Repeat-Offenders-with-Perma-Extended-Banning
######Blog: https://ubuntu101.co.za/
######Fail2Ban: http://www.fail2ban.org/wiki/index.php/Main_Page

####Tested On: 
				Fail2Ban 0.91
####Server: 
				Ubuntu 16.04
####Firewall: 
				IPTables

###Dependancies: 
				requires blacklist.conf in /etc/fail2ban/filter.d folder
				requires blacklist.conf in /etc/fail2ban/action.d folder
				requires jail settings called [blacklist]
				requires ip.blacklist file in /etc/fail2ban
				create with sudo touch /etc/fail2ban/ip.blacklist
				recidive filter must be disabled (do not run both at same time)

###Drawbacks: 
 				Only works with IPTables
 
###Based on: 
 				the Recidive Jail from Fail2Ban (do not run both at same time please)
 				
### How it works / Concepts:
		This jail monitors all your Fail2Ban log files including any rotated
		log files because the log file location setting in the jail is wild-carded
		
		It requires an action.d file called blacklist.conf in your /etc/fail2ban/action.d folder
		It requires an filter.d file called blacklist.conf in your /etc/fail2ban/filter.d folder
		It requires the jail [blacklist] settings in your jail.local file
		
		In my jail settings I have set a
						findtime of 1 year (31536000 seconds)
						bantime of 1 year (31536000 seconds)
						maxretry of 10 attempts
						
		This means Fail2Ban will scan through it's log files over a full year's period.
		If it finds the same attack pattern, let's say an SSH attack for instance, from the
		same IP address on 10 different occasions anywhere within 1 year, that IP address 
		is then regarded as a repeat offender and can very well be blacklisted for the 1 year 
		period or even forever.
		
		This is done through a very simple text based file called ip.blacklist
		This file needs to be created in your /etc/fail2ban folder as follows
		sudo touch /etc/fail2ban/ip.blacklist

### The Startup Action:		
		The startup action checks the existing ip.blacklist file for any duplicates and
		automatically removes them. It also sorts the file into numbered order which makes
		looking through the file later a breeze. The startup action then adds all IP's contained 
		in the blacklist file into your IPTables with a DROP command. This happens every time 
		Fail2Ban starts or even after a server reboot. This means this truly works across reboots 
		unlike other repeat offender jails out there. 
		
		It also effectively deals with any chance of duplicates.
		A really simple sort commandline used to sort and clear the file of dupes.

### The Ban Action:		
		The ban action takes a new IP address which was found to match our rules and writes this
		new entry into the ip.blacklist file and it then adds this new IP to the IPTables rules and the
		new repeat offender is immediately blocked with a DROP command.
		
### The UnBan Action:
		The unban action removes the IP address from the ip.blacklist file and deletes the 
		IPTables firewall entry. If the same offending IP address comes back and tries an attack
		again even just once, he will probably satisfy the 1 year rule again and will be blocked 
		again for another entire year. A really simple sed commandline used to delete the IP entry 
		from the ip.blocklist file.
		
#### Other Comments:
		Some may think this is harsh but if someone really tries 10 times they must be banned
		it's as simple as that. 
		
		If a rogue IP address really has not been dealt with by the network manager of the company 
		owning the IP address (in an entire year), then it's unlikely they will ever deal with it 
		or simply are being hacked to death themselves and don't even know how to stop it. 

		Then it is time to even consider using -1 as your bantime so these BAD IP's are blocked forever.
		
		I based this on the recidive filter which comes with Fail2Ban but I found this a better
		method at making sure bans are persistent across reboots and it's fool proof. It's also very
		fast and does not slow down Fail2Ban whatsoever.
		
		It is suggested to also modify your Fail2Ban log rotation settings to have logrotate
		retain Fail2Ban logs for at least 13 months. (see below for logrotate settings for Fail2Ban)
		
		It has only been tested on the 0.91 version of Fail2Ban on Ubuntu 16.04 but it should work perfectly
		for any previous versions too but there is no guarantee of this until I can test myself.
				
		If you are new to Fail2Ban go read my tutorial at
		https://ubuntu101.co.za/security/fail2ban/fail2ban-persistent-bans-ubuntu/
		
#### LogRotate Settings for Fail2Ban:

		edit this file at /etc/logrotate.d/fail2ban

		This is set to rotate the log file monthly and delete any log files older than
		13 months, assuring you, you always have a full 1 year of log's to reference for
		Repeat Offenders

			/var/log/fail2ban.log {
    			monthly
    			rotate 13
    			compress
				delaycompress
    			missingok
    			notifempty
    			postrotate
				fail2ban-client flushlogs 1>/dev/null
    			endscript
    			create 640 root adm
				}
		
#### Some Good Advice For You:
		In my time working with Fail2Ban I have had to rely on many forums for help and guidance
		with problems I ran into. Almost every time I found out my problems were all merely syntax
		related problems in my jail.local file so ALWAYS make sure your syntax is correct by starting
		the fail2ban client as follows after you have made ANY modifications to your jail.local file.
		sudo fail2ban-client -vvv -x start
		This will give you a verbose output for debugging purposes. 
		
		Finally and please pay attention to this. I have seen a lot of people on forums who have 
		had problems getting Fail2Ban to work properly receiving advice from strangers telling them
		to do silly things like disabling Ubuntu's SELinux / AppArmor module. This really is bad advice
		because I can assure you Fail2Ban works 100% perfectly with Apparmor / SELinux in it's default
		unmodified state. 
		
		Don't place yourself in a situation of going through the effort of installing Fail2Ban for
		added security measures while at the same time disabling other security measures. 

#### A Personal Comment on Country Blocking:
		Be careful of following advice of blocking entire country IP blocks. 
		It's just in my opinion a really bad network practice to block an entire country simply because 
		one or two networks are badly managed. 
		
		You may be hosting web sites for clients who are losing potential business from other countries
		simply because you have set rules to prevent that entire country from even seeing their web site
		or reaching your server.
		
		Fail2Ban and this custom Jail will work perfectly for you at dealing with individual repeat offender
		IP addresses and dealing with them permanently. 
		
		If you really must block an entire country, make sure you are 100% aware of the implications.

		If someone really wants to hack your servers though, they will just jump to another country
		.... and another ..... and another .... and another ..... are you going to block the entire world 
		eventually?
				
## Disclaimer:
		This software comes with no warranty of any sort and you use this at your own risk.
		The author will not be held responsible for any failures through the use of this software
		add on for the popular Fail2Ban plugin.
		
		This plugin / custom jail for Fail2Ban is also NOT official, it is customised by myself
		for my own server environment and I have made it available on Github as open source 
		software. 
		
		While this software has been thoroughly tested on the server environment and software
		versions listed in this readme file, the author can not offer any guarantee that it will 
		work on your server.
		
		The most common reason should this not work for you is that your file permissions have
		been fiddled with or your server has been modified in other non-standard ways.
		Fail2Ban requires root access to all it's files and folders.

## Free to Use - Free to Change:
		This is open source software and 100% free to use. 
		You can modify it to your liking if you don't like the way I have done something, 
		but if you break it you fix it yourself. This workign and tested version is truly all
		you should ever need.
		
## Issues:
		Feel free to log any issues using the issue logging system here on GitHub. I will do my
		best to help you if I can find any free time to do so.
		
### 	Thanks to all the really good folks out there who contribute to Fail2Ban and who write add ons and modules for it.
