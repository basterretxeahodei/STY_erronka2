# SZERI - Sareko Zerbitzuak eta Internet

**Modulua:** Sareko Zerbitzuak eta Internet  
**Helburua:** Sareko azpiegitura zerbitzuen administrazio osoa: DNS zerbitzari optimizatua, DHCP aurreratua, Apache/Nginx Web Virtual Hosts seguruak (HTTPS) eta FTP segurua (FTPS) erabiltzaile eta baimen isolatuekin.  
**Erreferentzia-rubrika:** IE1 (%11.0), IE2 (%1.0), IE3 (%20.0), IE4 (%16.0) -> Pisu Osoa: %48.00

---

## 1. DNS Zerbitzariaren Administrazioa (IE1)
*Rubrika Aurreratua (10):* Eremu zuzen eta alderantzizkoak, pribatuak zein publikoak, erregistro mota guztiekin definitu eta instalatu dira konfigurazio optimizatu batekin.

### 1.1. Eremuen Banaketa eta Arkitektura
* **Eremu Zuzenak (Forward Zones):**
  - Barneko eremu pribatua: `enpresa.lan` edo `erronka2.eus`
  - Eremu publikoa / simulazioa: `publikoa.enpresa.eus`
* **Alderantzizko Eremuak (Reverse Zones):**
  - IP segmentua (adib. `192.168.1.0/24` -> `1.168.192.in-addr.arpa`)

### 1.2. Konfigurazio Fitxategiak (Adib. BIND9 / Named)
```text
// /etc/bind/named.conf.local
zone "erronka2.lan" {
    type master;
    file "/etc/bind/zones/db.erronka2.lan";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.1";
};
```

* **Eremu Zuzeneko Erregistroak (`db.erronka2.lan`):**
```text
$TTL    604800
@       IN      SOA     ns1.erronka2.lan. admin.erronka2.lan. (
                              2026112401 ; Serial
                                  604800 ; Refresh
                                   86400 ; Retry
                                 2419200 ; Expire
                                  604800 ) ; Negative Cache TTL
;
@       IN      NS      ns1.erronka2.lan.
ns1     IN      A       192.168.1.10
srv-web IN      A       192.168.1.20
srv-ftp IN      A       192.168.1.30
mail    IN      A       192.168.1.40
@       IN      MX  10  mail.erronka2.lan.
www     IN      CNAME   srv-web.erronka2.lan.
ftp     IN      CNAME   srv-ftp.erronka2.lan.
```

* **Alderantzizko Eremua (`db.192.168.1`):**
```text
@       IN      NS      ns1.erronka2.lan.
10      IN      PTR     ns1.erronka2.lan.
20      IN      PTR     srv-web.erronka2.lan.
30      IN      PTR     srv-ftp.erronka2.lan.
```

* **Egiaztapena (`named-checkzone` eta `dig` / `nslookup`):**
```bash
named-checkconf
named-checkzone erronka2.lan /etc/bind/zones/db.erronka2.lan
dig @192.168.1.10 www.erronka2.lan
dig @192.168.1.10 -x 192.168.1.20
```
* **Ebidentziak:**
> *(Gehitu hemen erregistroen eta dig/nslookup komandoen pantaila-argazkiak)*

---

## 2. DHCP Zerbitzari Aurreratua (IE2)
*Rubrika Aurreratua (10):* Kokapen optimoan kokatua, eremuak, erreserbak (MAC bidez), eta parametro aurreratuak (atebidea, DNS zerbitzariak, lease-time) definituta.

### 2.1. Konfigurazioa (`kea-dhcp4` edo `isc-dhcp-server`)
```text
# /etc/dhcp/dhcpd.conf
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 192.168.1.10;
    option domain-name "erronka2.lan";
    default-lease-time 86400;
    max-lease-time 172800;

    # Erreserba finkoa MAC bidez
    host inprimagailua {
        hardware ethernet 00:11:22:33:44:55;
        fixed-address 192.168.1.50;
    }
}
```
* **Bezeroaren Egiaztapena (`ip a` / `ipconfig /all`):**
* **Ebidentziak:**
> *(Gehitu hemen bezeroak IPa eta aukerak behar bezala jaso izanaren pantaila-argazkia)*

---

## 3. Web Zerbitzaria eta Virtual Host Seguruak (IE3)
*Rubrika Aurreratua (10):* Konfigurazio optimizatua, hainbat Virtual Host (publikoak eta barne-pribatuak), SSL/TLS protokolo seguruak (HTTPS) eta HTTP-tik HTTPS-ra birbideratze automatikoa.

### 3.1. Virtual Host Publikoa eta Pribatua (Apache2 / Nginx)
```apache
# /etc/apache2/sites-available/intranet.conf (Virtual Host Pribatua - HTTPS)
<VirtualHost *:443>
    ServerName intranet.erronka2.lan
    DocumentRoot /var/www/intranet

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/intranet.crt
    SSLCertificateKeyFile /etc/ssl/private/intranet.key

    <Directory /var/www/intranet>
        Options FollowSymLinks
        AllowOverride All
        Require ip 192.168.1.0/24
    </Directory>
</VirtualHost>

# HTTP -> HTTPS Birbideratzea
<VirtualHost *:80>
    ServerName intranet.erronka2.lan
    Redirect permanent / https://intranet.erronka2.lan/
</VirtualHost>
```
* **Gaitzeko eta probatzeko komandoak:**
```bash
a2ensite intranet.conf
a2enmod ssl rewrite
systemctl reload apache2
```
* **Ebidentziak:**
> *(Gehitu hemen arakatzaileko konexio segurua eta guneen pantaila-argazkiak)*

---

## 4. Fitxategi Transferentzia Segurua - FTP/FTPS (IE4)
*Rubrika Aurreratua (10):* FTP zerbitzua definitu, autentifikazio seguruarekin (FTPS / TLS), taldeak eta erabiltzaileak dagozkien baimen isolatuekin (chroot) eta direktorio seguruekin.

### 4.1. vsftpd / ProFTPD Konfigurazio Segurua
```ini
# /etc/vsftpd.conf
listen=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
allow_writeable_chroot=YES

# SSL/TLS Konfigurazioa (FTPS)
ssl_enable=YES
rsa_cert_file=/etc/ssl/certs/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.key
force_local_data_ssl=YES
force_local_logins_ssl=YES
```

### 4.2. Erabiltzaile eta Taldeen Baimenak
| Taldea / Erabiltzailea | Karpetaren bidea | Baimenak |
| :--- | :--- | :--- |
| `ftp_teknikariak` (`ftpuser1`) | `/srv/ftp/teknikariak` | Irakurketa eta idazketa |
| `ftp_irakurleak` (`ftpuser2`) | `/srv/ftp/publikoa` | Irakurketa soilik |

* **Bezeroaren Konexio Egiaztapena (FileZilla / CLI lftp TLS bidez):**
* **Ebidentziak:**
> *(Gehitu hemen FileZilla-ko ziurtagiriaren onarpena eta fitxategien kargatze/deskargatze arrakastatsua)*
