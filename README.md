# STY Erronka 2 - Dokumentazio Teknikoa

Erronka honen helburua enpresa-ingurune simulatu baterako azpiegitura informatiko osoa, segurua, erredundantea eta jasangarria diseinatu, hedatu eta dokumentatzea da.

Proiektu honek Goi Mailako Lanbide Heziketako moduluen ikasketa-emaitzak (IE / RA) betetzen ditu, ebaluazio-errubrikan **AURRERATUA (10)** kalifikazioa lortzeko irizpideak jarraituz.

---

## 📚 Moduluen Dokumentazio Indizea

| Modulua | Deskribapena | Dokumentazio Fitxategia | Pisu Erlatiboa |
| :--- | :--- | :--- | :---: |
| **DBKSA** | Datu-Baseak Kudeatzeko Sistemak (Sarbidea, baimenak, funtzioak, prozedurak, triggerrak) | [01_DBKSA.md](file:///c:/Users/jokin/Documents/GitHub/STY_erronka2/docs/01_DBKSA.md) | 47,5% |
| **WAE & IAW** | Web Aplikazioak eta Ezarpena (WordPress, GitHub Projects, Intraneta, Iruzkinen panela) | [02_WAE_IAW.md](file:///c:/Users/jokin/Documents/GitHub/STY_erronka2/docs/02_WAE_IAW.md) | 25,0% |
| **SZERI** | Sareko Zerbitzuak eta Internet (DNS optimizatua, DHCP aurreratua, Web vhosts HTTPS, FTP segurua) | [03_SZERI.md](file:///c:/Users/jokin/Documents/GitHub/STY_erronka2/docs/03_SZERI.md) | 48,0% |
| **SEG** | Segurtasuna eta Datu Babesa (Kontingentzia plana PILAR, Honeypot, IDS, VPN, Firewall, DBLO/LOPD) | [04_SEG.md](file:///c:/Users/jokin/Documents/GitHub/STY_erronka2/docs/04_SEG.md) | 48,0% |
| **SEA** | Sareko Sistema Eragileak (Active Directory, DFS, Atazen automatizazioa/cron, Gidoiak eta AD komandoak) | [05_SEA.md](file:///c:/Users/jokin/Documents/GitHub/STY_erronka2/docs/05_SEA.md) | 40,0% |
| **SB** | Sistema Birtualizatuak eta HA (Jarraikortasuna, Proxmox/CEPH/corosync Klusterra, VM/LXC HA) | [06_SB.md](file:///c:/Users/jokin/Documents/GitHub/STY_erronka2/docs/06_SB.md) | 41,0% |
| **HW** | Hardware Espezifikoa eta Zentroak (IoT sentsoreak, RAID 5+ NAS, Inbentario automatikoa, Hot-Swap) | [07_HW.md](file:///c:/Users/jokin/Documents/GitHub/STY_erronka2/docs/07_HW.md) | 48,0% |
| **JASAN** | Jasangarritasuna eta Ekonomia Zirkularra (ESG, GJH/ODS 2030, Berrerabilera, Kutsadura aztarna) | [08_JASAN.md](file:///c:/Users/jokin/Documents/GitHub/STY_erronka2/docs/08_JASAN.md) | 30,0% |
| **PROG** | Programazioa eta Sentsorizazioa (GitHub kontrola, IoT mikrokontroladoreak, Web eta DB integrazioa) | [09_PROG.md](file:///c:/Users/jokin/Documents/GitHub/STY_erronka2/docs/09_PROG.md) | 42,0% |
| **HACK** | Hacking Etikoa eta Auditoria (Makina ahula, Kutxa Zuriko analisia, Pentesting ustiapena eta defentsak) | [10_HACK.md](file:///c:/Users/jokin/Documents/GitHub/STY_erronka2/docs/10_HACK.md) | 32,0% |

---

## 🚀 Proiektuaren Egitura Orokorra

```text
STY_erronka2/
├── README.md                  # Proiektuaren sarrera orokorra eta indizea
├── docs/                      # Modulu bakoitzeko dokumentazio teknikoa
│   ├── 01_DBKSA.md            # Datu-Baseak Kudeatzeko Sistemak
│   ├── 02_WAE_IAW.md          # Web Aplikazioak eta Ezarpena
│   ├── 03_SZERI.md            # Sareko Zerbitzuak eta Internet
│   ├── 04_SEG.md              # Segurtasuna eta Datu Babesa
│   ├── 05_SEA.md              # Sareko Sistema Eragileak
│   ├── 06_SB.md               # Sistema Birtualizatuak eta HA
│   ├── 07_HW.md               # Hardware Espezifikoa eta Zentroak
│   ├── 08_JASAN.md            # Jasangarritasuna
│   ├── 09_PROG.md             # Programazioa eta Sentsorizazioa
│   └── 10_HACK.md             # Hacking Etikoa eta Auditoria
└── src/                       # Gidoiak, konfigurazioak eta iturburu-kodea (aukerakoa)
```

## 📋 Nola Osatu Dokumentazioa
Fitxategi bakoitzak errubrikako eskakizun maximoak (**Aurreratua - 10**) betetzeko beharrezko egitura, atalak, komando-txantiloiak eta ebidentzia-koadroak barne hartzen ditu. Gehitu atal bakoitzean egindako konfigurazioak, kode zatiak eta pantaila-argazkiak.
