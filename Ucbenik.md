# ESP32: Arhitektura in strojna oprema ESP32

## 1.1 Strojna oprema (Hardware)

  ESP32 ni zgolj mikrokrmilnik, temveč System on a Chip (SoC), ki ga je razvilo podjetje Espressif Systems. Njegova glavna prednost pred predhodniki (npr. ESP8266 ali Arduino Uno) je integracija dveh procesorskih jeder ter vgrajenih modulov za Wi-Fi in Bluetooth komunikacijo.

Tehnične specifikacije (standardni model WROOM):
- Procesor: Xtensa® Dual-Core 32-bit LX6 (do 240 MHz).

- Pomnilnik: 520 KB notranjega SRAM; zunanji Flash (običajno 4 MB).

- Brezžična povezljivost: Wi-Fi 802.11 b/g/n in Dual-mode Bluetooth (Classic + BLE).

- Periferne enote: 12-bitni ADC, 8-bitni DAC, kapacitivna tipala na dotik, SPI, I2C, UART, PWM.


## 1.2 Različice in razvojne ploščice

Pri izbiri strojne opreme moramo razlikovati med čipom, modulom in razvojno ploščico.

- Čip: Goli silicij (npr. ESP32-D0WDQ6).

- Modul (npt. WROOM-32): Čip z dodanim kristalnim oscilatorjem, anteno in Flash pomnilnikom, zaprt v kovinsko ohišje.

- Razvojna ploščica (DevKit): Modul, nameščen na tiskano vezje (PCB) z USB-to-UART pretvornikom, regulatorjem napetosti (3,3 V) in pinskimi letvami za prototipiranje.

3. Družine ESP32:

- ESP32-S series: (S2, S3) Naprednejši modeli z več GPIO pini in podporo za USB native.

- ESP32-C series: (C3, C6) RISC-V arhitektura, cenovno dostopnejši, usmerjeni v nizko porabo

## 1.3 Različice in posebnosti

Ni vsak ESP32 enak. Razlikujejo se po številu jeder, moči in dodatkih.

| Različica        | CPU (arhitektura)     | Frekvenca | RAM / PSRAM        | Povezljivost                              | Poraba (približno)        | Opombe / Posebnosti                                      |
|------------------|------------------------|-----------|--------------------|-------------------------------------------|---------------------------|----------------------------------------------------------|
| ESP32            | Xtensa Dual-core       | do 240 MHz| ~520 KB / da       | Wi-Fi 4, BT Classic + BLE                 | ~80–260 mA aktivno        | Zrel, široka podpora, idealen za avdio                   |
| ESP32-S2         | Xtensa Single-core     | do 240 MHz| ~320 KB / da       | Wi-Fi 4                                  | ~20–80 mA                 | USB OTG, brez BT, bolj varčen                           |
| ESP32-S3         | Xtensa Dual-core       | do 240 MHz| ~512 KB / da       | Wi-Fi 4, BLE 5                           | ~30–100 mA                | AI (vector instrukcije), USB, LCD podpora               |
| ESP32-C3         | RISC-V Single-core     | do 160 MHz| ~400 KB / ne       | Wi-Fi 4, BLE 5                           | ~20–50 mA                 | Poceni, low-power, zamenjava za ESP8266                 |
| ESP32-C6         | RISC-V Single-core     | do 160 MHz| ~512 KB / ne       | Wi-Fi 6, BLE 5, 802.15.4                 | ~20–60 mA                 | Matter, Thread, prihodnost pametnih domov               |
| ESP32-H2         | RISC-V Single-core     | do 96 MHz | ~320 KB / ne       | BLE 5, 802.15.4                          | zelo nizka                | Brez Wi-Fi, ultra low-power mesh                        |
| ESP32-P4         | RISC-V Dual-core       | do 400 MHz| več MB / da        | brez wireless                            | odvisno od uporabe        | High-performance MCU, HMI, edge AI                      |

![Slika 1 Primerjava esp32 različic](Slike/Poglavje1/PrimerjavaESP32.jpg)




# 2 Strojna arhitektura in električne specifikacije


Za uspešno načrtovanje elektronskih vezij z ESP32 je nujno razumevanje fizičnih omejitev čipa, napajalnih protokolov in narave signalov.

## 2.1  Napajanje in napetostni nivoji

ESP32 je zasnovan na sodobni polprevodniški tehnologiji, ki deluje pri nizkih napetostih.

