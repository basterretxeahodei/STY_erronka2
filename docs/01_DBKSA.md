# DBKSA - Datu-Baseak Kudeatzeko Sistemak

**Modulua:** Datu-Baseak Kudeatzeko Sistemen Administrazioa  
**Helburua:** Datu-baseen sarbide-kontrola, segurtasuna, baimenen kudeaketa eta kudeatzailea administratzeko atazen automatizazioa (funtzioak, prozedurak eta triggerrak).  
**Erreferentzia-rubrika:** RA3 (%17.50) eta RA4 (%30.00) -> Pisu Osoa: %47.50

---

## 1. Sarbide Kontrola eta Pribilegioak (RA3)

### 1.1. Ikuspegi Pertsonalizatuak (Views)
*Rubrika Aurreratua (10):* Ikuspegi kopuru minimoa sortu eta dokumentatu da. Konfigurazioa eta helburuak zehaztasunez azalduta.

#### Ikuspegi 1: [Izenburua / Adib. `v_erabiltzaile_publikoa`]
* **Helburua eta eginkizuna:** Datu sentikorrak (pasahitzak, NAN, etab.) ezkutatzea eta kontsulta arruntetarako erraztasuna ematea.
* **SQL Definizioa:**
```sql
CREATE OR REPLACE VIEW v_erabiltzaile_publikoa AS
SELECT id, erabiltzaile_izena, rola, sortze_data
FROM erabiltzaileak
WHERE egoera = 'aktibo';
```
* **Funtzionamendu-froga:**
```sql
SELECT * FROM v_erabiltzaile_publikoa;
```
* **Ebidentzia / Pantaila-argazkia:**
> *(Gehitu hemen ikuspegiaren exekuzioaren pantaila-argazkia)*

#### Ikuspegi 2: [Izenburua / Adib. `v_iruzkinak_laburpena`]
* **Helburua eta eginkizuna:** Iruzkinen eta erabiltzaileen arteko erlazioa prestatzea estatistiketarako edo moderaziorako.
* **SQL Definizioa:**
```sql
-- Sartu hemen bigarren ikuspegiaren SQL kodea
```
* **Ebidentzia / Pantaila-argazkia:**
> *(Gehitu hemen exekuzioaren pantaila-argazkia)*

---

### 1.2. Erabiltzaileak, Taldeak/Rolak eta Baimenak
*Rubrika Aurreratua (10):* Erabiltzaile kopuru eta mota balekoa sortu, pribilegioak bikain definitu, taldeak/rolak sortu eta dokumentazio hedatua erantsi.

#### Talde/Rol Egitura
| Rola / Taldea | Deskribapena | Baimen Espezifikoak (GRANT) |
| :--- | :--- | :--- |
| `rol_administratzailea` | Kudeaketa orokorra | DDL eta DML osoa datu-base guztian |
| `rol_teknikaria` | Datuen kudeaketa | SELECT, INSERT, UPDATE tauletan |
| `rol_irakurlea` | Irakurketa hutsa | SELECT bakarrik baimendutako ikuspegietan |

#### Erabiltzaileen Sortzea eta Baimenen Esleipena
```sql
-- Rolak sortu
CREATE ROLE rol_irakurlea;
CREATE ROLE rol_teknikaria;

-- Baimenak esleitu rolei
GRANT SELECT ON v_erabiltzaile_publikoa TO rol_irakurlea;
GRANT SELECT, INSERT, UPDATE ON iruzkinak TO rol_teknikaria;

-- Erabiltzaileak sortu eta rolak esleitu
CREATE USER 'irakurle1'@'%' IDENTIFIED BY 'PasahitzaSegurua123!';
GRANT rol_irakurlea TO 'irakurle1'@'%';
SET DEFAULT ROLE rol_irakurlea TO 'irakurle1'@'%';

CREATE USER 'teknikari1'@'%' IDENTIFIED BY 'BestePasahitzSeguruBat456!';
GRANT rol_teknikaria TO 'teknikari1'@'%';
SET DEFAULT ROLE rol_teknikaria TO 'teknikari1'@'%';

FLUSH PRIVILEGES;
```

* **Ebidentzia / Pantaila-argazkia:**
> *(Gehitu hemen erabiltzaileak eta `SHOW GRANTS` komandoaren exekuzioak)*

---

### 1.3. Zerbitzariari Urruneko Atzipena
*Rubrika Aurreratua (10):* Enpresako beste ekipoetatik DBKS atzitzen da eta zeregin honetarako erabiltzaile espezifiko bat gaitu da.

* **DBKS Konfigurazioa (Adib. MariaDB / MySQL `my.cnf` edo PostgreSQL `pg_hba.conf`):**
```ini
# /etc/mysql/mariadb.conf.d/50-server.cnf
bind-address = 0.0.0.0
```
* **Urruneko erabiltzaile espezifikoa:**
```sql
CREATE USER 'remote_app'@'192.168.%.%' IDENTIFIED BY 'UrrunekoKonexioa2026!';
GRANT SELECT, INSERT, UPDATE, DELETE ON enpresa_db.* TO 'remote_app'@'192.168.%.%';
FLUSH PRIVILEGES;
```
* **Beste ekipo batetik egindako konexio-froga:**
```bash
mysql -h 192.168.1.100 -u remote_app -p enpresa_db
```
* **Ebidentzia / Pantaila-argazkia:**
> *(Gehitu hemen urruneko konexio arrakastatsuaren pantaila-argazkia)*

---

## 2. Ataza Administratiboen Automatizazioa (RA4)

