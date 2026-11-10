# Számítógép-hálózatok – Hálózatelemzés és szimuláció

Gyakorlatorientált hálózati projekt, amely a hálózati protokollok, a csomagelemzés, a hálózati forgalom megfigyelése, a hálózati konfiguráció és a biztonság témaköreit mutatja be **Wireshark** és **Cisco Packet Tracer** segítségével.

A projekt elméleti hálózati ismereteket és gyakorlati szimulációkat egyaránt tartalmaz. Bemutatja, hogyan kommunikálnak egymással az eszközök, miként vizsgálható a hálózati forgalom, valamint hogyan befolyásolják a különböző hálózati komponensek a teljesítményt és a biztonságot.

## Projekt áttekintése

A projekt két fő területre osztható:

- **Hálózati protokollok elemzése:** TCP, ICMP, HTTP, csomagstatisztikák és forgalomszűrés vizsgálata Wireshark segítségével.
- **Hálózattervezés és szimuláció:** Hálózati topológiák létrehozása és tesztelése, routerek konfigurálása, tűzfalak működésének vizsgálata, valamint hubok és switchek összehasonlítása Cisco Packet Tracer használatával.

---

# 1. rész – Hálózati protokollok elemzése

## 1. TCP-vezérlőbitek (TCP Flags)

A TCP (Transmission Control Protocol) vezérlőbiteket használ a kapcsolatok kezelésére, a fogadott adatok nyugtázására és a kapcsolatok lezárására.

A projekt során a következő hat fő TCP-vezérlőbitet vizsgáltam:

| Vezérlőbit | Funkció |
|---|---|
| SYN | TCP-kapcsolat kezdeményezése és a sorszámok szinkronizálása. |
| ACK | A fogadott adatok vagy kapcsolati üzenetek nyugtázása. |
| FIN | Jelzi, hogy az eszköz befejezte az adatküldést, és le szeretné zárni a kapcsolatot. |
| RST | Azonnal megszakítja vagy visszautasítja a TCP-kapcsolatot. |
| PSH | Kéri, hogy a fogadott adatok szükségtelen pufferelési késleltetés nélkül továbbításra kerüljenek az alkalmazás felé. |
| URG | Jelzi, hogy a sürgősségi mutató mező fontos, és sürgős adat található a csomagban. |

### A TCP háromlépéses kézfogása (Three-Way Handshake)

A TCP-kapcsolat általában egy háromlépéses kézfogási folyamattal jön létre:

1. **SYN:** A kliens kapcsolódási kérelmet küld a szervernek.
2. **SYN-ACK:** A szerver nyugtázza a kérést, majd elküldi saját szinkronizációs üzenetét.
3. **ACK:** A kliens nyugtázza a szerver válaszát, ezzel befejeződik a kézfogás.

Ez a folyamat megbízható kapcsolatot hoz létre, mielőtt megkezdődne a normál adatátvitel.

**Képernyőkép – TCP SYN, SYN-ACK és ACK csomagok**

<img src="./screenshots/SYN-ACK%20Flag.png" width="800">

### TCP-kapcsolat lezárása

A TCP-kapcsolat általában egy négylépéses folyamattal zárul le:

1. A kliens FIN csomagot küld.
2. A szerver ACK csomaggal nyugtázza azt.
3. A szerver elküldi saját FIN csomagját.
4. A kliens ACK csomaggal nyugtázza a szerver kérését.

Ez lehetővé teszi, hogy mindkét fél külön-külön lezárja a kapcsolatot, és befejezze a még folyamatban lévő adatátvitelt.

**Képernyőkép – TCP FIN és ACK csomagok**

<img src="./screenshots/FIN%20Flag.png" width="800">

---

## 2. ICMP – Ping és Echo Request

Az ICMP (Internet Control Message Protocol) hálózati diagnosztikára és hibajelzések továbbítására szolgál. A TCP/IP-modell internetes rétegén működik.

Az egyik leggyakoribb felhasználási módja a `ping` parancs, amely ellenőrzi, hogy egy adott célállomás elérhető-e.

A projekt során a Google nyilvános DNS-szerverének címét (`8.8.8.8`) használtam ICMP Echo Request csomagok célállomásaként.

