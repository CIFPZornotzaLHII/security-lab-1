# 🐳 Docker laborategia

![alt text](image.png)

- [🐳 Docker laborategia](#-docker-laborategia)
- [🛡️ Erasotzailearen makina instalatzea (Kali Linux)](#️-erasotzailearen-makina-instalatzea-kali-linux)
  - [⚙️ Kali Linux instalatu](#️-kali-linux-instalatu)
  - [🔓 Makinan sartu](#-makinan-sartu)
  - [🛠️ Tresnen instalazioa](#️-tresnen-instalazioa)
- [🎯 Makina zaurgarri bat gehitzen (DVWA)](#-makina-zaurgarri-bat-gehitzen-dvwa)
- [🔍 Sarea Nmap-ekin eskaneatzen](#-sarea-nmap-ekin-eskaneatzen)
  - [📡 Azpisare osoa eskaneatu](#-azpisare-osoa-eskaneatu)
    - [❓ 1. galdera:](#-1-galdera)
  - [📊 Aurkitutako hostei informazioa ateratzea](#-aurkitutako-hostei-informazioa-ateratzea)
    - [❓ 2. galdera:](#-2-galdera)
    - [❓ 3. galdera:](#-3-galdera)
  - [🔐 NMAP scriptak erabiltzen](#-nmap-scriptak-erabiltzen)
    - [❓ 4. galdera:](#-4-galdera)
    - [❓ 5. galdera:](#-5-galdera)

---
# 🛡️ Erasotzailearen makina instalatzea (Kali Linux)

## ⚙️ Kali Linux instalatu

Kali Linux Docker erabiliz instalatzeko, `docker-compose.yml` fitxategi bat sortuko dugu honako edukiarekin:

```yml
services:
  kali:
    image: kalilinux/kali-rolling
    container_name: kali
    restart: unless-stopped
    tty: true
    network_mode: "host" # Honek Dockerreko beste edozein sare baliogabetzen du
    stdin_open: true
    volumes:
      - ./home:/home
    cap_add:
      - NET_ADMIN
      - NET_RAW
```

Kali makina Dockerreko beste sare batera gehitu nahi baduzu, `network_mode: "host"` lerroa kendu edo komentatu, eta dagokion sarean gehitu.

## 🔓 Makinan sartu

```bash
docker exec -it kali bash
```

Pakete-kudeatzailea eta sistema eragilea eguneratzen ditugu:

```bash
apt update && apt upgrade -y
```

## 🛠️ Tresnen instalazioa

Kali Linuxen irudia oso arina denez, ez ditu tresna asko aurrez instalatuta ekartzen. Laborategirako beharrezko tresnak instalatzeko, exekutatu komando hau:

```bash
apt install -y sudo nmap net-tools locate iproute2
```

Tresnak instalatu ondoren, laborategirako prestatutako Kali Linux irudi pertsonalizatu bat sor dezakegu. Horrela, irudiak leku gehiago hartuko du, baina etorkizunean makina prest izango dugu tresnak berriro instalatu beharrik gabe. Irudi pertsonalizatua sortzeko, exekutatu:

```bash
docker commit kali kali-custom
```

Hemendik aurrera, tresnak instalatuta dituen Kali makina erabili nahi duzunean, `kalilinux/kali-rolling` irudiaren ordez `kali-custom` irudia erabil dezakezu. Horretarako, `docker-compose.yml` fitxategiko irudiaren lerroa aldatu besterik ez duzu egin behar.

---

# 🎯 Makina zaurgarri bat gehitzen (DVWA)

DVWA (Damn Vulnerable Web Application) izeneko makina zaurgarri bat gehituko dugu, sare-eskaneoa eta zaurgarritasunen analisia praktikatzeko. Horretarako, gehitu zerbitzu hau `docker-compose.yml` fitxategira:

```yml
    dvwa:
        image: vulnerables/web-dvwa
        container_name: dvwa
        restart: unless-stopped
        ports:
        - "8081:80"
```

Orain Dockerreko sare berri bat sortuko dugu bi makinak konektatzeko. Gehitu atal hau `docker-compose.yml` fitxategiaren amaieran:

```yml
networks:
  lab-network:
    driver: bridge
```

Ondoren, konektatu bi makinak sare horretara, zerbitzu bakoitzean lerro hau gehituta:

```yml
    networks:
      - lab-network
```

Orain prest gaude gure lehen sare-eskaneoa egiteko.

# 🔍 Sarea Nmap-ekin eskaneatzen

Nmap sareak eskaneatzeko tresna bat da, sare batean dauden hostak eta zerbitzuak aurkitzeko aukera ematen duena.

Tresna honek aukera ugari eskaintzen ditu eskaneoa pertsonalizatzeko.

Begiratu [Nmap-en dokumentazio ofiziala](https://nmap.org), haren gaitasunei buruz gehiago ikasteko eta modu eraginkorrean erabiltzeko. Bestela, biltegi honetan dagoen laburpen praktikoa ere erabil dezakezu: [Cheat Sheet](./nmap-cheatsheet-eu.md).

## 📡 Azpisare osoa eskaneatu

Hasi aurretik, exekutatu komando hau konektatuta gauden azpisarea identifikatzeko:

```bash
ip a
```

Beste aukera bat `docker network inspect sarearen_izena` erabiltzea da, Dockerreko sareari eta hari konektatutako edukiontziei buruzko informazioa lortzeko. Kasu honetan, ordea, ekipamenduak Nmap erabiliz aurkitu nahi ditugu.

*Azpisare oso bat eskaneatzeak sareko host aktibo guztiak aurkitzeko balio du. Kontuz ibili behar da mota honetako eskaneoak egitean, sarean trafiko handia sor dezakeelako, segurtasun-sistemek detekta dezaketelako eta amaitzeko denbora luzea behar dezakeelako.*

```bash
nmap --top-ports 100 -T5 --open Ip_de_la_subred/mascara_de_la_subred -v -n
```
- **--top-ports 100**: Gehien erabiltzen diren 100 portuak eskaneatzen ditu
- **-T5**: Eskaneo modurik azkarrena erabiltzen du (erasokorra)
- **--open**: Irekitako portuak bakarrik erakusten ditu
- **-n**: Ez ditu host izenak ebazten (ez du DNS erabiltzen)
- **-v**: Modu xehea, eskaneoan zehar informazio gehiago erakusten du
- **Ip_de_la_subred/mascara_de_la_subred**: Eskaneatuko den azpisarea zehazten du, adibidez `192.168.1.0/24`

Gehitu hemen azpisare osoaren eskaneoari buruz garrantzitsutzat jotzen duzun informazioa.

```bash

```

### ❓ 1. galdera:

Zenbat host aktibo aurkitu dituzu sarean?

## 📊 Aurkitutako hostei informazioa ateratzea

Host aktibo bat aurkitu ondoren, Nmap erabil dezakegu host horretan martxan dauden zerbitzuei, irekitako portuei, sistema eragileari eta abarri buruzko informazio zehatzagoa ateratzeko; ping soil batekin ez genuke hainbesteko informaziorik lortuko.

```bash
nmap -p- -sV Ip_del_host -v
```
- **-p-**: Portu guztiak eskaneatzen ditu (1-65535)
- **-sV**: Irekitako portuetan martxan dauden zerbitzuen bertsioak detektatzen ditu
- **Ip_del_host**: Eskaneatu beharreko hosta zehazten du

### ❓ 2. galdera:

Zer zerbitzu aurkitu dituzu hostean martxan? Zer bertsio aurkitu dituzu? Lortu al duzu hostaren sistema eragilea identifikatzea?

### ❓ 3. galdera:

Egin Interneteko bilaketa bat aurkitu dituzun zerbitzu eta bertsioekin lotutako zaurgarritasun ezagunak identifikatzeko.

[CVE Details](https://www.cvedetails.com/) edo [Exploit Database](https://www.exploit-db.com/) bezalako guneak erabil ditzakezu horretarako.

## 🔐 NMAP scriptak erabiltzen

Oinarrizko aukerez gain, Nmap-ek script ugari eskaintzen ditu eskaneo aurreratuagoak egiteko: zaurgarritasunak detektatzeko, zerbitzuei buruzko informazio gehigarria eskuratzeko, eta abar. [Ikusi Nmap-en script erabilgarriak](./nmap-cheatsheet-eu.md#scriptak-eta-kategoriak-aurkitzea)

Orain script batzuk exekutatuko ditugu hostean aurkitu ditugun zerbitzuetan zaurgarritasun ezagunak bilatzeko.

*Oharra: DVWA edukiontziaren 80 ataka ostalari-makinako 8081 atakara mapatuta dagoenez, 8081 ataka eta Dockerreko sarearen IDa erabili behar ditugu DVWAren web zerbitzua eskaneatzeko. Portua mapatuta egongo ez balitz, 80 ataka eta DVWA edukiontziaren IPa erabil genitzake.*

```bash
nmap -sV -p puerto-del-servicio --script "http-enum,http-methods,http-headers,http-title" ip-del-host > /home/dvwa_scan.txt
```

### ❓ 4. galdera:

Saiatu eskaneoaren irteera interpretatzen, DVWAren web zerbitzuan egon litezkeen zaurgarritasunak aurkitzeko.

Ondoren, erabili IA eskaneoaren irteera interpretatzeko eta DVWAren web zerbitzuko zaurgarritasun posibleak identifikatzeko.

### ❓ 5. galdera:

Lortu DVWAn saioa hastea aurkitu duzun kredentzialen bat erabiliz, eta sortu DVWAren datu-basea.

Igo DVWA behar bezala dabilela erakusten duen pantaila-argazki bat.

---
