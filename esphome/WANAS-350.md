# WANAS 350 → Home Assistant (ESPHome, Modbus RTU)

Konfig: [`wanas-350.yaml`](wanas-350.yaml)

## Hardver

- **Waveshare ESP32-S3-RS485-CAN** (DIN-sínes, izolált RS485). A CAN részt nem használjuk.
- **Táp:** 7–36 V DC a csavaros sorkapcson (pl. 12 V-os DIN-sínes táp), vagy 5 V USB-C-n.
  A kettőt **egyszerre ne** kösd rá, a Waveshare szerint ez károsíthatja a modult.
- **Kábel:** 2 vagy 3 eres, lehetőleg sodrott érpár (pl. egy UTP-kábel egy érpárja) a WANAS
  vezérlőpaneljéig.

## A meglévő 4 eres kábel

A WANAS vezérlőpaneljén a 4 eres kábel a **DISP (Wyświetlacz)** sorkapocsra megy:
`GND`, `+`, `B`, `A`. Ez a fali kijelző tápja és saját buszrendszere. **Ehhez ne nyúlj, és ne
kösd rá az ESP-t.** A kijelző ezen a buszon kommunikál, egy második eszköz összeakadna vele.

Az ESP a panel **MODBUS 2** sorkapcsára megy (a gyártó ezt javasolja). A modult ezért a gép
mellé kell tenni (padlás, gépház). Ott kell neki táp és Wi-Fi-lefedettség. A másik lehetőség,
hogy a modul lent marad, és külön kábelt húzol fel a MODBUS 2-höz. Az RS485 a hosszú kábelt is
bírja.

## Bekötés

A WANAS vezérlőpanelen a sorkapcsok sorrendje: DISP (GND, +, B, A), Modbus 1 (GND, B, A, GND),
Modbus 2 (B, A), TECH RS (RX, TX, +, GND).

| WANAS panel        | Waveshare RS485 |
|--------------------|-----------------|
| MODBUS 2 – **A**   | **A**           |
| MODBUS 2 – **B**   | **B**           |
| GND (a Modbus 2 melletti) | GND (ajánlott) |

- A Waveshare 120 Ω-os lezáró jumpere néhány méteres kábelnél maradhat kikapcsolva.
  Hosszú kábelnél kapcsold be.
- A WANAS leírása szerint **ha nincs kommunikáció, vedd le a jumpert** a MODBUS 2 / TECH RS
  csatlakozó mellett. A TECH RS-re a WANAS Mobile (W-15/W-19) internetes modul csatlakozik,
  ha van ilyened.
- Ha így sincs kommunikáció, cseréld fel az A és B vezetéket.

## WANAS beállítások

Gyári beállítás: **19200 baud, E-8-1 (páros paritás), cím: 1**. A konfig is ezt használja.
Ha a telepítő átállította, a `uart` és a `modbus_address` értékeit igazítsd hozzá.
A firmware-verziót a kijelzőn az *Informacje o programie* menüpont mutatja.

## Telepítés

1. Home Assistantban telepítsd az **ESPHome Device Builder** bővítményt.
2. Készíts új eszközt, és cseréld le a tartalmát a `wanas-350.yaml` tartalmára.
3. A `secrets.yaml`-ba (ESPHome → *Secrets*) írd be:
   ```yaml
   wifi_ssid: "..."
   wifi_password: "..."
   wanas_api_key: "..."      # pl.: openssl rand -base64 32
   wanas_ap_password: "..."  # tartalék hozzáférési pont jelszava
   ```
4. Az első feltöltés USB-C-n menjen (közben a sorkapocsról vedd le a tápot). Utána már
   Wi-Fi-n is frissíthető.
5. Home Assistantban az ESPHome integráció automatikusan megtalálja az eszközt.

## Mit tud

- **Szenzorok:** külső, befújt, távozó és helyiség hőmérséklet, szűrőcseréig hátralévő napok,
  aktuális befúvás/elszívás fokozat, hibák szövegesen. Ha vannak opcionális pára- és
  CO2-érzékelőid, azok alapból le vannak tiltva, a HA-ban engedélyezheted őket.
- **Állapotok:** bypass nyitva, fagyvédelem, előfűtő, szabadság mód, digitális bemenetek
  (I./III. fokozat, konyhai elszívó, tűzjelző), szűrőcsere szükséges, hiba.
- **Vezérlés:**
  - *Bypass engedélyezve* kapcsoló.
  - *Party mód:* III. fokozat; a hossza a `party_perc` substitutionnel állítható, alapból 120 perc.
  - *Szabadság mód:* I. fokozat; alapból 7 nap.
  - *Kandalló mód:* 180 s.
  - A fenti módok hossza külön is beállítható szám mezőkkel.
- **Heti program:** a *Program: szerkesztett nap* listában válaszd ki a napot. Utána a zónák
  vége, fokozata és komforthőmérséklete arra a napra vonatkozik.
- **Óra szinkronizálása** gomb: a Home Assistant idejét beírja a WANAS-ba.

## Korlátok

- A WANAS leírása szerint az aktuális fokozat (2–3. regiszter) **csak olvasható**. Közvetlen
  fokozatválasztó ezért nincs. A fokozatot a Party (III.) és a Szabadság (I.) móddal vagy a
  heti programmal lehet állítani.
- A WANAS 350 nem XF típus. A légmennyiség (m³/h) szenzorok ezért 0-t mutatnak, és alapból le
  vannak tiltva. A fokozatok teljesítménye százalékban állítható.
- Az óraszinkron formátuma (`óra<<8 | perc`) a leírás 12:55-ös példáján alapul. A leírás egy
  másik táblázata ennek ellentmond. Az első használat után nézd meg a kijelzőn, hogy jó-e az
  idő.
