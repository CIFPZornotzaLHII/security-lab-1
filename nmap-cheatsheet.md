
- [Parámetros básicos](#parámetros-básicos)
  - [Plantilla de temporizado:](#plantilla-de-temporizado)
  - [TCP connect scan](#tcp-connect-scan)
  - [Escanear cualquier host (activos e inactivos)](#escanear-cualquier-host-activos-e-inactivos)
  - [Escanear usando protocolo UDP](#escanear-usando-protocolo-udp)
  - [Encontrar hosts activos en una subred (Ping sweep)](#encontrar-hosts-activos-en-una-subred-ping-sweep)
  - [Encontrar características del sistema operativo](#encontrar-características-del-sistema-operativo)
    - [Usando el TTL de los paquetes](#usando-el-ttl-de-los-paquetes)
  - [Detectar servicios en puertos](#detectar-servicios-en-puertos)
- [Usando scripts NMAP](#usando-scripts-nmap)
  - [Encontrar scripts y categorías](#encontrar-scripts-y-categorías)
  - [Ejecutar un conjunto de scripts](#ejecutar-un-conjunto-de-scripts)
  - [Ejecutar un conjunto de scripts por categorías](#ejecutar-un-conjunto-de-scripts-por-categorías)


Como atacante, es funcional conocer qué puertos hay abiertos.

## Parámetros básicos

Para escanear todos los puertos

```bash
nmap -p- 192.168.0.1 
```

Hay 65535 puertos, para escanear todos ellos

```bash
nmap -p1-65535 192.168.0.1 
```
---

- El puerto puede estar Abierto, Cerrado o Filtrado

Para escanear los 500 puertos más utilizados:

```bash
nmap --top-ports 500 192.168.0.1 
```

Para escanear solo los puertos abiertos:

```bash
nmap --top-ports 500 --open 192.168.0.1 
```

---
Verboso, si el escaneo va lento y queremos saber qué puertos están abiertos al mismo tiempo:

```bash
nmap --top-ports 500 --open 192.168.0.1 -v
```

No usar DNS, el escaneo será más rápido:
```bash
nmap --top-ports 500 --open 192.168.0.1 -v -n
```

---

### Plantilla de temporizado:

-Tn  (n : 0 - 5) 
0. Muy lenta pero silenciosa (modo paranoico)
5. Muy rápida pero bulliciosa (modo loco)

```bash
nmap -p- -T5 192.168.0.1 
```

---
### TCP connect scan

En este modo de escaneo, nmap intentará realizar el 3-way-handshake del protocolo tcp. Recuerda cómo capturamos estos paquetes en Wireshark:
- Si el puerto está abierto (SYN ACK -> SYN -> ACK )
- Si el puerto está cerrado (SYN ACK -> RST )
  
```bash
nmap -p- -T5 -sT --open 192.168.0.1 -v -n
```

---
### Escanear cualquier host (activos e inactivos)

- Necesitaremos usar el parámetro -Pn


```bash
nmap -p- -T5 --open 192.168.0.1 -v -n -Pn
```

---
### Escanear usando protocolo UDP

- Usaremos el parámetro -Su para esto

```bash
nmap --top-ports 100 --open -Su 192.168.0.1 -v -n
```

---
### Encontrar hosts activos en una subred (Ping sweep)

```bash
nmap -sn 192.168.0.1/24
 ```

### Encontrar características del sistema operativo

⚠️ No se recomienda demasiado, es pesado.

```bash
nmap -O 192.168.0.1
```

#### Usando el TTL de los paquetes

El tiempo de vida (TTL), se refiere a la cantidad de tiempo o "saltos" durante los cuales un paquete debe existir dentro de una red antes de ser descartado por un enrutador. El TTL también se usa en otros contextos, como el almacenamiento en caché de CDN y el almacenamiento en caché de DNS.

Cuando se crea un paquete de información y se envía a través de Internet, existe el riesgo de que este pasar de enrutador en enrutador sin límite. Para aliviar esa posibilidad, los paquetes se diseñan con una expiración llamada tiempo de vida o límite de saltos. El TTL de los paquetes también puede ser útil para determinar exactamente cuánto tiempo un paquete específico ha estado circulando, y permite que el remitente reciba información sobre la ruta de un paquete a través de Internet.

Cada paquete tiene una ubicación donde se almacena un valor numérico, y ese valor determina cuánto tiempo debe continuar moviéndose en la red. Cada vez que un enrutador recibe un paquete, resta uno a la cantidad de TTL y lo pasa a la siguiente ubicación de la red. Si en algún momento la cantidad de TTL llega a cero después de la resta, el enrutador desechará el paquete y enviará un mensaje ICMP al host original.

¿Qué tiene que ver esto con la identificación del sistema operativo? Bueno, varios sistemas operativos tienen valores TTL predeterminados. Por ejemplo, en los sistemas operativos Windows, el valor TTL predeterminado es 128, y en los sistemas operativos Linux, es 64.

Entonces, si enviamos un paquete a una máquina y recibimos una respuesta con un valor TTL de 128, es probable que la máquina esté ejecutando Windows. Si recibimos una respuesta con un valor TTL de 64, es probable que la máquina esté ejecutando Linux.

Este método no es infalible y los administradores de red pueden engañarlo, pero en algunas situaciones puede ser útil para identificar el sistema operativo de una máquina.

A continuación, compartimos la página que se muestra en esta clase para identificar el sistema operativo correspondiente a los valores TTL existentes.

Linux/Unix: 64
Windows: 128
MacOS: 64
Solaris/AIX: 254
FreeBSD: 64

Subin 's Blog: https://subinsb.com/default-device-ttl-values 

De igual manera, compartimos un script en Python que se encarga de identificar el sistema operativo según el TTL obtenido:

WhichSystem: https://pastebin.com/HmBcu7j2

### Detectar servicios en puertos

```bash
nmap -p80 192.168.0.1 -sV
```

## Usando scripts NMAP

### Encontrar scripts y categorías


```bash
# Scripts
locate .nse

# Número de scripts
locate .nse | wc -l

# Encontrar categorías en paralelo
locate .nse | xargs grep 'categories'

# Filtrar servicios
locate .nse | xargs grep 'categories' | grep -oP '".*?"' | sort -u

```

*Nota: Si no encuentras el comando locate, es posible que necesites instalarlo y actualizar la base de datos de archivos. Puedes hacerlo con los siguientes comandos:*

```bash
sudo apt install locate
sudo updatedb
```

### Ejecutar un conjunto de scripts
Ejecutar un conjunto de scripts, con el parámetro -sC se ejecutará un conjunto de scripts más utilizados desde los que se encuentren disponibles en ese puerto.

```bash
nmap -p22 192.168.0.1 -sC -sV
```


### Ejecutar un conjunto de scripts por categorías

```bash
nmap -p80 192.168.0.1 --script="vuln or/and safe" -sV 
```
 

```bash
# Ver servicios en el puerto
lsof -i:80

# Para encontrar desde dónde se ejecuta un proceso
pwdx idproceso 
```