- Delovna napetost ($V_{CC}$): Nominalna napetost znaša 3,3 V. Operativno območje je med 2,2 V in 3,6 V.
- Vhodna napetost na razvojni ploščici (VIN/5V): Razvojne ploščice (npr. DevKit V1) vključujejo LDO (Low-DropOut) regulator napetosti, ki omogoča napajanje preko USB vrat (5 V) ali zunanjega vira do 9 V.
- Poraba toka: Med aktivnim prenosom podatkov preko Wi-Fi lahko tok naraste do 240 mA, v načinu globokega spanja (Deep Sleep) pa pade pod 10 µA.


**OPOZORILO**: Vhodno-izhodni pini (GPIO) niso tolerantni na 5 V. Priklop 5 V signala neposredno na pin bo povzročil prebitje izolacijske plasti znotraj čipa in trajno uničenje krmilnika.

## 2.2 Digitalni vhodi in izhodi (GPIO)

Vsi GPIO (General Purpose Input/Output) pini so programabilni, vendar se električno razlikujejo glede na svojo notranjo strukturo.

1. Logični nivoji (CMOS)
ESP32 uporablja CMOS logiko, kjer so pragovi za preklop naslednji:
- $V_{IL}$ (Input Low): Napetost pod 0,8 V se zazna kot **logična 0**.
- $V_{IH}$ (Input High): Napetost nad 2,4 V se zazna kot **logična 1**.
- $V_{OL} / V_{OH}$: Izhodna napetost pri polni obremenitvi je blizu 0 V oziroma 3,3 V.



![Slika 1 Napetostni nivoji](Slike/Poglavje2/Napetostni_nivoji.jpg)

2. Tokovne omejitve

Vsak GPIO pin lahko varno zagotavlja ali sprejema omejeno količino toka. 
Privzeta nastavitev je običajno **12 mA,** največja dovoljena pa **40 mA**. Skupni tok vseh pinov ne sme preseči omejitev toplotne disipacije čipa.

## 3. Pomnilniška arhitektura (Internal & External Storage)

Arhitektura ESP32 temelji na Harvardski zasnovi, kar pomeni, da uporablja ločena vodila za ukaze (instrukcije) in podatke. To omogoča procesorju sočasen dostop do obeh vrst informacij, kar bistveno poveča hitrost izvajanja operacij. Pomnilnik v ESP32 je hierarhično razdeljen na notranji (vgrajen v čip) in zunanji (povezan preko vodil).

## 3.3 Pomnilniška arhitektura (Internal & External Storage)

Arhitektura ESP32 temelji na **Harvardski zasnovi**, kar pomeni, da uporablja ločena vodila za ukaze (instrukcije) in podatke. To omogoča procesorju sočasen dostop do obeh vrst informacij, kar bistveno poveča hitrost izvajanja operacij.

Pomnilnik v ESP32 je hierarhično razdeljen na:
- **notranji (vgrajen v čip)**
- **zunanji (povezan preko vodil)**

---

### 3.3.1 Notranji pomnilnik (Internal Memory)

Notranji pomnilnik je integriran neposredno v silicijevo strukturo SoC (System on a Chip), kar omogoča najhitrejši dostop brez zakasnitev.

#### Internal ROM (Read-Only Memory)

- Vsebuje kritično kodo, ki je zapisana med proizvodnjo in je ni mogoče spreminjati.
- V njem se nahaja **Bootloader**, ki upravlja z zagonom čipa in nalaganjem kode preko UART/USB.
- Vključuje osnovne knjižnice za nizko-nivojsko delovanje (npr. matematične funkcije in osnovni Wi-Fi sklad).

#### Internal SRAM (Static Random Access Memory)

- Primarni delovni pomnilnik med izvajanjem programa.
- Uporablja se za:
  - dinamične podatke
  - spremenljivke
  - sklad (*stack*)
  - kopico (*heap*)
- Najhitrejši dostop, vendar **ni obstojen** (podatki se izgubijo ob izklopu).

#### RTC Fast Memory

- Majhen pomnilnik za podatke, ki morajo ostati dostopni po **Deep Sleep**.
- Uporablja ga predvsem **ULP koprocesor** (Ultra Low Power).

#### RTC Slow Memory

- Podoben RTC Fast Memory, vendar optimiziran za **nižjo porabo energije**.
- Hrani spremenljivke za takojšnjo uporabo po prebujanju.

---

### 3.3.2 Zunanji pomnilnik (External Memory)

Ker notranji pomnilnik pogosto ne zadostuje za kompleksne aplikacije, ESP32 uporablja dodatne zunanje komponente.

#### External Flash (Programski pomnilnik)

- Povezan preko **SPI vodila**.
- Vsebuje:
  - programsko kodo (firmware)
  - slike
  - spletne strani
  - statične podatke
- **Non-volatile** (podatki ostanejo brez napajanja).
- Procesor uporablja **cache (predpomnilnik)** za hitrejši dostop.

#### External PSRAM (Pseudo-Static RAM)

