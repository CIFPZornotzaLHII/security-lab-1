- [Oinarrizko parametroak](#oinarrizko-parametroak)
  - [Denboralizazio txantiloia:](#denboralizazio-txantiloia)
  - [TCP connect scan](#tcp-connect-scan)
  - [Edozein host eskaneatu (aktiboak eta ez-aktiboak)](#edozein-host-eskaneatu-aktiboak-eta-ez-aktiboak)
  - [UDP protokoloa erabiliz eskaneatu](#udp-protokoloa-erabiliz-eskaneatu)
  - [Azpisare bateko host aktiboak aurkitu (Ping sweep)](#azpisare-bateko-host-aktiboak-aurkitu-ping-sweep)
  - [Sistema eragilearen ezaugarriak aurkitu](#sistema-eragilearen-ezaugarriak-aurkitu)
    - [Paketeen TTL erabiliz](#paketeen-ttl-erabiliz)
  - [Portuetako zerbitzuak detektatu](#portuetako-zerbitzuak-detektatu)
- [NMAP scriptak erabiltzen](#nmap-scriptak-erabiltzen)
  - [Scriptak eta kategoriak aurkitu](#scriptak-eta-kategoriak-aurkitu)
  - [Script multzo bat exekutatu](#script-multzo-bat-exekutatu)
  - [Kategoriaren araberako script multzo bat exekutatu](#kategoriaren-araberako-script-multzo-bat-exekutatu)

Erasotzaile gisa, funtsezkoa da jakitea zein portu dauden irekita.

## Oinarrizko parametroak

Portu guztiak eskaneatzeko:

```bash
nmap -p- 192.168.0.1
```

65.535 portu daude; guztiak eskaneatzeko:

```bash
nmap -p1-65535 192.168.0.1
```

---

- Portu bat `irekita`, `itxita` edo `iragazita` egon daiteke

Gehien erabiltzen diren 500 portuak eskaneatzeko:

```bash
nmap --top-ports 500 192.168.0.1
```

Irekita dauden portuak bakarrik ikusteko:

```bash
nmap --top-ports 500 --open 192.168.0.1
```

---

Modu xehea erabil dezakezu, eskaneoa mantso badoa eta aldi berean zein portu dauden irekita ikusi nahi baduzu:

```bash
nmap --top-ports 500 --open 192.168.0.1 -v
```

DNSrik ez erabiltzeko; horrela, eskaneoa azkarragoa izango da:

```bash
nmap --top-ports 500 --open 192.168.0.1 -v -n
```

---

### Denboralizazio txantiloia:

`-Tn` (`n`: 0 - 5)

0. Oso motela baina isila (modu paranoikoa)
5. Oso azkarra baina zaratatsua (modu oldarkorra)

```bash
nmap -p- -T5 192.168.0.1
```

---

### TCP connect scan

Eskaneo modu honetan, Nmap TCP protokoloaren `3-way handshake` prozesua osatzen saiatzen da. Gogoratu Wiresharken nola ikusten genituen pakete hauek:

- Portua irekita badago: `SYN -> SYN/ACK -> ACK`
- Portua itxita badago: `SYN -> RST`

```bash
nmap -p- -T5 -sT --open 192.168.0.1 -v -n
```

---

### Edozein host eskaneatu (aktiboak eta ez-aktiboak)

- Horretarako `-Pn` parametroa erabiliko dugu

```bash
nmap -p- -T5 --open 192.168.0.1 -v -n -Pn
```

---

### UDP protokoloa erabiliz eskaneatu

- Horretarako `-sU` parametroa erabiliko dugu

```bash
nmap --top-ports 100 --open -sU 192.168.0.1 -v -n
```

---

### Azpisare bateko host aktiboak aurkitu (Ping sweep)

```bash
nmap -sn 192.168.0.1/24
```

### Sistema eragilearen ezaugarriak aurkitu

⚠️ Ez da gehiegi gomendatzen, nahiko astuna izan baitaiteke.

```bash
nmap -O 192.168.0.1
```

#### Paketeen TTL erabiliz

TTLak (`Time To Live`) pakete batek sarean zenbat denbora edo zenbat jauzi egin ditzakeen adierazten du, bideratzaile batek baztertu aurretik. TTL kontzeptua beste testuinguru batzuetan ere erabiltzen da, adibidez CDN cachean edo DNS cachean.

Informazio-pakete bat sortzen denean eta Internet bidez bidaltzen denean, teorikoki bideratzailez bideratzaile mugarik gabe ibiltzeko arriskua egon liteke. Hori saihesteko, paketeek iraungitze-balio bat dute: TTL edo jauzi-muga. Horri esker, jakin daiteke pakete batek zenbat denbora daraman zirkulazioan, eta bidaltzaileak paketeak sarean egin duen ibilbideari buruzko pistak jaso ditzake.

Pakete bakoitzak TTL balio numeriko bat darama. Bideratzaile batek paketea jasotzen duen bakoitzean, balio horri unitate bat kentzen dio, eta ondorengo sare-puntura bidaltzen du. Une batean TTL balioa zerora iristen bada, bideratzaileak paketea baztertu eta ICMP mezu bat bidaltzen dio jatorrizko hostari.

Zer lotura du horrek sistema eragilea identifikatzearekin? Sistema eragile askok TTL balio lehenetsiak dituzte. Adibidez, Windows sistemetan ohikoa da TTL lehenetsia `128` izatea; Linux sistemetan, berriz, `64`.

Beraz, makina bati pakete bat bidaltzen badiogu eta `128` TTL duen erantzuna jasotzen badugu, litekeena da Windows exekutatzen aritzea. `64` TTL duen erantzuna jasotzen badugu, berriz, litekeena da Linux izatea.

Metodo hau ez da hutsezina, eta sare-administratzaile batek erraz engaina dezake; hala ere, egoera batzuetan lagungarria izan daiteke makina baten sistema eragilea identifikatzeko.

Jarraian, TTL balioak eta haiei lotutako sistema eragile arrunten adibideak dituzu:

Linux/Unix: 64
Windows: 128
MacOS: 64
Solaris/AIX: 254
FreeBSD: 64

Subin's Blog: https://subinsb.com/default-device-ttl-values

Era berean, TTL balioaren arabera sistema eragilea identifikatzen laguntzen duen Python script hau ere partekatzen dugu:

WhichSystem: https://pastebin.com/HmBcu7j2

### Portuetako zerbitzuak detektatu

```bash
nmap -p80 192.168.0.1 -sV
```

## NMAP scriptak erabiltzen

### Scriptak eta kategoriak aurkitu

```bash
# Scriptak
locate .nse

# Script kopurua
locate .nse | wc -l

# Kategoriak paraleloan aurkitu
locate .nse | xargs grep 'categories'

# Zerbitzuak iragazi
locate .nse | xargs grep 'categories' | grep -oP '".*?"' | sort -u
```

*Oharra: `locate` komandoa ez baduzu aurkitzen, baliteke instalatu eta fitxategien datu-basea eguneratu behar izatea. Honela egin dezakezu:*

```bash
sudo apt install locate
sudo updatedb
```

### Script multzo bat exekutatu

`-sC` parametroarekin, Nmap-ek ataka horretarako erabilgarrienak diren script lehenetsi batzuk exekutatzen ditu.

```bash
nmap -p22 192.168.0.1 -sC -sV
```

### Kategoriaren araberako script multzo bat exekutatu

```bash
nmap -p80 192.168.0.1 --script="vuln or/and safe" -sV
```

```bash
# Portu batean entzuten ari diren zerbitzuak ikusi
lsof -i:80

# Prozesu bat nondik exekutatzen den ikusi
pwdx idproceso
```
