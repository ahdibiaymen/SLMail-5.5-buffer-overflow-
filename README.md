# SLMail 5.5 buffer overflow

Seattle Lab Mail 5.5 server is vulnerable to an unauthenticated buffer overflow attack when receiving an excessively long password. This vulnerability can be exploited in any version of Windows running the executable SLmail.exe.

I exploited this vulnerability in the Windows XP SP3 environment using Kali Linux version 2021, using an automated exploit 'Metasploit' and a manual step-by-step exploit with the 'Immunity Debugger'.

## TEST

* start the service POP3 SLMail in the victim machine.  
* execute the python script '**slmail.py**' using the following syntaxe :
NB: @machine_victime needs to be replaced by the real victim machine address 

```bash
./slmail.py @machine_victime
```

* Don't forget to grant execution privilege to the script   
```bash
chmod +x slmail.py
```

## Exploit Details (FR)
 
For a comprehensive, step-by-step explanation of the exploit, refer to the file **Rapport exploit SLMail v5.5.0.DOCX** for thorough exploit details. 
