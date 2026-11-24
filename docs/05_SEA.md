# SEA - Sareko Sistema Eragileak

**Modulua:** Sareko Sistema Eragileak  
**Helburua:** Active Directory domeinu zerbitzua, DFS banatua eta konfiantzazko erlazioak, Linux (cron) eta Windows atazen automatizazioa (antirootkit, eguneratzeak, babes-kopiak, antibirusa), sistema heterogeneoen urruneko administrazioa eta PowerShell bidezko AD automatizazio aurreratua (iragazkiak eta pipe-ak).  
**Erreferentzia-rubrika:** IE1 (%10.0), IE3 (%25.0), IE4 (%10.0), IE7 (%5.0) -> Pisu Osoa: %50.00

---

## 1. Direktorio Zerbitzua, Konfiantzazko Erlazioak eta DFS (IE1)
*Rubrika Aurreratua (10):* Egitura osoa dokumentatuta, domeinuen arteko konfiantzazko erlazioak ezarrita, DFS biltegiratze banatua eratuta mintegika baimen pertsonalizatuekin, sarbide automatizatua eta domeinu gurutzatuen egiaztapena (fitxategiak sortuz).

### 1.1. Active Directory Egitura eta Konfiantzazko Erlazioa (Trust Relationship)
* **Domeinuak:**
  - Domeinu Nagusia: `enpresa.lan`
  - Bigarren Domeinua / Filiala: `filiala.lan`
* **Konfiantzazko Erlazioaren Arrazoia eta Zergatia:** Bi erakunde edo sailen arteko baliabideak (fitxategiak, zerbitzuak) modu zentralizatu eta seguruan partekatzeko baimenak kudeatzea.
* **Ezarpena:**
  - Bi norabideko konfiantza (Two-way Forest / External Trust).
* **Ebidentzia / Froga:**
  - `filiala.lan` domeinuko erabiltzaileak `enpresa.lan` sareko makinan saioa hasten du.

### 1.2. DFS (Distributed File System) Egitura
* **DFS Erroaren Izena:** `\\enpresa.lan\Datuak`
* **Helburua:** Hainbat zerbitzaritan banatutako karpetak izen-espazio bakar batean zentralizatzea, erredundantzia eta erabilgarritasuna bermatuz.
* **Karpeta eta Mintegi Baimenak:**
  - `\\enpresa.lan\Datuak\Departamentuak\Administrazioa` -> Soilik Administrazioko erabiltzaileak
  - `\\enpresa.lan\Datuak\Departamentuak\Teknikariak` -> Soilik Teknikariak
  - `\\enpresa.lan\Datuak\Publikoa` -> Erabiltzaile guztiak
