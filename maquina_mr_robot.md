# Walkthrough para Encontrar las Tres Claves de la Máquina

## Búsqueda de la Primera Clave

### 1. Escaneo de Puertos

Como primer paso, realizamos un escaneo de puertos a la máquina usando la herramienta [nmap](https://nmap.org/). Observamos que la máquina tiene abiertos los siguientes puertos:

- **80** (HTTP)
- **443** (SSL)

![Escaneo de Puertos](images/robot.png)

Al acceder al puerto **80**, encontramos una página web que simula una especie de terminal. Como pista para encontrar la primera clave, nos sugieren "robots". Esto nos lleva a explorar el archivo `robots.txt`.

![Página Web](images/robots2.png)

### 2. Exploración del Archivo robots.txt

En el archivo `robots.txt`, encontramos dos entradas:

- **key-1-of-3.txt**
- **fsociety.dic** (un diccionario)

Esto sugiere que podríamos necesitar realizar un ataque de fuerza bruta más adelante.

![Contenido de robots.txt](images/robot3.png)

Al abrir el archivo `fsociety.dic`, confirmamos que es un diccionario.

![Diccionario fsociety](images/robots4.png)

Al descargar el archivo `key-1-of-3.txt`, encontramos la **primera clave**.

![Primera Clave](images/robots5.png)

---

## Búsqueda de la Segunda Clave

### 1. Enumeración de Directorios

Regresamos a la página web y realizamos una enumeración de directorios utilizando la herramienta [Gobuster](https://github.com/OJ/gobuster).

![Enumeración de Directorios](images/robots6.png)

Entre los directorios encontrados, identificamos:

- Una página de inicio de sesión (`login`)
- Un archivo de texto codificado en Base64

### 2. Decodificación de Base64

Descargamos el archivo de texto y al decodificar su contenido en Base64, obtenemos un usuario y una contraseña que podrían ser para WordPress.

![Decodificación Base64](images/robots7.png)

![Credenciales Obtenidas](images/robots8.png)

### 3. Acceso a WordPress

Utilizamos las credenciales para acceder al panel de WordPress y comenzamos a buscar una forma de subir una **reverse shell**.

![Panel de WordPress](images/robots9.png)

### acceso por medio de fuerza bruta 

### 4. Investigación Adicional

Antes de continuar, investigamos más información sobre la máquina. Descubrimos que está relacionada con una serie de televisión donde el protagonista es un hacker llamado *Elliot*. Esto sugiere que el nombre de usuario podría ser "Elliot".

Usamos [Burp Suite](https://portswigger.net/burp) para interceptar y analizar las cookies, donde confirmamos el nombre de usuario "Elliot".

![Interceptando Cookies](images/robots11.png)

### 5. Fuerza Bruta con Hydra

Utilizamos [Hydra](https://github.com/vanhauser-thc/thc-hydra) para realizar un ataque de fuerza bruta con el usuario "Elliot" y el diccionario `fsociety.dic`. Finalmente, obtenemos la contraseña.

![Ataque de Fuerza Bruta](images/robot12.png)

---

## Subida de una Reverse Shell

### 1. Modificación de Temas en WordPress

Dentro del panel de WordPress, editamos un archivo `.php` de uno de los temas activos y subimos nuestra **reverse shell**.

![Modificación de Tema](images/robots13.png)

Modificamos la IP en el código de la reverse shell para que apunte a nuestra máquina y activamos una escucha.

Accedemos al archivo subido en:


![Reverse Shell Activa](images/robots14.png)

Con esto, conseguimos acceso a la máquina y encontramos un archivo que contiene la **segunda clave**.

![Segunda Clave](images/robots15.png)

### 2. Cracking de Hash

Encontramos un hash en la máquina y lo crackeamos usando las herramientas:

- [John the Ripper](https://www.openwall.com/john/):
  

![John the Ripper](images/robots16.png)


- [Hashcat](https://hashcat.net/hashcat/):


![Hashcat](images/robots17.png)

Con la contraseña obtenida, abrimos el archivo `key.txt` y conseguimos la segunda clave.

---

## Búsqueda de la Tercera Clave

### Escalación de Privilegios

Como pista, sabemos que la tercera clave está relacionada con `nmap`. Enumeramos los permisos en la máquina y vemos que tenemos permisos para ejecutar `nmap` como superusuario. Esto nos permite explotar `nmap` para escalar privilegios.

![Permisos de nmap](images/robots19.png)

Una vez escalados los privilegios a **root**, navegamos al directorio `/root` y encontramos el archivo `key.txt` con la tercera y última clave.

---

Con esto, completamos la máquina y encontramos las tres claves. ¡Buen trabajo! 🎉



