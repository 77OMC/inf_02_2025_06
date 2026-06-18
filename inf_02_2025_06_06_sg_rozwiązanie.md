# Rozwiązanie egzaminu INF.02 - Sesja Czerwiec 2025, wersja arkusza 06 SG

## 1. Wykonaj montaż okablowania sieciowego

**Rozwiązanie:**
Zgodnie z wytycznymi w pliku `README.md` (Zasada A), pomijamy zadania dotyczące fizycznego montażu okablowania sieciowego. W realiach egzaminacyjnych należałoby przygotować patchcord U/UTP z jednej strony zakończony w module Keystone (gniazdku natynkowym), a z drugiej zamontowany nożem uderzeniowym (Krone) na odpowiednim pinie Panelu Krosowego. Według wytycznych należy wszędzie zarabiać kable zgodnie z sekwencją T568B.

## 2. Przeprowadź identyfikację parametrów dysku twardego stacji roboczej

Zgodnie z poleceniem na egzaminie należy zidentyfikować parametry dla systemu Windows, natomiast poniżej znajduje się rozwiązanie dla obu systemów operacyjnych. Wymagane jest również sporządzenie zrzutów do folderu `Identyfikacja` na USB i uzupełnienie Tabeli 1.

### C1 - Windows 10/11

Dane pozyskujemy głównie z systemowych przystawek zarządzania.
**Polecenia do odczytu ustawień (Weryfikacja):**

```cmd
:: Model dysku
wmic diskdrive get model
:: Wersja sterownika dysku
wmic path win32_PNPEntity where "Caption like '%Dysk%'" get caption, driverversion
:: Liczba i Typ partycji
echo list disk > list_disk.txt && diskpart /s list_disk.txt
echo select disk 0 > list_part.txt && echo list partition >> list_part.txt && diskpart /s list_part.txt
```

Alternatywnie z interfejsu graficznego:

1. `diskmgmt.msc` -> Pokazuje całkowitą pojemność, liczbę i typ partycji (np. Podstawowa / NTFS).
2. `devmgmt.msc` -> Zakładka Dyski. Prawy klik na dysk -> Właściwości.
   - W karcie Szczegóły: Model sprzętu.
   - W karcie Sterownik: Wersja sterownika.

### C2 - Linux Ubuntu Stacja

W Linuksie korzystamy z linii komend.
**Polecenia do odczytu ustawień (Weryfikacja):**

```bash
# Model dysku i całkowita pojemność:
lsblk -d -o name,model,size

# Liczba i Typ partycji
# Typ widnieje najczęściej w kolumnie "FSTYPE" (np. ext4, ntfs, vfat) lub "Type" przy fdisk (np. Linux, EFI)
sudo fdisk -l
lsblk -f

# Wersja "sterownika" / informacja o załadowanym module jądra blokowego dla dysku (np. sda):
udevadm info -a -n /dev/sda | grep DRIVER
```

Zrzuty odpowiednio dopinamy do folderu.

## 3. Konfiguracja rutera (MikroTik)

**Rozwiązanie:**
Konfiguracja CLI MikroTik. Zmienne oznaczające stanowisko przyjęto jako `XX = 06`.

**Konfiguracja CLI:**

```routeros
# Zmiana hasła, jeśli jest wymuszona
/user set admin password=NoweHaslo!

# 1. Adres IP interfejsu LAN
/ip address add address=192.168.0.3/24 interface=bridge_lan

# 2. Serwer DHCP włączony na interfejsie LAN (np. bridge)
# Ustalamy pulę:
/ip pool add name=pula_dhcp ranges=192.168.0.30-192.168.0.40
# Sieć serwera:
/ip dhcp-server network add address=192.168.0.0/24 gateway=192.168.0.3 dns-server=8.8.8.8
# Uruchomienie serwera dhcp na interfejsie bridge_lan powiązanym z naszą pulą:
/ip dhcp-server add name=dhcp1 interface=bridge_lan address-pool=pula_dhcp disabled=no

# Zarezerwowanie IP 192.168.0.30 dla interfejsu WLAN stacji roboczej:
# Odczytujemy wcześniej MAC adres stacji roboczej (podano losowy przykładowy)
/ip dhcp-server lease add address=192.168.0.30 mac-address=11:22:33:44:55:66 server=dhcp1

# 3. Konfiguracja sieci bezprzewodowej 2.4 GHz
# Dodanie profilu WPA2 (AES/CCMP)
/interface wireless security-profiles add name=sec_egzamin authentication-types=wpa2-psk wpa2-pre-shared-key=stanowisko06 mode=dynamic-keys
# Ustawienie i włączenie bezprzewodowego interfejsu z profilem (Stanowisko 6 -> Kanał 6 -> Częstotliwość 2437 MHz)
/interface wireless set wlan1 disabled=no mode=ap-bridge ssid=STANOWISKO_06 band=2ghz-b/g/n frequency=2437 security-profile=sec_egzamin
```