* **Sarbidea Automatizatzea:** GPO bidez unitate sareko disko gisa mapeatzea (`Z:\` diskoa).
* **Domeinu Desberdinetik Atzipen Froga:**
  - `filiala.lan` domeinuko erabiltzaileak DFS karpetan sartu eta `.txt` fitxategi berri bat sortu du.
* **Ebidentziak:**
> *(Gehitu hemen DFS kudeaketa, GPO unitate-mapeoa eta bigarren domeinuko fitxategi-sortzearen pantaila-argazkiak)*

---

## 2. Atazen Automatizazioa eta Programatutako Prozesuak (IE3)
*Rubrika Aurreratua (10):* Linux cron atazak (antirootkit analisia + logak, eguneratze automatikoa + logak) eta Windows atazak (biltegiratze partekatuaren backup automatikoa eta Windows Defender eskaneatze automatikoa).

### 2.1. Linux Atazak: `cron` Bidezko Automatizazioa
#### A) Antirootkit Analisia eta Logak (`rkhunter` / `chkrootkit`)
* **Script-a (`/opt/scripts/antirootkit_scan.sh`):**
```bash
#!/bin/bash
LOG_FILE="/var/log/antirootkit_$(date +\%F).log"
echo "=== Antirootkit analisia hasia: $(date) ===" >> "$LOG_FILE"
rkhunter --check --skip-keypress --report-warnings-only >> "$LOG_FILE" 2>&1
echo "=== Analisia amaitua: $(date) ===" >> "$LOG_FILE"
```
* **Cron Programazioa (`/etc/crontab`):**
```text
# Gauero 02:00etan exekutatu
0 2 * * * root /bin/bash /opt/scripts/antirootkit_scan.sh
```

#### B) Sistema Eguneratze Automatikoa eta Logak
* **Script-a (`/opt/scripts/system_update.sh`):**
```bash
#!/bin/bash
LOG_FILE="/var/log/system_update_$(date +\%F).log"
echo "=== Sistema eguneratzea hasia: $(date) ===" >> "$LOG_FILE"
apt-get update && apt-get upgrade -y >> "$LOG_FILE" 2>&1
echo "=== Eguneratzea amaitua: $(date) ===" >> "$LOG_FILE"
```
* **Cron Programazioa:**
```text
# Igandero 04:00etan
0 4 * * 0 root /bin/bash /opt/scripts/system_update.sh
```

---

### 2.2. Windows Atazak: Task Scheduler (Ataza Programatzailea)
#### A) Biltegiratze Partekatuaren Backup Automatikoa
* **PowerShell Script-a (`C:\Scripts\Backup_DFS.ps1`):**
```powershell
$source = "C:\DFS_Roots\Datuak"
$dest = "E:\Backups\DFS_$(Get-Date -Format 'yyyy-MM-dd')"
$log = "C:\Logs\Backup_$(Get-Date -Format 'yyyy-MM-dd').log"

New-Item -ItemType Directory -Force -Path $dest | Out-Null
robocopy $source $dest /MIR /R:2 /W:5 /LOG:$log
```
* **Ataza Programatzailearen Konfigurazioa:** Egunero 23:00etan SISTEMA kontuarekin exekutatua.

#### B) Windows Defender Antibirus Eskaneatze Automatikoa
* **PowerShell komandoa / Script-a:**
```powershell
Start-MpScan -ScanType QuickScan
```
* **Ataza Programatzailearen Konfigurazioa:** Egunero 13:00etan abiarazia.
* **Ebidentziak:**
> *(Gehitu hemen cron log fitxategiak, Task Scheduler leihoak eta Windows gertaera-erregistroak)*

---

## 3. Sareko Sistemen Urruneko Administrazioa (IE4)
*Rubrika Aurreratua (10):* Urruneko administrazio tresnak justifikatuta, integratutakoak (SSH, RDP, WinRM) eta gehigarriak instalatuta (Cockpit / Windows Admin Center), sistema heterogeneoen arteko probak (Linux <-> Windows).

### 3.1. Tresnen Justifikazioa eta Kokapena
| Tresna | Motakoa | Justifikazioa eta Erabilera |
| :--- | :--- | :--- |
| **OpenSSH** | Natiboa (Linux / Windows) | Komando lerro bidezko kudeaketa seguru eta azkarra, zifratua |
| **RDP** | Natiboa (Windows) | Windows zerbitzarien ingurune grafikoa administratzeko |
| **Cockpit / Webmin** | Zerbitzu Gehigarria (Web) | Linux zerbitzariaren errendimendua, diskoak eta sareak web bidez kontrolatzeko |
| **Windows Admin Center** | Zerbitzu Gehigarria (Web) | Windows zerbitzarien kudeaketa grafiko zentralizatua |

### 3.2. Sistema Heterogeneoen Arteko Probak
* **Linuxetik Windows administratzea:** `xfreerdp` edo PowerShell Remoting bidez konexioa.
* **Windowsetik Linux administratzea:** PuTTY edo OpenSSH bidez terminal konexioa, eta Cockpit web panela arakatzailetik.
* **Ebidentziak:**
> *(Gehitu hemen bi norabideetako urruneko saioen pantaila-argazkiak)*

---

## 4. Gidoiak eta Active Directory Administrazioa (IE7)
*Rubrika Aurreratua (10):* Gidoi-lengoaia egituratuak (PowerShell), OUak/taldeak/erabiltzaileak sortzeko komandoak, kontsulta komandoak iragazkiekin (`-Filter`) eta komandoen pipe-ekin (`|`).

### 4.1. Egiturak Sortzeko PowerShell Script-a
```powershell
# Antolaketa Unitatea (OU) sortu
New-ADOrganizationalUnit -Name "Teknikariak" -Path "DC=enpresa,DC=lan"

# Segurtasun Taldea sortu
New-ADGroup -Name "GRP_Teknikariak" -GroupScope Global -GroupCategory Security -Path "OU=Teknikariak,DC=enpresa,DC=lan"

# Erabiltzaile berria sortu eta taldera gehitu
$pwd = ConvertTo-SecureString "PasahitzSegurua2026!" -AsPlainText -Force
New-ADUser -Name "Aitor Mendia" -SamAccountName "amendia" -UserPrincipalName "amendia@enpresa.lan" `
           -AccountPassword $pwd -Enabled $true -Path "OU=Teknikariak,DC=enpresa,DC=lan"

Add-ADGroupMember -Identity "GRP_Teknikariak" -Members "amendia"
```

### 4.2. Kontsultak Iragazkiekin (`-Filter`) eta Pipe-ekin (`|`)
```powershell
# 1. Filtro bidez kontsultatu eta Pipe bidez propietateak hautatu
Get-ADUser -Filter {Enabled -eq $true} -Properties Department, EmailAddress |
    Where-Object {$_.Department -like "*Teknika*"} |
    Select-Object Name, SamAccountName, EmailAddress |
    Sort-Object Name

# 2. Taldeko partaideak kontsultatu eta CSV fitxategira esportatu
Get-ADGroupMember -Identity "GRP_Teknikariak" |
    Get-ADUser -Properties Title |
    Select-Object SamAccountName, Name, Title |
    Export-Csv -Path "C:\Txostenak\Teknikariak.csv" -NoTypeInformation -Encoding UTF8
```
* **Ebidentziak:**
> *(Gehitu hemen PowerShell kontsolako exekuzioen eta esportatutako CSVren pantaila-argazkiak)*