- Prisoten pri nekaterih modulih (npr. WROVER).
- Uporaba:
  - obdelava slik
  - veliki bufferji
  - zahtevni strežniki
- Razširi omejen notranji SRAM.

#### eFuse (Electronic Fuses)

- Enosmerno zapisljiv pomnilnik.
- Uporablja se za:
  - MAC naslov
  - kalibracijo ADC
  - varnostne ključe
  - sistemske nastavitve
- Ko je bit zapisan, ga ni mogoče spremeniti.

---

### 3.3.3 Virtualni naslovni prostor in MMU

ESP32 uporablja **MMU (Memory Management Unit)** za preslikavo zunanjega pomnilnika v naslovni prostor procesorja.

To omogoča:
- dostop do Flash in PSRAM kot da sta notranji pomnilnik
- enostavnejše programiranje
- večjo fleksibilnost

Kljub temu podatki fizično še vedno potujejo preko **SPI vodila**, kar pomeni nekoliko večjo latenco kot pri notranjem pomnilniku.


![Silka1 Arhitektura procesorja](Slike/Poglavje3/Arhitektura_spomina.png)

## 3 Datotečni sistemi v Flash pomnilniku

Zunanji Flash pomnilnik ESP32 je razdeljen na več particij. Ena particija je namenjena izvajanju programa (App), ostale pa lahko uporabimo kot virtualni trdi disk. Za upravljanje teh particij uporabljamo namenske datotečne sisteme.

---

### 3.1 SPIFFS (SPI Flash File System)

SPIFFS je bil dolga leta standard za ESP8266 in ESP32.

- **Zasnova:** Prilagojen za majhne Flash čipe, ki ne podpirajo pravih imenikov (podmap).  
- **Omejitve:** 
  - Ni dejanske strukture map; poti, kot je `/data/config.txt`, obravnava le kot dolgo ime datoteke.  
  - Ni podpore za preverjanje napak pri pisanju.  
  - Počasnejši pri delu z velikimi particijami.  
- **Status:** Danes velja za opuščenega (*Deprecated*), nadomešča ga **LittleFS**.

---

### 3.2 LittleFS (LITTLE File System)

LittleFS je sodobnejši in robustnejši datotečni sistem, trenutno priporočljiv za nove projekte na ESP32.

- **Odpornost na izpad napajanja:** Ob nenadni izgubi napajanja med pisanjem ne pride do poškodbe celotnega datotečnega sistema (*corruption resilience*).  
- **Dinamično porazdeljevanje obrabe (Wear Leveling):** Omejeno število ciklov pisanja Flash čipa (~100.000), LittleFS enakomerno porazdeli pisanje po celotnem čipu.  
- **Hitrost:** Hitrejše branje in iskanje datotek kot SPIFFS.  
- **Podpora mapam:** Omogoča pravo hierarhično strukturo map.

---

### 3.3 Particijska tabela (Partition Table)

Da datotečni sistem deluje, moramo v razvojnem okolju (PlatformIO) določiti, kolikšen del zunanjega Flasha bo pripadal kodi in kolikšen datotekam. To določimo s particijsko tabelo (.csv).

**Primer standardne razdelitve 4 MB Flash pomnilnika:**

| Particija        | Velikost | Namen |
|-----------------|-----------|-------|
| nvs             | 20 KB     | Shranjevanje Wi-Fi gesel in kalibracijskih podatkov |
| otadata         | 8 KB      | Podatki za brezžično posodabljanje (OTA) |
| app0            | 1.2 MB    | Glavni program |
| app1            | 1.2 MB    | Rezervni prostor za OTA posodobitev |
| spiffs/littlefs | 1.5 MB    | Prostor za uporabniške datoteke |

---

### 3.4 Praktična uporaba v IoT sistemih

- **Spletni strežnik:** HTML in CSS datoteke (`index.html`, `style.css`) shranimo v LittleFS namesto v C++ spremenljivke.  
- **Datalogger:** Meritve senzorjev v realnem času zapisujemo v datoteko `log.csv`, ki jo kasneje prenesemo za analizo.  
- **Konfiguracija:** Shranjevanje nastavitev naprave (npr. ime Wi-Fi omrežja), ki jih uporabnik nastavi preko spletnega vmesnika.

![Slika 2 Flash spomin](<Slike/Poglavje3/Datotetični sistem.png>)

# 4. Razvojna okolja za ESP32

Za razvoj programske opreme (**firmware**) na platformi **ESP32** potrebujemo **integrirano razvojno okolje (IDE)**.  
V izobraževalne in profesionalne namene se poslužujemo dveh pristopov: enostavnejšega **Arduino IDE** in naprednejšega **VS Code s PlatformIO**.