A fontosabb csomagmezők:

| Mező | Leírás |
|---|---|
| Destination | A cél IP-cím, amely fogadja az Echo Request üzenetet. |
| Length | A teljes csomag hossza, a megjelenített protokollrétegtől függően. |
| Type 8 | ICMP Echo Request (visszhangkérés). |
| Type 0 | ICMP Echo Reply (visszhangválasz). |
| Identifier | Segít az egyes kérések és a hozzájuk tartozó válaszok párosításában. |
| Sequence Number | Az egyes Echo Request üzenetek sorrendjét azonosítja. |
| TTL | Korlátozza, hogy a csomag hány hálózati ugráson haladhat keresztül. |
| Response Time | Megmutatja, mennyi idő alatt érkezik válasz a célállomástól. |

Az Echo Reply sikeres fogadása azt jelzi, hogy a célállomás az adott hálózati útvonalon keresztül elérhető.

**Képernyőkép – ICMP Echo Request és Echo Reply a Wiresharkban**

<img src="./screenshots/ICMP%20and%20Echo.png" width="800">

<img src="./screenshots/Echo%20Request%20Type%200.png" width="800">

---

## 3. ICMP Traceroute

A Traceroute egy hálózati diagnosztikai eljárás, amelynek segítségével azonosíthatók azok a routerek, amelyeken a csomagok áthaladnak a célállomás felé vezető úton.

A működés alapja az IP-fejlécben található **Time to Live (TTL)** érték módosítása.

Minden router, amely továbbít egy csomagot, eggyel csökkenti annak TTL-értékét. Amikor a TTL eléri a nullát, a router eldobja a csomagot, és általában egy ICMP Time Exceeded üzenetet küld vissza.

A folyamat a következőképpen működik:

1. A Traceroute 1-es TTL-értékkel küld egy csomagot.
2. Az első router nullára csökkenti a TTL-t, majd ICMP Time Exceeded üzenetet küld vissza (Type 11).
3. A következő csomag 2-es TTL-értékkel indul, így elérheti a második routert.
4. A TTL értéke minden további próbálkozásnál növekszik, amíg a csomag el nem éri a célállomást.
5. A célállomás válaszol, így azonosítható az útvonal végpontja.

A folyamat során megfigyelhető legfontosabb ICMP-üzenettípusok:

| ICMP-típus | Leírás |
|---|---|
| Type 8 | Echo Request |
| Type 11 | Time Exceeded |
| Type 0 | Echo Reply |

Ezeknek a csomagoknak a vizsgálatával azonosíthatók az egyes hálózati ugrások, valamint mérhető az útvonalon található routerek válaszideje.

**Képernyőkép – Traceroute csomagelemzés**

<img src="./screenshots/Echo%20Request%20Type%2011.png" width="800">

<img src="./screenshots/Echo%20Request%20Type%208.png" width="800">

---

## 4. Hálózati forgalom megfigyelése

A Wireshark grafikus statisztikai eszközöket biztosít, amelyek segítségével időben vizualizálható a hálózati aktivitás.

A **Packets per Second (PPS)** grafikon az egyes időintervallumokban rögzített csomagok számát jeleníti meg.

A grafikon legfontosabb elemei:

- **Időtengely:** A csomagrögzítés kezdete óta eltelt időt mutatja.
- **Csomagtengely:** Az adott időintervallumban rögzített csomagok számát jelzi.
- **Vonal- és oszlopdiagramok:** Különböző módokon teszik szemléletessé a forgalmi mintázatokat.
- **Megjelenítési szűrők:** Lehetővé teszik meghatározott protokollok vagy csomagtípusok kiválasztását.
- **Nagyítás és mozgatás:** Segít az egyes időszakok részletesebb vizsgálatában.
- **Logaritmikus skála:** Megkönnyíti a nagy eltéréseket mutató forgalmi mintázatok áttekintését.
- **Napszak szerinti idő:** Az eltelt idő helyett a napszakhoz viszonyított időbélyegeket jeleníti meg.
- **Automatikus frissítés:** Új csomagok rögzítésekor frissíti a grafikont.
- **Jelmagyarázat:** Azonosítja a megjelenített adatokat és azok színeit.
- **Intervallum:** Meghatározza, hogy egy adatpont milyen időtartamot képvisel.

