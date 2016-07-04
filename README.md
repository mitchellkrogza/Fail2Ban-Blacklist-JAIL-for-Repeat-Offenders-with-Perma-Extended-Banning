# Fail2Ban Blacklist JAIL for Repeat Offenders
## with Perma / Extended Banning

A customised jail with action and filter file for Fail2Ban. 
This jail is based on the recidive jail but makes use of a simple 
text file to enable extended and permanent bans.

##Author: Mitchell Krog <mitchellkrog@gmail.com>
##Version: 1.0
######GitHub: https://github.com/mitchellkrogza/Fail2Ban-Blacklist-JAIL-for-Repeat-Offenders-with-Perma-Extended-Banning

####Tested On: Fail2Ban 0.91
####Server: Ubuntu 16.04
####Firewall: IPTables

###Dependancies: requires blacklist.conf in /etc/fail2ban/filter.d folder
				requires blacklist.conf in /etc/fail2ban/action.d folder
				requires jail settings called [blacklist]
				requires ip.blacklist file in /etc/fail2ban
				create with sudo touch /etc/fail2ban/ip.blacklist

 ####Drawbacks: Only works with IPTables
 #####Based on: the Recidive Jail from Fail2Ban