**Odczyt konfiguracji rutera (Weryfikacja):**

```routeros
/ip address print
/ip dhcp-server print
/ip dhcp-server lease print
/interface wireless print advanced
/interface wireless security-profiles print
```

## 4. Konfiguracja przełącznika 1 (MikroTik)

**Rozwiązanie:**
(Przełącznik podpięty pod sieć wydzieloną 172.16.11.x)

**Konfiguracja CLI:**

```routeros
# Włączenie vlan-filtering
/interface bridge add name=bridge1 vlan-filtering=no

# Porty 1 i 2 bez tagowania w trybie dostępowym
/interface bridge port add bridge=bridge1 interface=ether1 pvid=10
/interface bridge port add bridge=bridge1 interface=ether2 pvid=10

# Ustawienie tagowania dla portu trunk (np. port 3 - niewymieniony ale niezbędny przy przesyłaniu vlan z serwera,
# jeżeli LAN_Z z serwera wchodzi na porcie dostępowym pominąć ten krok i dodać port serwera do wpisu powyżej)
/interface bridge vlan add bridge=bridge1 vlan-ids=10 untagged=ether1,ether2

/interface bridge set bridge1 vlan-filtering=yes

# Adres IP dla L3 na VLAN
/interface vlan add name=vlan10 vlan-id=10 interface=bridge1
/ip address add address=172.16.11.3/25 interface=vlan10

# Brama domyślna: adres IP interfejsu LAN_Z serwera
/ip route add dst-address=0.0.0.0/0 gateway=172.16.11.5
```

## 5. Konfiguracja przełącznika 2 (MikroTik)

**Rozwiązanie:**
Zarządzalny L2/L3 przełącznik skonfigurowany pod główną sieć.

**Konfiguracja CLI:**

```routeros
# Nie przypisujemy go z zadania do dodatkowych vlan-id, konfiguracja IP tylko dla zarządzenia
/ip address add address=192.168.0.2/24 interface=bridge1
# Brama domyślna dla switcha kierowana na LAN routera
/ip route add dst-address=0.0.0.0/0 gateway=192.168.0.3
```

**Odczyt konfiguracji obu przełączników (Weryfikacja):**

```routeros
/interface bridge vlan print
/interface bridge port print
/ip address print
/ip route print
```

## 6. Połączenie urządzeń sieciowych

*(Zadanie pominięto zgodnie z uwagą A w pliku README).*

## 7. Konfiguracja serwera

(W rozwiązaniach używany jest symbol stanowiska XX=06).

### D1 - Windows Server 2022

**Rozwiązanie:**

1. **Konfiguracja Przewodowej Karty `LAN_Z`:**
   - Otwieramy `ncpa.cpl`.
   - Zmieniamy nazwę karty podpiętej do Przełącznika 1 na `LAN_Z`.
   - Edytujemy Właściwości IPv4: Adres IP to `172.16.11.5`. Maska podsieci: `255.255.255.128` (/25). Brama domyślna - pusta. Serwer DNS: `127.0.0.1`.

2. **Konfiguracja Przewodowej Karty `LAN_W`:**
   - Zmieniamy nazwę karty podpiętej do Przełącznika 2 na `LAN_W`.
   - Właściwości IPv4: Adres IP: `192.168.0.16` (10 + 6). Maska podsieci: `255.255.255.0`. Brama domyślna: `192.168.0.3` (Ruter). Serwer DNS: `127.0.0.1`.

3. **Serwer WWW (IIS) oraz DNS:**
   - Prawa klik z Start -> Utworzenie folderu `C:\DODATKI`.
   - Skopiowanie pliku `index.html` z pamięci USB.
   - W Menedżerze serwera Instalacja Roli Web Server (IIS) oraz DNS.
   - Otwieramy `inetmgr` (Zarządzanie IIS). Rozwijamy lewe pole "Witryny". Klikamy prawym "Dodaj witrynę".
   - Nazwa: `INF.02`.
   - Ścieżka fizyczna: `C:\DODATKI`.
   - Powiązanie z adresem IP (z listy rozwijanej wskazujemy IP LAN_W: `192.168.0.16`), port zostawiamy jako `80`.
   - W ustawieniach tejże witryny przechodzimy do apletu "Dokument Domyślny", dodajemy `index.html` i przesuwamy go wyżej.