### 2.1. Funtzioak (Functions)
*Rubrika Aurreratua (10):* Helburu zehatz baterako funtzio baliagarri bat baino gehiago sortu dira, modu egokian erabili eta dokumentatu dira.

#### Funtzioa 1: [Adib. `fnt_zenbatu_erabiltzaile_iruzkinak`]
* **Deskribapena eta logika:** Erabiltzaile baten IDa jasota, egindako iruzkin kopurua itzultzen du.
```sql
DELIMITER $$
CREATE FUNCTION fnt_zenbatu_erabiltzaile_iruzkinak(p_erabiltzaile_id INT)
RETURNS INT
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE total INT;
    SELECT COUNT(*) INTO total FROM iruzkinak WHERE erabiltzaile_id = p_erabiltzaile_id;
    RETURN total;
END$$
DELIMITER ;
```
* **Erabilera Adibidea:**
```sql
SELECT erabiltzaile_izena, fnt_zenbatu_erabiltzaile_iruzkinak(id) AS guztira
FROM erabiltzaileak;
```

#### Funtzioa 2: [Adib. `fnt_egiaztatu_pasahitza_segurua`]
* **Deskribapena eta logika:** Pasahitzak gutxieneko luzera eta segurtasun baldintzak betetzen dituela egiaztatzen du (TRUE/FALSE).
```sql
-- Sartu bigarren funtzioaren SQL kodea eta erabilera
```

* **Ebidentzia / Pantaila-argazkia:**
> *(Gehitu hemen bi funtzioen sortze eta deien exekuzioak)*

---

### 2.2. Prozedurak (Stored Procedures)
*Rubrika Aurreratua (10):* Helburu zehatz baterako prozedura baliagarri bat baino gehiago sortu dira, modu egokian erabili eta dokumentatu dira.

#### Prozedura 1: [Adib. `sp_erabiltzaile_berria_erregistratu`]
* **Deskribapena:** Erabiltzaile berri bat sortzen du transakzio seguru batean, auditoretza erregistro bat gehituz.
```sql
DELIMITER $$
CREATE PROCEDURE sp_erabiltzaile_berria_erregistratu(
    IN p_izena VARCHAR(50),
    IN p_email VARCHAR(100),
    IN p_pasahitza VARCHAR(255),
    OUT p_emaitza VARCHAR(100)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_emaitza = 'ERROREA: Ezin izan da erabiltzailea sortu.';
    END;

    START TRANSACTION;
        INSERT INTO erabiltzaileak (erabiltzaile_izena, email, pasahitza_hash, sortze_data)
        VALUES (p_izena, p_email, SHA2(p_pasahitza, 256), NOW());

        INSERT INTO audit_log (ekintza, deskribapena, data)
        VALUES ('ERABILTZAILE_SORTUA', CONCAT('Erabiltzailea: ', p_izena), NOW());
    COMMIT;
    SET p_emaitza = 'ARRAKASTA: Erabiltzailea ondo sortu da.';
END$$
DELIMITER ;
```
* **Erabilera Adibidea:**
```sql
CALL sp_erabiltzaile_berria_erregistratu('proba_user', 'proba@enpresa.eus', 'Pasahitz123!', @msg);
SELECT @msg;
```

#### Prozedura 2: [Adib. `sp_iruzkin_zaharrak_artxibatu`]
* **Deskribapena:** Data jakin bat baino zaharragoak diren iruzkinak artxibo taula batera mugitzen ditu automatikoki.
```sql
-- Sartu bigarren prozeduraren SQL kodea eta deia
```

* **Ebidentzia / Pantaila-argazkia:**
> *(Gehitu hemen bi prozeduren sortze eta deien emaitzak)*

---

### 2.3. Triggerrak (Triggers)
*Rubrika Aurreratua (10):* Helburu zehatz baterako trigger baliagarri bat baino gehiago sortu dira, modu egokian erabili eta dokumentatu dira.

#### Trigger 1: [Adib. `trg_audit_iruzkinak_delete`]
* **Deskribapena:** Iruzkin bat ezabatzen den bakoitzean, nork eta noiz ezabatu duen auditoretza taula batean (`audit_borratuak`) automatikoki gordetzen du.
```sql
DELIMITER $$
CREATE TRIGGER trg_audit_iruzkinak_delete
AFTER DELETE ON iruzkinak
FOR EACH ROW
BEGIN
    INSERT INTO audit_borratuak (iruzkin_id, edukia, egilea, ezabatze_data, ezabatzailea_user)
    VALUES (OLD.id, OLD.mezua, OLD.egilea, NOW(), USER());
END$$
DELIMITER ;
```
* **Funtzionamendu-froga:**
```sql
DELETE FROM iruzkinak WHERE id = 1;
SELECT * FROM audit_borratuak;
```

#### Trigger 2: [Adib. `trg_eguneratu_azken_aldaketa`]
* **Deskribapena:** Erregistro bat eguneratzen den bakoitzean `eguneratze_data` automatikoki uneko ordura pasatzen du (BEFORE UPDATE).
```sql
DELIMITER $$
CREATE TRIGGER trg_eguneratu_azken_aldaketa
BEFORE UPDATE ON erabiltzaileak
FOR EACH ROW
BEGIN
    SET NEW.azken_aldaketa = NOW();
END$$
DELIMITER ;
```

* **Ebidentzia / Pantaila-argazkia:**
> *(Gehitu hemen triggerrak sortu eta ekintza eragin ostean taula aldaketak erakusten dituzten pantaila-argazkiak)*