A rövidebb időintervallumok hasznosak a hirtelen forgalmi kiugrások és ingadozások azonosításához, míg a hosszabb intervallumok átfogóbb képet adnak a hálózati aktivitásról.

**Képernyőkép – Wireshark Packets per Second grafikon**

<img src="./screenshots/All%20Packets%20per%20sec.png" width="800">

---

## 5. HTTP – Hypertext Transfer Protocol

A HTTP egy alkalmazási rétegbeli protokoll, amelyet a webes kliensek és szerverek közötti kommunikációra használnak.

Amikor egy felhasználó megnyit egy weboldalt, a böngésző HTTP-kérést küld a szervernek. A szerver feldolgozza a kérést, majd HTTP-választ küld vissza.

### HTTP-kérés elemzése

A projekt során vizsgált HTTP-kérés a `GET` metódust használta.

A legfontosabb kérésmezők:

| Mező | Leírás |
|---|---|
| Request Method | Meghatározza a kért műveletet, például a GET metódust. |
| Request URI | Azonosítja a kért erőforrást, például a `/` útvonalat. |
| HTTP Version | Meghatározza a protokoll verzióját, például a HTTP/1.1-et. |

A GET metódust általában erőforrások, például weboldalak, képek és egyéb szervertartalmak lekérésére használják.

### HTTP-válasz elemzése

A szerver HTTP-válasszal reagál, amely információt tartalmaz a kérés eredményéről.

A legfontosabb válaszmezők:

| Mező | Leírás |
|---|---|
| Response Version | A szerver által használt HTTP-verzió. |
| Status Code | A kérés eredményét leíró numerikus kód. |
| Status Description | A státuszkód rövid szöveges magyarázata. |

Néhány gyakori HTTP-státuszkód:

- **200 OK:** A kérés sikeresen végrehajtódott.
- **404 Not Found:** A kért erőforrás nem található.
- **500 Internal Server Error:** A szerver hibába ütközött a kérés feldolgozása során.

### Kliens–szerver kommunikáció

Az alapvető kommunikációs folyamat:

1. A böngésző HTTP GET kérést küld.
2. A szerver fogadja és feldolgozza a kérést.
3. A szerver HTTP-választ küld vissza, amely tartalmazza a státuszkódot és szükség esetén a kért tartalmat.
4. A böngésző fogadja és megjeleníti a visszaküldött erőforrást.

**Képernyőkép – HTTP GET kérés a Wiresharkban**

<img src="./screenshots/HTTP%20Request%20Details.png" width="800">

**Képernyőkép – HTTP-válasz és státuszkód**

<img src="./screenshots/HTTP%20Response%20details.png" width="800">

---

## 6. Hálózati forgalmi statisztikák

A Wireshark további statisztikai eszközöket biztosít a csomagok viselkedésének és a hálózati teljesítménynek az elemzéséhez.

### Csomagméret

A csomagméret befolyásolja, hogy egyetlen csomagban mennyi információ továbbítható.

A nagyobb csomagok viszonylag kisebb protokolltöbblettel több adatot képesek továbbítani, míg a kisebb csomagok ugyanannyi információ továbbításához több fejlécet igényelhetnek.

A rögzített forgalomban a 41. és az 54. másodperc környékén jelentősebb csomagméret-növekedés volt megfigyelhető. Ezek a kiugrások egybeestek a letöltési aktivitás időszakaival.

### Átviteli időközök

Az átviteli időközök az egymást követő csomagok elküldése között eltelt időt jelentik.

- A rövid időközök gyakori csomagküldésre és fokozott hálózati aktivitásra utalhatnak.
- A rendszertelen időközök jittert vagy egyenetlen forgalmi mintázatokat jelezhetnek.
- A hosszabb szünetek csökkent aktivitásra vagy az adatátvitel megszakadására utalhatnak.

### Hálózati torlódás

Hálózati torlódás akkor alakul ki, amikor a forgalom mennyisége meghaladja a rendelkezésre álló hálózati kapacitást.

Lehetséges következményei:

