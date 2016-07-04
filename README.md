# Fail2Ban Blacklist JAIL for Repeat Offenders
## with Perma / Extended Banning

>A customised jail with action and filter file for Fail2Ban. 
This jail is based on the recidive jail but makes use of a simple 
text file to enable extended and permanent bans.

#####Author: Mitchell Krog <mitchellkrog@gmail.com>
#####Version: 1.0
######GitHub: https://github.com/mitchellkrogza/Fail2Ban-Blacklist-JAIL-for-Repeat-Offenders-with-Perma-Extended-Banning
######Blog: https://ubuntu101.co.za/

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

###Drawbacks: 
 				Only works with IPTables
 
###Based on: 
 				the Recidive Jail from Fail2Ban
 				
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
						
		This means Fail2Ban will scan through it's log files over a full year's period
		If it finds the same attack pattern, let's say an SSH attack for instance, from the
		same IP address on 10 different occasions anywhere within 1 year, that IP address 
		is then regarded as a repeat offender and can very well be blacklisted for the 1 year 
		period or even forever.
		
		This is done through a very simple text based file called ip.blacklist
		This file needs to be created in your /etc/fail2ban folder as follows
		sudo touch /etc/fail2ban/ip.blacklist

### The Startup Action:		
		The startup action checks the existing ip.blacklist file for any duplicates and
		automatically removes them. The startup action then adds all IP's contained in the blacklist
		file into your IPTables with a DROP command. This happens every time Fail2Ban starts or
		after a server reboot. This means this truly works across reboots unlike other repeat 
		offender jails out there. It also effectively deals with any chance of duplicates.

### The Ban Action:		
		The ban action takes a new IP address which was found to match our rules and writes this
		new entry into the ip.blacklist file and it then adds this new IP to the IPTables rules and the
		new repeat offender is immediately blocked with a DROP command.
		
### The UnBan Action:
		The unban action removes the IP address from the ip.blacklist file and deletes the 
		IPTables firewall entry. If the same offending IP address comes back and tries an attack
		again even just once, he will probably satisfy the 1 year rule again and will be blocked 
		again for another entire year.
		
#### Other Comments:
		Some may think this is harsh but if someone really tries 10 times they must be banned
		it's as simple as that.
		
		I based this on the recidive filter which comes with Fail2Ban but I found this a better
		method at making sure bans are persistent across reboots and it's fool proof.
		
		It is suggested to also modify your Fail2Ban log rotation settings to have logrotate
		retain Fail2Ban logs for at least 13 months.
		
		If you are new to Fail2Ban go read my tutorial at
		https://ubuntu101.co.za/security/fail2ban/fail2ban-persistent-bans-ubuntu/
		
		
