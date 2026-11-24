# PROG - Programazioa eta Sentsorizazioa

**Modulua:** Programazioa eta IoT Sentsorizazioa  
**Helburua:** Mikrokontroladore/SOC plataformaren justifikazioa eta programazioa (4 sentsore baino gehiago), datuak WiFi bidez zerbitzarira bidaltzea, zerbitzari aldean datuak jasotzea (API/Backend) eta datu-basean gordetzea, eta web orrian datu horiek erakustea eta kudeatzea, dena GitHub bidez bertsionatuta.  
**Erreferentzia-rubrika:** IE2 (%2.0), IE3 (%10.0), IE4 (%20.0), IE5 (%10.0) -> Pisu Osoa: %42.00

---

## 1. Bertsio-Kontrola eta Git Biltegia (IE2)
*Rubrika Aurreratua (10):* Programa guztiak, script-ak eta bertsioak GitHub biltegian kudeatuta commit argiekin.

* **GitHub Biltegia:** `https://github.com/basterretxeahodei/STY_erronka2`
* **Iturburu-Fitxategien Antolaketa:**
  - `src/firmware/`: Mikrokontroladorearen C++/Arduino/MicroPython kodea.
  - `src/backend/`: Datuak jasotzeko API zerbitzaria (PHP / Python Flask / Node.js).
  - `src/frontend/`: Datuak kontsultatzeko eta bistaratzeko web interfazea.
  - `src/database/`: Datu-baseko eskemak eta migrazioak.

---

## 2. Plataforma Elektronikoa eta Mikrokontroladorearen Justifikazioa (IE4)
*Rubrika Aurreratua (10):* Mikrokontroladore edo SOC aukeraketa teknikoki bikain justifikatuta, 4 sentsore edo gehiagoren programazioa, eta WiFi bidezko hari gabeko komunikazioa.

### 2.1. Aukeratutako Mikrokontroladorea eta Justifikazio Teknikoa
* **Aukeratutako Plataforma:** **ESP32 NodeMCU (Xtensa Dual-Core 32-bit LX6)**.
* **Justifikazio Teknikoa:**
  1. **Konektibitate Integratua:** WiFi (802.11 b/g/n) eta Bluetooth BLE integratuak txip berean, kanpoko modulurik gabe kostua eta espazioa murriztuz.
  2. **GPIOn Aberastasuna:** ADC pin ugari (sentsore analogikoetarako) eta I2C/SPI bus euskarria (sentsore digitaletarako).
  3. **Eraginkortasun Energetikoa:** Deep Sleep moduak kontsumoa mA gutxitara murrizten du, IoT proiektuetarako aproposa.
  4. **Prozesatzeko Gaitasuna:** 240 MHz-ko abiadura datuak lokalean iragazi, zifratu eta JSON formatuan prestatzeko.

### 2.2. Sentsoreen Irakurketa Kodea (ESP32 - C++ / Arduino)
```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include <DHT.h>
#include <ArduinoJson.h>

const char* ssid = "Enpresa_WiFi";
const char* password = "WiFiPasahitza2026!";
const char* serverUrl = "http://192.168.1.20/api/ingurunea.php";

#define DHTPIN 4
#define DHTTYPE DHT22
#define LDRPIN 34
#define PIRPIN 27
#define GASPIN 35

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();
  pinMode(PIRPIN, INPUT);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) { delay(500); }
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    float temp = dht.readTemperature();
    float hum = dht.readHumidity;
    int argia = analogRead(LDRPIN);
    int mugimendua = digitalRead(PIRPIN);
    int gasa = analogRead(GASPIN);

    StaticJsonDocument<200> doc;
    doc["tenperatura"] = temp;
    doc["hezetasuna"] = hum;
    doc["argia"] = argia;
    doc["mugimendua"] = mugimendua;
    doc["gasa"] = gasa;

    String jsonStr;
    serializeJson(doc, jsonStr);

    HTTPClient http;
    http.begin(serverUrl);
    http.addHeader("Content-Type", "application/json");
    int httpResponseCode = http.POST(jsonStr);
    http.end();
  }
  delay(10000); // 10 segundoro bidali
}
```

---

## 3. Zerbitzari Aldeko Datu-Bilketa eta Datu-Basea (IE5)
*Rubrika Aurreratua (10):* Zerbitzari aldean datuak jasotzeko API programa sendoa (akatsik gabe) eta datu-baseko tauletan datuak egoki gordetzea.

### 3.1. Datu-Basearen Egitura (`sentsore_datuak`)
```sql
CREATE TABLE IF NOT EXISTS ingurumen_neurketak (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tenperatura DECIMAL(5,2) NOT NULL,
    hezetasuna DECIMAL(5,2) NOT NULL,
    argia INT NOT NULL,
    mugimendua TINYINT(1) NOT NULL,
    gasa INT NOT NULL,
    jasotze_data TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3.2. Datuak Jasotzeko Backend APIa (`api/ingurunea.php`)
```php
<?php
header("Content-Type: application/json; charset=UTF-8");
require_once '../config/db.php';

$edukia = file_get_contents("php://input");
$datuak = json_decode($edukia, true);

if ($datuak) {
    try {
        $stmt = $pdo->prepare("INSERT INTO ingurumen_neurketak 
            (tenperatura, hezetasuna, argia, mugimendua, gasa) 
            VALUES (:t, :h, :a, :m, :g)");
        $stmt->execute([
            ':t' => $datuak['tenperatura'],
            ':h' => $datuak['hezetasuna'],
            ':a' => $datuak['argia'],
            ':m' => $datuak['mugimendua'],
            ':g' => $datuak['gasa']
        ]);
        echo json_encode(["status" => "success", "message" => "Datuak ondo gorde dira"]);
    } catch (PDOException $e) {
        http_response_code(500);
        echo json_encode(["status" => "error", "message" => $e->getMessage()]);
    }
} else {
    http_response_code(400);
    echo json_encode(["status" => "error", "message" => "JSON baliogabea"]);
}
```

---

## 4. Web Ingurunea: Datuen Aurkezpena eta Tratamendua (IE3)
*Rubrika Aurreratua (10):* Enpresako web orrian sentsoreen datuak denbora errealean eta historikoan bistaratzea akatsik gabe.

* **Frontend bistaratzea (`panela.php`):**
  - Neurketa berrienak (tenperatura, hezetasuna, airearen kalitatea) widget ikusgarrietan bistaratu.
  - Alertak konfiguratu (adib. Tenperatura > 30ºC edo Gasa > atalase kritikoa bada gorriz margotu).
  - Grafiko interaktiboak (Chart.js erabiliz azken orduetako joera erakusteko).
* **Ebidentziak:**
> *(Gehitu hemen datu-baseko taularen erregistroen eta web paneleko grafikoen pantaila-argazkiak)*
