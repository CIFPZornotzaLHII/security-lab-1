# 🐳 Laboratorio de Docker

![alt text](image.png)

- [🐳 Laboratorio de Docker](#-laboratorio-de-docker)
- [🛡️ Instalación de la máquina atacante (Kali Linux)](#️-instalación-de-la-máquina-atacante-kali-linux)
  - [⚙️ Instalar Kali Linux](#️-instalar-kali-linux)
  - [🔓 Acceder a la máquina](#-acceder-a-la-máquina)
  - [🛠️ Instalación de herramientas](#️-instalación-de-herramientas)
- [🎯 Agregando una máquina vulnerable (DVWA)](#-agregando-una-máquina-vulnerable-dvwa)
- [🔍 Escaneando la Red con Nmap](#-escaneando-la-red-con-nmap)
  - [📡 Escanear la subred completa](#-escanear-la-subred-completa)
    - [❓ Pregunta 1:](#-pregunta-1)
  - [📊 Extraer información de los hosts encontrados](#-extraer-información-de-los-hosts-encontrados)
    - [❓ Pregunta 2:](#-pregunta-2)
    - [❓ Pregunta 3:](#-pregunta-3)
  - [🔐 Usando Scripts de NMAP](#-usando-scripts-de-nmap)
    - [❓ Pregunta 4:](#-pregunta-4)
    - [❓ Pregunta 5:](#-pregunta-5)

---
# 🛡️ Instalación de la máquina atacante (Kali Linux)

## ⚙️ Instalar Kali Linux

Para instalar Kali Linux usando Docker, vamos a crear un archivo `docker-compose.yml` con el siguiente contenido:

```yml

services:

  kali:
    image: kalilinux/kali-rolling
    container_name: kali
    restart: unless-stopped
    tty: true
    network_mode: "host" # Esto anula cualquier otra red de Docker comentar
    stdin_open: true
    volumes:
      - ./home:/home
    cap_add:
      - NET_ADMIN
      - NET_RAW

```

Para agregar la máquina Kali a otra red de Docker, habría que comentar `network_mode: "host"` y agregarla a otra red.

## 🔓 Acceder a la máquina

```bash
docker exec -it kali bash
```

Actualizamos el gestor de paquetes y el sistema operativo:

```bash
apt update && apt upgrade -y
```

## 🛠️ Instalación de herramientas

Dado que la imagen de Kali Linux es muy ligera, no incluye muchas herramientas preinstaladas. Para instalar las herramientas necesarias para el laboratorio, ejecutamos el siguiente comando:

```bash
apt install -y sudo nmap net-tools locate iproute2
```

Una vez instaladas las herramientas, podemos crear una imagen personalizada de Kali Linux con las herramientas necesarias para el laboratorio, esto aumentará el tamaño de la imagen, pero nos permitirá tener una máquina lista para usar en futuras ocasiones sin necesidad de instalar las herramientas cada vez. Para crear la imagen personalizada, ejecutamos el siguiente comando:

```bash
docker commit kali kali-custom
```

Ahora, cada vez que queramos usar la máquina Kali con las herramientas instaladas, podemos usar la imagen `kali-custom` en lugar de la imagen original `kalilinux/kali-rolling`. Para hacerlo, simplemente cambiamos la línea de la imagen en el archivo `docker-compose.yml`.

---

# 🎯 Agregando una máquina vulnerable (DVWA)

Vamos a agregar una máquina vulnerable llamada DVWA (Damn Vulnerable Web Application) para practicar nuestras habilidades de escaneo de red y análisis de vulnerabilidades. Para hacerlo, agregamos el siguiente servicio a nuestro archivo `docker-compose.yml`:

```yml
    dvwa:
        image: vulnerables/web-dvwa
        container_name: dvwa
        restart: unless-stopped
        ports:
        - "8081:80"
```

Ahora creamos una nueva red de Docker para conectar ambas máquinas, agregamos la siguiente sección al final del archivo `docker-compose.yml`:

```yml

networks:
  lab-network:
    driver: bridge
```

y luego conectamos ambas máquinas a esta red agregando la siguiente línea en cada servicio:

```yml
    networks:
      - lab-network
```

Ya estamos listos para hacer nuestro primer escaneo de red.


# 🔍 Escaneando la Red con Nmap

Nmap es una herramienta de escaneo de red que nos permite descubrir hosts y servicios en una red.

Esta herramienta incluye una gran cantidad de opciones para personalizar el escaneo. 

Echa un vistazo a la [documentación oficial de Nmap](https://nmap.org) para aprender más sobre sus capacidades y cómo usarlo de manera efectiva. O al cheatsheet que puedes encontrar en este repositorio [Cheat Sheet](./nmap-cheatsheet.md).

## 📡 Escanear la subred completa

Antes de empezar, ejecutamos el siguiente comando para descubrir la subred a la que estamos conectados:

```bash
ip a
```

Otra alternativa es usar `docker network inspect nombre_de_la_red` para obtener información sobre la red de Docker y los contenedores conectados a ella pero en este caso queremos descubrir los equipos usando nmap.


*Escanear una subred completa es una tarea que nos permite descubrir todos los hosts activos en una red. Hay que tener cuidado al realizar este tipo de escaneo, ya que puede generar una gran cantidad de tráfico en la red y puede ser detectado por los sistemas de seguridad además de que pueden demorar mucho tiempo en completarse.*


```bash
nmap --top-ports 100 -T5 --open Ip_de_la_subred/mascara_de_la_subred -v -n
```
- **--top-ports 100**: Escanea los 100 puertos más comunes
- **-T5**: Usa el modo de escaneo más rápido (agresivo)
- **--open**: Escanea solo los puertos abiertos
- **-n**: No resuelve nombres de host (no usa DNS)
- **-v**: Modo verboso, muestra información detallada durante el escaneo
- **Ip_de_la_subred/mascara_de_la_subred**: Especifica la subred a escanear, por ejemplo 192.168.1.0/24

Agrega aquí la información que consideres relevante sobre el escaneo de la subred completa.

```bash

```

### ❓ Pregunta 1:

Cuántos hosts activos has encontrado en la red?

## 📊 Extraer información de los hosts encontrados

Ahora que hemos encontrado un host activo, podemos usar Nmap para extraer información sobre los servicios que están corriendo en ese host, los puertos abiertos, el sistema operativo, etc., de una manera más exhaustiva que no podemos obtener con un simple ping.

```bash
nmap -p- -sV Ip_del_host -v
```
- **-p-**: Escanea todos los puertos (1-65535)
- **-sV**: Detecta la versión de los servicios que están corriendo en los puertos abiertos
- **Ip_del_host**: Especifica el host a escanear

### ❓ Pregunta 2:

Qué servicios has encontrado corriendo en el host? Qué versión de esos servicios has encontrado? Has podido identificar el sistema operativo del host?

### ❓ Pregunta 3:

Realiza una búsqueda en internet para encontrar vulnerabilidades conocidas de los servicios y versiones que has encontrado. 

Puedes usar sitios como [CVE Details](https://www.cvedetails.com/) o [Exploit Database](https://www.exploit-db.com/) para buscar vulnerabilidades.

## 🔐 Usando Scripts de NMAP

Además de las opciones básicas, Nmap también incluye una gran cantidad de scripts que nos permiten realizar escaneos más avanzados y específicos para detectar vulnerabilidades, obtener información adicional sobre los servicios, etc. [Descubre los scripts disponibles en Nmap](./nmap-cheatsheet.md#encontrar-scripts-y-categorías)

Vamos a ejecutar algunos scripts para detectar vulnerabilidades conocidas en los servicios que hemos encontrado corriendo en el host.

*Nota: Al estar mapeado el puerto 80 del contenedor DVWA al puerto 8081 de la máquina host, debemos usar el puerto 8081 y el id de red de docker, para escanear el servicio web de DVWA. Si no lo hubiéramos mapeado, podríamos usar el puerto 80 y la ip del contenedor DVWA para escanearlo.*


```bash
nmap -sV -p puerto-del-servicio --script "http-enum,http-methods,http-headers,http-title" ip-del-host > /home/dvwa_scan.txt
```

### ❓ Pregunta 4:

Trata de interpretar la salida del escaneo para encontrar posibles vulnerabilidades en el servicio web de DVWA.

Ahora, usa la IA para interpretar la salida del escaneo y encontrar posibles vulnerabilidades en el servicio web de DVWA.

### ❓ Pregunta 5:

Logra logearte en DVWA usando alguna credencial que hayas descubierto y crea la base de datos de DVWA.

Sube una captura de pantalla de DVWA funcionando correctamente.

---