- Megnövekedett késleltetés.
- Csomagvesztés.
- TCP-újraküldések.
- Csökkent tényleges adatátviteli sebesség.

A rögzített forgalomban a 10. másodperc környékén hirtelen forgalomnövekedés volt látható, amely egy szokatlan forgalmi mintázat példájaként további vizsgálatot indokolhat.

### Időintervallumok elemzése

A különböző grafikonintervallumok eltérő részletességű képet biztosítanak.

**Rövid intervallumok (körülbelül 0,1–10 másodperc):**
- Hasznosak a hirtelen forgalmi kiugrások azonosítására.
- Segítenek a csomagvesztés és a rövid idejű torlódások vizsgálatában.
- Részletesebb képet adnak letöltések és streamingtevékenységek elemzésekor.

**Hosszú intervallumok (10 másodperc vagy több):**
- Áttekinthetőbb képet adnak a hálózati teljesítményről.
- Segítenek a hosszabb távú forgalmi mintázatok azonosításában.
- Alkalmasak az ismétlődő tevékenységek, például az ütemezett letöltések vagy biztonsági mentések megfigyelésére.

**Képernyőkép – Csomagméret és hálózati forgalmi statisztikák**

<img src="./screenshots/Statistics%20Feature.png" width="800">

---

## 7. Wireshark forgalomszűrők

A Wireshark szűrői lehetővé teszik, hogy a hálózati forgalmat meghatározott protokollok, eszközök, portok és kommunikációs mintázatok alapján szűkítsük.

Két fő szűrőtípus létezik.

### Rögzítési szűrők (Capture Filters)

A rögzítési szűrőket **a csomagrögzítés megkezdése előtt** kell alkalmazni.

Ezek határozzák meg, hogy a Wireshark mely csomagokat rögzítse, így csökkenthető az összegyűjtött adatok mennyisége.

A Berkeley Packet Filter (BPF) szintaxisát használják, és többek között az alábbiak alapján szűrhetnek:

- IP-címek.
- Hálózati alhálózatok.
- Protokollok.
- Portszámok.

### Megjelenítési szűrők (Display Filters)

A megjelenítési szűrőket **a csomagok rögzítése után** lehet alkalmazni.

Elrejtik azokat a csomagokat, amelyek nem felelnek meg a megadott feltételeknek, de nem törlik őket a rögzítésből.

A Wireshark saját szűrőszintaxisát használják, és különösen hasznosak akkor, amikor egy már rögzített forgalomban szeretnénk meghatározott kommunikációt megvizsgálni.

### Haladó megjelenítési szűrők – példák

**HTTP- és HTTPS-forgalom szűrése**

```wireshark
tcp.port == 80 || tcp.port == 443
```

Megjeleníti a 80-as és 443-as TCP-portokat használó forgalmat, amelyek jellemzően a HTTP- és HTTPS-kommunikációhoz kapcsolódnak.

**Két meghatározott eszköz közötti kommunikáció szűrése**

```wireshark
ip.src == 192.168.42.178 && ip.dst == 95.100.104.29
```

Megjeleníti a megadott forrás IP-címről a megadott cél IP-címre továbbított csomagokat.

**ARP- és ICMP-forgalom kizárása**

```wireshark
!(arp || icmp)
```

Elrejti az ARP- és ICMP-csomagokat, így könnyebb más protokollokra, például HTTP-re, DNS-re vagy FTP-re koncentrálni.

A feltételek logikai operátorokkal történő kombinálása hatékonyabbá teszi a konkrét hálózati kommunikáció vizsgálatát.

**Képernyőkép – Haladó Wireshark megjelenítési szűrők**

<img src="./screenshots/Advanced Filter%20tcp%20port%20==%2080%20II%20top%20port%20==%20443.png" width="800">

---

# 2. rész – Hálózattervezés és szimuláció

A Cisco Packet Tracer segítségével hálózati topológiákat hoztam létre, eszközöket konfiguráltam, és teszteltem a különböző hálózati komponensek közötti kommunikációt.

## 8. Kis méretű homogén hálózat – Routerek konfigurálása

Létrehoztam egy kisebb hálózatot, amely három routerből, három switchből és három számítógépből állt.

