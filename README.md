# Proiect TSC - E-Book Reader
## Udățeanu Andreea-Laura

Acest proiect reprezinta un sistem embedded de tip **E-Book Reader**, bazat pe microcontrollerul **ESP32-C6**. Dispozitivul este capabil sa afiseze continut text pe un ecran **E-Paper**, oferind o solutie cu consum redus de energie, ideala pentru citirea offline a documentelor. Sistemul include, de asemenea, functionalitati precum citire de pe card SD, afisare dinamica, butoane pentru control si conectivitate extinsa prin interfata I2C.

# Diagrama bloc

![Diagrama Bloc](https://github.com/user-attachments/assets/6f1df5ae-9206-46c9-8b78-b8b4f937c67f)


# Functionalitati Hardware

Proiectul utilizeaza microcontrollerul **ESP32-C6-WROOM-1-N8**, un modul avansat cu suport pentru **Wi-Fi 6**, **Bluetooth Low Energy 5.0** si arhitectura **RISC-V**:

## Microcontrollerul ESP32-C6

- Modul avansat cu suport pentru **Wi-Fi 6**, **Bluetooth Low Energy 5.0** si arhitectura **RISC-V**
- **Interfete:** SPI, I2C, UART, USB, PWM, GPIO
	- **SPI (Serial Peripheral Interface):** este utilizat pentru comunicatia rapida cu dispozitive precum E-Paper Display-ul, memoria NOR Flash si cardul microSD.
	- **I2C (Inter-Integrated Circuit):** este o magistrala seriala bidirectionala cu doi pini (SDA si SCL), folosita pentru conectarea senzorului BME688, RTC-ului DS3231SN si monitorului de baterie MAX17048.
	- **UART (Universal Asynchronous Receiver/Transmitter):** interfata seriala folosita pentru comunicatie cu terminalul serial (TX/RX).
	- **GPIO (General Purpose Input/Output):** pini de uz general configurati ca intrare/iesire digitala – utilizati pentru butoane, controlul ecranului E-Ink, etc.

- **Caracteristici speciale:** ofera suport pentru moduri de economisire (deep sleep)
- **Conexiuni:** Interfata centrala – conecteaza toate modulele externe din cadrul proiectului(E-Paper, SD card, senzori, RTC, memorie NOR Flash, etc.)

## Alimentare si protectie

### MAX17048 – Monitorizare nivel de baterie Li-Po
- **Comunicare:** prin interfata I2C cu ESP32-C6 (SCL, SDA)
- **Iesire:** are la iesire un semnal ALERT pentru nivel critic baterie

### MCP73831 – Controller de incarcare a bateriei Li-Po
- **Incarcare:** utilizeaza o celula Li-Po prin USB (VBUS) → 4.2V
- **Curent de incarcare:** pana la 500 mA
- **LED-uri:** care indica starea de incarcare (CHG_LED)

### XC6220A331MR-G – Regulator de tensiune LDO
- Primeste la intrare 5V prin USB (VBUS), iar la iesire transmite o tensiune de 3.3V
- **Curent maxim:** 200 mA
- **Consum standby:** <1 µA

### Conector USB-C si Protectie ESD
- **Conector USB-C:** utilizat atat pentru alimentarea dispozitivului (VBUS) cat si pentru comunicatie USB cu microcontrollerul ESP32-C6.

- **Protectie ESD:** implementata pe liniile de date si alimentare folosind diode TVS dedicate:
USBLC6-2SC6Y – protejeaza liniile D+ si D−. PMF0501 – asigura protectie suplimentara pe linia de alimentare VBUS.
- **Utilizare:** permite incarcarea bateriei prin controllerul MCP73831, dar si programarea/diagnosticarea ESP32-C6 fara conexiuni suplimentare.

## Stocare

### SD Card
- **Functie:** citire/scriere a fisierelor E-Book din cardul microSD
- **Interfata:** SPI partajata cu E-Paper
- **Semnale:** SS_SD, MOSI, MISO, SCK
- **Slot:** 112A-TAAR-R03

### W25Q512JVEIQ – Memorie NOR Flash externa 64MB
- **Capacitate:** 64 MB
- **Interfata:** conectata la magistrala SPI (nu partajeaza cu E-Paper)
- **Semnale:** FLASH_CS, MOSI, MISO, SCK
- **Functie:** stocare fonturi, imagini, firmware, configurari

## Afisaj E-Paper

- Interfete SPI si GPIO dedicate, avand semnale precum: CS (EPD_CS), DC (EPD_DC), RST (EPD_RST), BUSY (EPD_BUSY), MOSI, SCK  
## **Header special EPD** cu 20 pini
## **Controlul alimentarii EPD:** 
Permite microcontrollerului sa activeze sau sa dezactiveze alimentarea modulului E-Paper pentru a economisi energie, avand la iesire o tensiune de 3V3
## **Selectare mod:** 
Comutator tip jumper pentru a selecta tipul de display (circuit EPD driver dedicat)
- Consum 0 in standby, ideal pentru afisaj static

## Senzor de Mediu BME688
- **Functionalitati:** temperatura, umiditate, presiune, calitate aer (VOC)
- **Interfata:** I2C
- **Pini:** SDA, SCL legati la magistrala I2C comuna
- **Tensiune de alimentare:** 3.3V

## Modul RTC – DS3231SN
- **Functie:** surprinde timpul real cu precizie ±2ppm, avand suport deep sleep
- **Conexiune:** I2C – partajeaza pinii SDA/SCL cu senzorul BME688 si cu monitorizatorul de baterie MAX17048
- **Iesiri:** semnal 32kHz si semnal de intrerupere (INT_RTC)
- **Alimentare:** 3.3V, avand si backup intern (capacitor)

## Functii de control, reset si protectie

### Butoane de Control
- **BOOT / RESET si IO/CHANGE personalizat:** conectate pe GPIO cu retele RC
- **Functii:** BOOT: initializare microcontroller in modul bootloader, RESET: resetare generala a sistemului si IO/CHANGE: buton suplimentare pentru interactiuni custom de tipul schimbarii paginii, a meniului, etc.

### BD5229G-TR – Voltage Supervisor
- **Rol:** detecteaza scaderile de tensiune (brown-out) sub un anumit prag critic si genereaza automat un semnal de rest
- **Asigura o pornirea sigura a sistemului

### Protectii ESD si Interfete Extensibile

- **Linii protejate:** toate magistralele sensibile – USB, SPI, I2C si GPIO – sunt protejate impotriva descarcarilor electrostatice (ESD) prin diode (ex: **USBLC6-2SC6Y**, **PGB1010603MR**, **MBR0530**).
- **Conector Qwiic / Stemma QT:** ofera o interfata I2C standard (3.3V, GND, SDA, SCL) pentru extinderea rapida a sistemului cu senzori si periferice de la SparkFun sau Adafruit, fara necesitatea lipirii.
- **Paduri de test:** sunt incluse pe PCB pentru depanare usoara, verificari de semnal si programare.

### Consum de energie estimativ

| Componenta                | Consum aproximativ   |
|:--------------------------|:---------------------|
| ESP32-C6                  | 120–150 mA           |
| E-Paper Display           | 20–40 mA             |
| Senzor BME688             | 1–2 mA               |
| RTC DS3231SN              | 200 µA               |
| SD Card                   | 40–100 mA            |
| MAX17048                  | 50 µA                |
| NOR Flash (W25Q512JVEIQ)  | 20 mA                |
| Total sistem - mod activ  | aprob. 300 mA        |
| Total sistem - deep sleep | max. 10 µA           |

Daca nu sunt dezactivate toate modulele fizic (cu MOSFET sau power gate), deep sleep-ul poate ajunge usor la 1–2 mA, deci <10 µA este ideal si atins doar cu management riguros al energiei.


### Alocarea pinilor

## E-Paper Display

| Pin ESP32-C6 | Functie     | Utilizare                               |
|--------------|-------------|------------------------------------------|
| IO3  (26)    | EPD_BUSY    | Semnalizează dacă ecranul este activ    |
| IO5  (5)     | EPD_DC      | Schimba modul intre date și comanda     |
| IO6  (6)     | SCK         | Linie de ceas SPI - transfer de date    |
| IO7  (7)     | MOSI        | Trimitere date către ecran prin SPI     |
| IO10 (11)    | EPD_CS      | Activare comunicație cu modulul EPD     |
| IO23 (21)    | EPD_RST     | Resetare ecran                          |
| IO20 (18)    | EPD_3V3_C   | Control tensiune de alimentare 3V3 EPD  |

## Magistrala I2C (MAX17048, RTC DS3231, BME688)

| Pin ESP32-C6 | Functie     | Utilizare                               |
|--------------|-------------|------------------------------------------|
| IO21 (19)    | SDA         | Linie de transmitere date I2C           |
| IO22 (20)    | SCL         | Linie de ceas I2C                       |
| IO19 (17)    | I2C_PW      | Pornire/oprire senzori I2C              |

## Modul RTC DS3231SN

| Pin ESP32-C6 | Functie     | Utilizare                               |
|--------------|-------------|------------------------------------------|
| IO0 (8)      | INT_RTC     | Semnal de întrerupere RTC               |
| IO1 (9)      | 32KHz       | Iesire ceas precis                      |
| IO18 (16)    | RTC_RST     | Resetare hardware al modulului RTC      |

## USB
| Pin ESP32-C6 | Functie     | Utilizare                               |
|--------------|-------------|------------------------------------------|
| IO12 (13)    | USB_D-      | Linia negativa de comunicatie USB       |
| IO13 (14)    | USB_D+      | Linia pozitiva de comunicatie USB       |

## Seriala UART

| Pin ESP32-C6 | Functie     | Utilizare                               |
|--------------|-------------|------------------------------------------|
| IO16 (25)    | TX          | Transmisie UART                         |
| IO17 (24)    | RX          | Receptie UART                           |

## SD Card

| Pin ESP32-C6 | Functie     | Utilizare                               |
|--------------|-------------|------------------------------------------|
| IO2 (27)     | MISO        | Receptie date                           |
| IO4 (4)      | SS_SD       | Selectare modul SD                      |
| IO6 (6)      | SCK         | Ceas SPI                                |
| IO7 (7)      | MOSI        | Transmisie date                         |

## Memorie Externa NOR Flash 64MB

| Pin ESP32-C6 | Functie     | Utilizare                               |
|--------------|-------------|------------------------------------------|
| IO11 (12)    | FLASH_CS    | Selectare chip NOR                      |
| IO2  (27)    | MISO        | Receptie date                           |
| IO6  (6)     | SCK         | Ceas SPI                                |
| IO7  (7)     | MOSI        | Transmisie date                         |

## Butoane

| Pin ESP32-C6 | Functie     | Utilizare                               |
|--------------|-------------|------------------------------------------|
| EN   (3)     | RESET       | Resetare microcontroller                |
| IO8  (10)    | IO/BOOT     | Buton BOOT (în modul de programare)     |
| IO15 (23)    | IO/CHANGE   | Buton custom pentru diverse actiuni     |

## Alimentare si Masă

| Pin ESP32-C6 | Functie     | Utilizare                               |
|--------------|-------------|------------------------------------------|
| Pin 2        | 3V3         | Alimentare principala la 3.3V           |
| Pin 1        | GND         | Conectare la masa                       |


## Bill of Materials

| Componenta | Link Cumpărare | Datasheet |
|------------|-----------------|-----------|
| Condensatoare Tantalum | [Buy](https://ro.mouser.com/ProductDetail/KYOCERA-AVX/TAJW107M010RNJ?qs=Wtp%252Bf%2FAeVqIH8v1VxV%252B1Rg%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/40/TAJ-3165264.pdf) |
| Rezistente R0402 | [Buy](https://store.comet.srl.ro/Search/?keywords=r0402#eyJpcHAiOiIyMCIsIm0iOiIiLCJrIjoicjA0MDIifQ%3D%3D) | [Datasheet](https://www.yageo.com/upload/media/product/products/datasheet/rchip/PYu-RC_Group_51_RoHS_L_12.pdf) |
| Condensatoare C0402 | [Buy](https://store.comet.srl.ro/CatalogueFarnell/Search/?keywords=c0402#eyJpcHAiOiIyMCIsImsiOiJjMDQwMiJ9) | [Datasheet](https://componentsearchengine.com/Datasheets/2/CC0402MRX5R5BB106.pdf) |
| 112A-TAAR-R03 | [Buy](https://store.comet.srl.ro/Catalogue/Product/43497/) | [Datasheet](https://www.snapeda.com/parts/112A-TAAR-R03/Attend/datasheet/) |
| SD0805S020S1R0 | [Buy](https://ro.mouser.com/ProductDetail/KYOCERA-AVX/SD0805S020S1R0?qs=jCA%252BPfw4LHbpkAoSnwrdjw%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/40/schottky-3165252.pdf) |
| MBR0530 | [Buy](https://ro.mouser.com/ProductDetail/onsemi-Fairchild/MBR0530?qs=VOMQJJE%252BBNniwJKcE3T43Q==) | [Datasheet](https://ro.mouser.com/datasheet/2/308/MBR0530_D-1810985.pdf) |
| CHG_LED | [Buy](https://ro.mouser.com/ProductDetail/Kingbright/KP-1608SURCK?qs=2JU0tDl2GZ3FuyEWfBV1%2Fg==) | [Datasheet](https://www.snapeda.com/parts/KP-1608SURCK/Kingbright/datasheet/) |
| USBLC6-2SC6Y | [Buy](https://ro.mouser.com/ProductDetail/STMicroelectronics/USBLC6-2SC6Y?qs=gNDSiZmRJS%2FOgDexvXkdow%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/389/usblc6_2sc6y-1852505.pdf) 
| PGB1010603MR | [Buy](https://www.digikey.com/en/products/detail/littelfuse-inc/PGB1010603MR/715755) | [Datasheet](https://www.littelfuse.com/assetdocs/pulseguard-esd-suppressors-pgb1-datasheet?assetguid=8a337998-d54d-466b-be4e-dc5bcd1f9321) |
| ESP32-C6-WROOM-1-N8 | [Buy](https://ro.mouser.com/ProductDetail/Espressif-Systems/ESP32-C6-WROOM-1-N8?qs=8Wlm6%252BaMh8ST02Gmwp74cw%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/891/Espressif_ESP32_C6_WROOM_1__Datasheet_V0_1_PRELIMI-3239987.pdf) |
| DS3231SN | [Buy](https://store.comet.srl.ro/Catalogue/Product/47690/) | [Datasheet](https://ro.mouser.com/datasheet/2/609/DS3231-3421123.pdf) |
| BME680 | [Buy](https://ro.mouser.com/ProductDetail/Bosch-Sensortec/BME688?qs=IS%252B4QmGtzzqQoVDscqwx3A%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/783/bst_bme688_fl000-2307034.pdf) |
| MAX17048 | [Buy](https://ro.mouser.com/ProductDetail/Analog-Devices-Maxim-Integrated/MAX17048G%2bT10?qs=D7PJwyCwLAoGnnn8jEPRBQ%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/609/MAX17048_MAX17049-3469099.pdf) |
| MCP73831 | [Buy](https://ro.mouser.com/ProductDetail/Microchip-Technology/MCP73831T-2ATI-OT?qs=yUQqVecv4qsZbioEUu%252B83g%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/268/MCP73831_Family_Data_Sheet_DS20001984H-3441711.pdf) |
| BD5229G-TR | [Buy](https://www.digikey.com/en/products/detail/rohm-semiconductor/BD5229G-TR/3663792) | [Datasheet](https://fscdn.rohm.com/en/products/databook/datasheet/ic/power/voltage_detector/bd52xxg-e.pdf) |
| W25Q512JVEIQ | [Buy](https://ro.mouser.com/ProductDetail/Winbond/W25Q512JVEIQ?qs=l7cgNqFNU1jw6svr3at6tA%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/949/Winbond_W25Q512JV_Datasheet-3240039.pdf) |
| XC6220A331MR-G | [Buy](https://ro.mouser.com/ProductDetail/Torex-Semiconductor/XC6220A331MR-G?qs=AsjdqWjXhJ8ZSWznL1J0gg%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/760/xc6220-3371556.pdf) |
| USB4110-GF-A | [Buy](https://ro.mouser.com/ProductDetail/GCT/USB4110-GF-A?qs=KUoIvG%2F9IlYiZvIXQjyJeA%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/837/GCT_USB4110_Product_Drawing___20k_cycles-3455479.pdf) |
| FH34SRJ | [Buy](https://ro.mouser.com/ProductDetail/Hirose-Connector/FH34SRJ-24S-0.5SH99?qs=vcbW%252B4%252BSTIpKBl5ap9J8Fw%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/185/FH34SRJ_24S_0_5SH_99__CL0580_1255_6_99_2DDrawing_0-1615044.pdf) |
| QWIIC_RIGHT_ANGLE | [Buy](https://ro.mouser.com/ProductDetail/SparkFun/PRT-14417?qs=wd5RIQLrsJhgdz%2FpmZ%2F3GQ==) | [Datasheet](https://ro.mouser.com/datasheet/2/813/Qwiic_Connector_Datasheet-1223982.pdf) |
| Bobine 744043680 | [Buy](https://ro.mouser.com/ProductDetail/Wurth-Elektronik/744043680?qs=PGXP4M47uW6VkZq%252BkzjrHA%3D%3D) | [Datasheet](https://www.we-online.com/components/products/datasheet/744043680.pdf) |
| Si1308EDL-T1-GE3 | [Buy](https://www.digikey.com/en/products/detail/vishay-siliconix/SI1308EDL-T1-GE3/4876435) | [Datasheet](https://www.vishay.com/docs/63399/si1308edl.pdf) |
| PFMF.050.1 | [Buy](https://ro.mouser.com/ProductDetail/EPCOS-TDK/B72520T0350K062?qs=dEfas%2FXlABIszF52uu7vrg%3D%3D) | [Datasheet](https://www.tdk-electronics.tdk.com/inf/75/db/CTVS_14/Surge_protection_series.pdf) |
| DMG2305UX-7 | [Buy](https://www.digikey.com/en/products/detail/diodes-incorporated/DMG2305UX-7/4340666) | [Datasheet](https://www.diodes.com/assets/Datasheets/DMG2305UX.pdf) |
| SJ (Solder Jumpers) | [Buy](https://grabcad.com/library/solder-jumpers-1) | [Datasheet](https://grabcad.com/library/solder-jumpers-1) |
| Butoane | [Buy](https://industry.panasonic.com/global/en/products/control/switch/light-touch/number/evqpuj02k) | [Datasheet](https://industry.panasonic.com/global/en/downloads?tab=catalog&small_g_cd=203&part_no=EVQPUJ02K) |
| CPH3225A | [Buy](https://www.digikey.com/en/products/detail/seiko-instruments/CPH3225A/8692444) | [Datasheet](https://mm.digikey.com/Volume0/opasdata/d220001/medias/docus/6537/rev05-CPHCPM.pdf) |

## Concluzii si observatii privind realizarea designului hardware
- In realizarea designului am urmarit respectarea regulilor impuse de ERC si DRC specifice. Amplasarea componentelor a fost facuta tinand cont atat de documentatiile tehnice oferite (dimensiuni si recomadari de plasare a componentelor), cat si de considerente legate de rutare si functionare optima a dispozitivului.
- Pentru evitarea erorilor in cadrul PCB-ului, am ajustat dimensiunile unor footprint-uri si am corectat pad-urile pentru a indeplini regulile de buna functionare. Am folosit regulile din checklist-ul de proiectare (inclusiv asezarea condensatorilor de decuplare plasati cat mai aproape de pinii VCC ai circuitelor integrate).
- Am distribuit traseele atat pe stratul top, cat si pe stratul bottom, incercand sa obtin cat mai putine vias-uri pe traseele de alimentare, pentru a reduce posibilitatea unei caderea de tensiune. Am rutat traseele de alimentare cu o latime de 0.3 mm, traseele de semnal avand latime de 0.15 mm conform cerintelor.
- Test pad-urile au fost amplasate cat mai eficient in zone relevante, la marginea placii: langa Header-ul E-Paper-ului si langa microcontroller.
- Am orientat antena modulului ESP32-C6 spre exteriorul placii, iar sub aceasta a fost lasata o zona decupata, conform recomandarilor pentru functionare optima a conexiunii wireless.
- Am pozitionat componentele cat mai bine pentru a putea fi integrate in carcasa proiectata, urmarind atat potrivirea mecanica, cat si o utilizare eficienta a spatiului disponibil. Am amplasat bateria in zona libera ramasa, astfel incat sa nu se intersecteze cu alte componente.
- Am tinut cont de amplasarea elementelor in functie de conectorul USB-C pentru asigurarea compatibilitatii cu carcasa. Butoanele au fost plasate cat mai eficient pentru a permite accesul la acestea si integrarea cu carcasa.
- In ansamblu, designul rezultat respecta cerintele functionale si mecanice, existand un echilibru intre robustete si eficienta.