---

## 4.1 Arduino IDE 2.x (Hitri razvoj in testiranje)

Novejša generacija Arduino IDE (2.0 in novejše) vključuje izboljšan **upravljalnik ploščic** in **serijski risalnik (Serial Plotter)**.

### Konfiguracija okolja

**Namestitev jedra (Core):**

1. V levi orodni vrstici izberite ikono **Boards Manager**.  
2. V iskalno polje vpišite `ESP32`.  
3. Poiščite paket **esp32 by Espressif Systems** in kliknite **Install**.  


![dodajanje esp32 board](<Slike/Poglavje4/dodajanje esp32 board.png>)

**Izbira strojne opreme:**

1. Povežite ESP32 z računalnikom preko USB kabla.  
2. V spustnem meniju na vrhu (**Select Board**) izberite ustrezna **COM vrata** in model ploščice (npr. **ESP32 DEVKIT**).

![select board](Slike/Poglavje4/Select_board.png)

![izbira plošče ter COM port](<Slike/Poglavje4/Izbira plosce + COM3.png>)
---

# 4.2.1 Namestitev in začetna konfiguracija VS Code

Preden začnemo z razvojem, moramo vzpostaviti **programsko verigo (toolchain)**. Postopek namestitve razdelimo na tri sklope: namestitev urejevalnika, namestitev okolja PlatformIO in priprava prvega projekta.

---

## 1. Namestitev Visual Studio Code (VS Code)

**VS Code** je odprtokodni urejevalnik kode podjetja Microsoft, ki služi kot osnova za razvoj.

1. Obiščite uradno stran: [code.visualstudio.com](https://code.visualstudio.com/).  
2. Prenesite namestitveno datoteko za svoj operacijski sistem (Windows, macOS ali Linux).  
3. **Pomembno:** Med namestitvijo na Windows označite možnosti:
   - `Add to PATH`  
   - `Open with Code`  

To olajša kasnejše delo z datotekami in ukazno vrstico.

---

## 2. Namestitev vtičnika PlatformIO IDE

**PlatformIO** deluje kot razširitev znotraj VS Code in avtomatizira upravljanje s strojno opremo ter knjižnicami.

1. Odprite VS Code.  
2. V levi orodni vrstici kliknite na ikono **Extensions** (kvadratki) ali pritisnite `Ctrl+Shift+X`.  
3. V iskalno polje vpišite **PlatformIO IDE**.  
4. Kliknite **Install**.  


![Platformio install](Slike/Poglavje4/Platformio_install.png)

> Po namestitvi se v spodnjem levem kotu pojavi ikona **PlatformIO Home** (hišica), v levi vrstici pa ikona **glave mravlje ali čebelice**.  
> Prva namestitev lahko traja nekaj minut, saj PlatformIO v ozadju prenaša potrebna prevajalna orodja in Python okolje.

---

## 3. Postavitev novega projekta (Project Wizard)

Ustvarjanje projekta preko **PlatformIO Wizard** zagotavlja pravilno nastavitev strojne opreme že pred prvo vrstico kode.

1. Kliknite na **PlatformIO Home** (ikona hišice).  
2. Izberite **+ Create New Project**.  
3. Izberite **+ New Project**.  
4. V čarovniku za projekte izpolnite naslednja polja:
   - **Name:** Ime projekta (izogibajte se šumnikom in presledkom, npr. `vaja1_blink`).  
   - **Board:** Vpišite `esp32dev` in izberite **Espressif ESP32 Dev Module**.  
   - **Framework:** Izberite **Arduino**.  
5. Kliknite **Finish**. Sistem bo avtomatsko generiral strukturo map in datoteko `platformio.ini`.

![Platformio_projekt_1](Slike/Poglavje4/Platformio_1.png)

![Platformio_projekt_2](Slike/Poglavje4/Platformi_projekt_2.png)
![Platformio_projekt_3](Slike/Poglavje4/Platformio_projekt_3.png)


---

## 4. Prva konfiguracija: platformio.ini

Po ustvarjanju projekta je priporočljivo dopolniti datoteko `platformio.ini` s parametri za **serijsko komunikacijo**, sicer bodo izpisi v terminalu neberljivi.

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino

; Parametri za pravilno delovanje terminala
monitor_speed = 115200
upload_speed = 921600

```

![Platformio_projekt_4](Slike/Poglavje4/Platformio_projekt_4.png)



<iframe src="https://withdiode.com/embed/16da7aba-a027-487e-b6e2-7a39b55cb888" style="width:100%; height:500px; border:1px solid rgba(0,0,0,0.1); border-radius: 0.5rem; overflow:hidden;" title="Arduino Uno Blink" allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking" sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts" ></iframe>