A routereket soros kapcsolatokkal kötöttem össze, míg a többi eszközt Copper Straight-Through kábelekkel csatlakoztattam.

### Hálózati konfiguráció

Minden routeren az alábbi beállításokat végeztem el:

- Soros interfészek konfigurálása a szomszédos routerekkel történő kommunikációhoz.
- FastEthernet interfészek beállítása a helyi hálózatokhoz.
- IP-címek hozzárendelése a megfelelő interfészekhez.
- RIP útválasztási protokoll konfigurálása a hálózati információk cseréjéhez.

Példa a címzési sémára:

| Hálózat | Router interfész | Példa IP-cím |
|---|---|---|
| 1. hálózat | FastEthernet 0/0 | 192.168.1.1 |
| 2. hálózat | FastEthernet 0/0 | 192.168.2.1 |
| 3. hálózat | FastEthernet 0/0 | 192.168.3.1 |

A soros interfészek a `11.0.0.0` hálózatból kaptak IP-címeket, hogy lehetővé váljon a routerek közötti kommunikáció.

### RIP konfiguráció

A Routing Information Protocol (RIP) segítségével a routerek útválasztási információkat cserélnek, és megtanulják azokat a hálózatokat is, amelyekhez nincsenek közvetlenül csatlakoztatva.

Ennek köszönhetően a különálló helyi hálózatokon található eszközök kézi útvonal-konfigurálás nélkül is kommunikálhatnak egymással.

### Kapcsolat tesztelése

A routerek és számítógépek konfigurálása után a parancssorban a `ping` parancs segítségével teszteltem a különböző hálózatokhoz tartozó eszközök közötti kommunikációt.

A négy sikeres válasz és a nulla százalékos csomagvesztés azt jelezte, hogy az eszközök képesek kommunikálni a konfigurált hálózaton keresztül.

**Képernyőkép – Háromrouteres hálózati topológia**

<img src="./screenshots/Small Homogenous Network.png" width="800">

**Képernyőkép – Routerkonfiguráció és RIP**

<img src="./screenshots/Router0%20config.png" width="300"> <img src="./screenshots/Router1%20config.png" width="300"> <img src="./screenshots/Router2%20config.png" width="300">

**Képernyőkép – Sikeres ping különböző hálózatok között**

<img src="./screenshots/PC0%20Pings.png" width="800">

---

## 9. Tűzfal konfigurálása és tesztelése

Létrehoztam egy kisebb hálózatot, amely egy switchből, három számítógépből és egy szerverből állt. A hálózatot a tűzfal működésének bemutatására konfiguráltam.

A cél annak tesztelése volt, hogy a tűzfalszabályok hogyan korlátozhatják az eszközök közötti kommunikációt.

### Konfiguráció

1. A számítógépeket és a szervert ugyanahhoz a switchhez csatlakoztattam.
2. Minden eszközhöz külön IP-címet rendeltem.
3. Beállítottam a szerver tűzfalának paramétereit.
4. Létrehoztam egy szabályt az ICMP-forgalom tiltására.
5. Engedélyeztem a tűzfalat, majd ping parancsokkal teszteltem a kapcsolatot.

### Tesztelési eredmények

A tűzfalszabály alkalmazása után az egyik számítógépről pingeltem a többi eszközt.

A többi eszköz kérés-időtúllépéssel válaszolt, miközben a számítógép saját magát továbbra is sikeresen tudta pingelni.

Ez bemutatta, hogyan korlátozhatják a tűzfalszabályok bizonyos hálózati forgalomtípusok használatát, és hogyan akadályozhatják meg az eszközök közötti, meghatározott protokollokon keresztüli kommunikációt.

**Képernyőkép – Tűzfal konfigurációja**

<img src="./screenshots/Firewall%20network.png" width="800">

**Képernyőkép – Ping tesztek bekapcsolt tűzfallal**

<img src="./screenshots/PC3%20Pings.png" width="800">

---

## 10. Hubok és switchek összehasonlítása

A Cisco Packet Tracerben két különálló hálózati szimulációt készítettem a hubok és a switchek működésének összehasonlítására.

Mindkét topológia három számítógépből állt, amelyeket egy központi hálózati eszközhöz csatlakoztattam.