**Polecenia do weryfikacji ustawień (PowerShell):**

```powershell
Get-NetIPAddress -InterfaceAlias LAN_Z
Get-NetIPAddress -InterfaceAlias LAN_W
Get-Website
```

---

### D2 - Linux Ubuntu Server

**Rozwiązanie:**

1. **Konfiguracja interfejsów (netplan):**
   - Plik `/etc/netplan/01-netcfg.yaml`:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      set-name: LAN_Z
      addresses: [172.16.11.5/25]
      nameservers:
        addresses: [127.0.0.1]
    ens34:
      set-name: LAN_W
      addresses: [192.168.0.16/24] # Założono XX=06
      routes:
        - to: default
          via: 192.168.0.3
      nameservers:
        addresses: [127.0.0.1]
```

- Zaaplikowanie planu: `sudo netplan apply`

1. **Przygotowanie plików WWW:**
   - `sudo mkdir -p /DODATKI`
   - Kopiujemy z pendrive do katalogu nowo powstałego (zakładając folder montowania):
     `sudo cp /media/pendrive/DOKUMENTACJA/PROGRAMY/PLIKI/index.html /DODATKI/`

2. **Serwer WWW (Apache2) i Serwer DNS (Bind9):**
   - Instalacja pakietów: `sudo apt update && sudo apt install -y apache2 bind9`
   - Nowy plik hosta virtualnego `sudo nano /etc/apache2/sites-available/inf02.conf`:

```apache
<VirtualHost 192.168.0.16:80>
    ServerName inf.02
    DocumentRoot /DODATKI
    DirectoryIndex index.html
</VirtualHost>
```

- Aktywacja:
     `sudo a2dissite 000-default.conf`
     `sudo a2ensite inf02.conf`
     `sudo systemctl restart apache2`

**Weryfikacja parametrów (Bash):**

```bash
ip a show LAN_Z
ip a show LAN_W
systemctl status apache2 bind9
```

## 8. Konfiguracja stacji roboczej

(Zmienna stanowiska dla ułatwienia przyjęta jako `XX` = 06).

### C1 - Windows 10/11

**Rozwiązanie:**

1. **Konfiguracja sieci `LAN_K` oraz Wi-Fi:**
   - W narzędziu Połączeń Sieciowych `ncpa.cpl` zmieniamy nazwę bezprzewodowej karty na `LAN_K`.
   - Zaznaczamy w niej, po wejściu we Właściwości -> IPv4 "Uzyskaj adres IP automatycznie" oraz "Uzyskaj adres serwera DNS automatycznie".
   - Łączymy się z siecią `STANOWISKO_06`, wpisując hasło `stanowisko06`. W tle urządzenie dostanie adres `192.168.0.30`.
   - Celem odświeżenia otwieramy wiersz poleceń `CMD` i piszemy: `ipconfig /release` i następnie `ipconfig /renew`.

2. **Identyfikacja domeny i środowisko:**
   - W `sysdm.cpl` (Właściwości Systemu) zmieniamy:
     - Nazwa komputera: `stacja_06`.
     - Zmiana opisu: Wypełniamy pole tekstowe `Opis komputera:` -> `stacja_06_robocza`.
     - Grupa robocza: Zmieniamy z WORKGROUP na `INF02`. (Pamiętaj o wymaganym po tym kroku restarcie).

3. **Konto i Logowanie:**
   - `compmgmt.msc` -> Użytkownicy. Tworzymy nowego użytkownika.
   - Nazwa: `jnowak`. Hasło: `JN!20#`. Upewniamy się, że w zakładce z grupami dodana jest grupa "Użytkownicy", usuwamy inne.
   - Logujemy się z ekranu blokady na konto i odpalamy przeglądarkę pod adresem odpowiadającym interfejsowi serwera (np. wpisując `http://192.168.0.16` lub domenę DNS jeśli utworzono odpowiednią na serwerze i ustawiono go DHCPem).

**Weryfikacja parametrów (CMD/PowerShell):**

```powershell
ipconfig /all
systeminfo | findstr /B /C:"Domain" /C:"Host Name"
net user jnowak
```

---

### C2 - Linux Ubuntu Stacja

**Rozwiązanie:**

