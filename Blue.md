
Tags: #writeup #ActiveDirectory #smb  #EternalBlue #Metasploit  #RemoteAccessRCE #EscaladaPrivilegios 


## WRITE UP
Ejemplo: 10.10.10.40 (víctima) TTL= 127 LINUX

### INTRODUCCIÓN

Aprenderemos sobre que es y como explotar la vulnerabilidad **Eternal Blue** para ganar una RCE privilegiada sin tener credenciales.


#### RECONOCIMIENTO

Lo primero que vamos hacer es crear nuestro entorno de trabajo: <span style="color:lightblue">blue</span>

Dentro usaremos la herramienta de s4vitar [[mkt]] cual nos generara las carpetas necesarias para tener todo más organizado; **Nmap...**

>**which** mkt.py | **xargs** **batcat** -l python

*Ver en que ubicación está una herramienta y ver su código*
*Adelante explicamos a tener batcat, que es un cat pero con esteroides*

Para empezar el reconocimiento, enviamos una traza ICMP a la <span style="color:red">IP</span> de la maquina víctima, para comprobar que tenemos conectividad, tenemos dos alternativas:

-Usar el script [[whichSystem]] que nos dirá directamente el equipo al que estamos atacando, por ende habrá tenido ping para dar la respuesta. Es más silencioso que nmap.

![[Pasted image 20251001182437.png]]


-Usar el comando [[ping]] <span style="color:red">10.10.10.40</span> <span style="color:yellow">-c</span> 1  <span style="color:yellow">-R</span>
*-R Lo que hace es un record route que consiste que a la hora de hacer la petición se lo envía a un nodo intermediario para que no sea directa la petición, al ser windows no muestra el prceso* 

![[Pasted image 20251001182444.png]]



Después de confirmar que tenemos conectividad, usaremos **nmap** para a ver que puertos tenemos abiertos y que protocolos/servicios tenemos.

**nmap** <span style="color:yellow">-p- --open -Pn -vvv -n -sS --min-rate 5000</span> <span style="color:red">10.10.10.40</span> <span style="color:yellow">-oG</span> <span style="color:pink">allPorts</span>
*Veremos porque el formato grapeable, es importante.*

![[Pasted image 20251001182426.png]]


Una vez hecho, usaremos otra herramienta de s4vitar [[extractports]] al archivo allPorts
cual nos copiara los puertos, y escanearlos con nmap. 

![[Pasted image 20251001182458.png]]


**nmap** <span style="color:yellow">-sVC</span>  <span style="color:yellow">-p</span><span style="color:purple">135,139,445,49152,49153,49154,49155,49156,49157</span> <span style="color:red">10.10.10.40</span> <span style="color:yellow">-oN</span> <span style="color:pink">targeted</span> 
*Este formato lo emplearemos con batcat lenguaje java para verlo mejor*
*Nos mostrará la versión de los servicios que están corriendo*
*Usará scripts defaults*

![[Pasted image 20251001182546.png]]


Una vez escaneado, usaremos batcat:
>https://github.com/sharkdp/bat.git

**batcat** <span style="color:pink">target</span> <span style="color:yellow">-l</span> <span style="color:orange">java</span>
*nos mostrara la salida en un formato más bonito, con java

![[Pasted image 20251001182609.png]]


Puerto <span style="color:purple">135</span> que corre el servicio <span style="color:orange">RPC</span>

Puerto <span style="color:purple">445</span> que corre el servicio <span style="color:orange">SMB</span>
*Versión Windows 7 Professional*


###### VULNERABILIDAD SMB: ETERNAL BLUE

Al estar en  un Windows antiguo (En este caso Windows 7 Professional) podemos usar el script de **nmap**  "vuln and safe" para comprobar si es vulnerable a [[EternalBlue]].

**nmap**  <span style="color:yellow">--script</span>  <span style="color:orange">"vuln and safe"</span> <span style="color:green">-vvv</span>  <span style="color:yellow">-p</span><span style="color:purple">445</span>  <span style="color:red">10.10.10.40</span>   <span style="color:yellow">-oN</span> <span style="color:pink">bluevuln</span> 

![[Pasted image 20251001182857.png]]


En el escaneo nos saldrá **VULNERABLE** que nos informa de una vulnerabilidad de smb que nos permite **RCE**. -> <span style="color:gold">CVE-2017-0143</span> 

![[Pasted image 20251001182908.png]]



#### EXPLORACIÓN SMB

Podríamos intentar el ataque [[ENUM USERS RPC TO BRUTE FORCE SHARES]] pero no va a ver éxito. En los shares no va a ver nada importante.

![[Pasted image 20251001190655.png]]

![[Pasted image 20251001190602.png]]


#### CONFIGURACIÓN EXPLOIT ETERNALBLUE

Usaremos [[Metasploit]] para configurar un exploit y conseguir una Shell sin necesitar credenciales de ningún usuario.

**search** <span style="color:lightblue">eternal blue</span>

![[Pasted image 20251001190809.png]]


Usaremos el primer exploit, por ejemplo.

**set** 0

![[Pasted image 20251001191115.png]]


Veremos que valores tenemos que configurar:

**show** options

![[Pasted image 20251001191217.png]]


Tendremos que agregar la IP del host víctima y host atacante

**set** valor

![[Pasted image 20251001191435.png]]


Ahora ejecutamos el exploit.

**run** o **exploit**

![[Pasted image 20251001191608.png]]


Nos dará una Shell y estaremos con privilegios como  <span style="color:blue">NT AUTHORITY\SYSTEM</span>

![[Pasted image 20251001191632.png]]


#### BANDERA USER

Nos vamos al usuario <span style="color:blue">haris</span> y cogemos su bandera en el escritorio.

![[Pasted image 20251001191723.png]]



#### BANDERA ROOT

Nos vamos al usuario <span style="color:blue">Administrator</span> y cogemos su bandera en el escritorio.

![[Pasted image 20251001191944.png]]