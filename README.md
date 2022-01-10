# SLMail 5.5 buffer overflow

Le serveur POP3 du Seattle Lab Mail 5.5 souffre d'une vulnérabilité de dépassement de tampon non authentifié lors de la réception d'un mot de passe très long. on peut  exploiter cette vulnérabilité dans n'importe quelle version de Windows exécutant l'exécutable SLmail.exe.

j'ai exploité cette vulnérabilité dans l'environnement **Windows Xp Sp3** par **KALI linux** version 2021 tout en utilisant un exploit automatisé '**Metasploit**' et un exploit Manuel pas à pas avec le débugger '**Immunity debugger**' 

## Test d'exploit

* lancer le service POP3 SLMail dans la machine victime 
* exécuter le script python '**slmail.py**' par la syntaxe suivante :
NB: @machine_victime doit être remplacé par l'adresse réel de la machine victime 

```bash
./slmail.py @machine_victime```

## Vous devez donner les droits d'execution au script 
## chmod +x slmail.py

```

## Détail de l'exploit 
vous trouverez tout détail d'exploit bien expliqué étape par étape dans le fichier 

'**Rapport exploit SLMail v5.5.0.DOCX**' 