1. **Ustawienia sieci WLAN (`LAN_K`):**
   - Komendą menadżera sieci:
   - `nmcli connection add type wifi con-name LAN_K ifname wlan0 ssid STANOWISKO_06`
   - `nmcli connection modify LAN_K wifi-sec.key-mgmt wpa-psk wifi-sec.psk stanowisko06`
   - Opcja auto jest domyślnie wybrana. Do odświeżenia DHCP klienta można użyć m.in.: `sudo dhclient -r wlan0` oraz `sudo dhclient wlan0`.
   - `nmcli connection up LAN_K`

2. **Zarządzanie nazewnictwem komputera i grupą:**
   - Zmiana nazwy komputera: `sudo hostnamectl set-hostname stacja_06`
   - Zmiana opisu: `sudo hostnamectl set-chassis "desktop"` lub dodanie metadanych komendą `sudo hostnamectl set-location "stacja_06_robocza"`.
   - Sieci linuksowe Samba - w pliku konfiguracyjnym w polu workgroup zmieniamy nazewnictwo na grupy `INF02` (w `/etc/samba/smb.conf`).

3. **Konto i test domeny:**
   - `sudo useradd -m -G users -s /bin/bash jnowak`
   - `echo "jnowak:JN!20#" | sudo chpasswd`
   - Po przelogowaniu do menedżera, odpalenie np. Firefoxa i wejście na adres IP strony Serwera by odebrać treść.

**Weryfikacja parametrów (Bash):**

```bash
nmcli connection show LAN_K
hostnamectl
ip addr show wlan0
```

## 9. Konfiguracja urządzenia peryferyjnego (Drukarka na serwerze)

Niezależnie od wybranego systemu OS drukarki są zarządzane po zdefiniowaniu odpowiednich zadań z udziałem sprzętowym. Skonfigurowana ma być na maszynie o nazwie `Serwer` i udostępniona w środowisku lokalnym.

### D1 - Windows Server 2022

- Otwieramy Zarządzanie drukowaniem lub Drukarki i Skanery w Panelu Sterowania.
- Dodajemy Urządzenie TCP/IP. Host: `192.168.0.200`. Odznaczamy automatyczne pytania. W portach wskaż standard Raw/TCP i port 9100. Kiedy serwer zapyta o ścieżkę instalacyjną sterowników wskazujemy folder `DRUKARKA` z pendrive'a.
- Prawy klik na Drukarkę -> Właściwości.
  - Zakładka "Zaawansowane" -> "Zawsze dostępne od" : wpisujemy `7:30` - `21:30`.
  - Zakładka "Udostępnianie" -> "Udostępnij tę drukarkę" -> Wpisujemy Nazwa udziału: `druk_06`.
- Prawy klik na Drukarkę -> Preferencje drukowania.
  - Odszukujemy "Układ" -> wybieramy kierunek: `Pozioma`
  - W zaawansowanych lub szczegółowych ustawieniach profilu szukamy Trybu Toner Save / EconoMode i wymuszamy na włączony (Oszczędność toneru / Ekonomiczny).
- Klikamy drukuj stronę testową w preferencjach i meldujemy podniesieniem ręki.

### D2 - Linux Ubuntu Server (Konfiguracja PPD CUPS w pętli opcji)

- `sudo lpadmin -p drukarka_06 -E -v socket://192.168.0.200:9100 -m everywhere`
- Do zmiany czasu CUPS pozwala nakładać limity lub harmonogramy na zadania przez systemy poleceń cron. Właściwa konfiguracja CUPS obejmuje polecenia konfiguracyjne dla drukarki (jak np. Toner i Page-Orientation) realizowane głównie sterownikiem przez protokół PPD. Domyślne wartości: `lpoptions -p drukarka_06 -o landscape` dla orientacji poziomej.

## 10. Test komunikacji

**Rozwiązanie:**
Zadanie wymaga wykonania polecenia pingu za pomocą terminala Serwera.
Sprawdzamy zaporę systemu `Stacja robocza`: Na stacji otwieramy `wf.msc` i w Regułach wchodzących dla Windows Defendera Włączamy `ICMPv4` (zapytania echowe pingu).

Z serwera dla stacji Windows / Linuksa (przykład adresów stanowiska 06 z poleceń):

```cmd
ping 192.168.0.3     (Interfejs LAN rutera)
ping 192.168.0.30    (Adres Stacji w WLAN, która dostała poprawny przydział IP wg rezerwacji)
ping 192.168.0.200   (Nasza dodana Drukarka RAW)
```

Na stacji roboczej by ujrzeć automatycznie uzyskany przydział poinformuj wpisaniem komendy w wiersz poleceń `ipconfig` (Win) lub `ip a` (Linux). Po pojawieniu się pozytywnych zwrotów weryfikacji powiadom Przewodniczącego Zespołu.