Minden számítógéphez egyedi IP-címet rendeltem ugyanazon a hálózaton belül, majd ping kérésekkel teszteltem az eszközök közötti kommunikációt.

### Hub szimuláció

A hub az OSI-modell fizikai rétegén (Layer 1) működik.

Amikor egy hub keretet fogad, a beérkező jelet az összes többi csatlakoztatott port felé továbbítja, ahelyett hogy kizárólag a célállomásnak küldené tovább.

**A hubok főbb hátrányai:**

- Minden csatlakoztatott eszköz megkapja a továbbított forgalmat.
- Az eszközök egy közös ütközési tartományon osztoznak.
- Félduplex módban működnek.
- Ütközések történhetnek, ha több eszköz egyszerre küld adatot.
- A hálózati forgalmat a többi csatlakoztatott eszköz könnyebben megfigyelheti.
- A szükségtelen forgalom csökkenti a rendelkezésre álló hálózati sávszélességet.

### Switch szimuláció

A hálózati switch elsősorban az OSI-modell adatkapcsolati rétegén (Layer 2) működik.

A hubbal ellentétben a switch megtanulja az eszközök MAC-címeit, és a MAC-címtáblázat alapján a megfelelő célport felé továbbítja a kereteket.

Ez növeli a hálózat hatékonyságát, mivel csökkenti a többi eszköz felé szükségtelenül továbbított forgalmat.

### Összehasonlítás

| Tulajdonság | Hub | Switch |
|---|---|---|
| OSI-réteg | Layer 1 | Layer 2 |
| Továbbítás | A jeleket a többi portra ismétli | MAC-címek alapján továbbítja a kereteket |
| Ütközési tartományok | Egy közös tartomány | Portonként külön ütközési tartomány |
| Duplex mód | Félduplex | Jellemzően full-duplex |
| Hatékonyság | Alacsonyabb | Magasabb |
| Biztonság | A forgalom könnyebben hozzáférhető a csatlakoztatott eszközök számára | Jobb forgalomelkülönítés |

A switchek általában lényegesen hatékonyabbak és biztonságosabbak, mint a hagyományos hubok.

További biztonság érhető el hálózati szegmentálással, VLAN-ok használatával és megfelelő switchbiztonsági konfigurációval.

**Képernyőkép – Hub hálózati szimuláció**

<img src="./screenshots/PC3%20Desktop%20ping.png" width="800">

**Képernyőkép – Switch hálózati szimuláció**

<img src="./screenshots/PC4%20Ping%20Cisco.png" width="800">

---

# Technológiák és eszközök

| Technológia | Felhasználási terület |
|---|---|
| Wireshark | Csomagrögzítés, protokollelemzés, forgalomszűrés és statisztikák. |
| Cisco Packet Tracer | Hálózattervezés, eszközkonfiguráció és hálózati szimuláció. |
| TCP/IP | Hálózati kommunikáció és szállítási protokollok megértése. |
| ICMP | Hálózati diagnosztika, ping és traceroute. |
| HTTP | Kliens–szerver webkommunikáció elemzése. |
| RIP | Dinamikus útválasztás összekapcsolt hálózatok között. |
| Ethernet | Vezetékes hálózati kommunikáció az eszközök között. |

# Bemutatott készségek

A projekt során az alábbi területeken szereztem gyakorlati tapasztalatot:

- TCP-kapcsolatok létrehozásának és lezárásának megértése.
- Hálózati protokollok csomagszintű elemzése.
- ICMP-alapú diagnosztika és traceroute.
- HTTP kliens–szerver kommunikáció vizsgálata.
- Hálózati forgalom megfigyelése és statisztikai elemzése.
- Wireshark rögzítési és megjelenítési szűrők használata.
- IP-címzés és alhálózat-alapú hálózattervezés.
- Routerek konfigurálása és dinamikus útválasztás RIP segítségével.
- Tűzfal konfigurálása és hálózati kapcsolat tesztelése.
- A hubok és switchek közötti különbségek megértése.
- Hálózati torlódások és lehetséges teljesítményproblémák azonosítása.

# Készítette

**Laszlo Major**

Copyright © 2026 Laszlo Major. Minden jog fenntartva.
