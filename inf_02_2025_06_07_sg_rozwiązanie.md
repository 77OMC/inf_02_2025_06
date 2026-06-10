# Rozwiązanie egzaminu INF.02 - Sesja Czerwiec 2025, wersja arkusza 07 SG

## 1. Wykonaj montaż okablowania sieciowego

**Rozwiązanie:**
Zgodnie z wytycznymi w pliku `README.md` (Zasada A), pomijamy zadania dotyczące fizycznego montażu okablowania sieciowego. W realiach egzaminacyjnych należałoby przygotować patchcord U/UTP prosty, co w oparciu o polecenie oznacza założenie wtyków RJ-45 (8P8C) po obu stronach zgodnie z sekwencją T568A: Biało-Zielony, Zielony, Biało-Pomarańczowy, Niebieski, Biało-Niebieski, Pomarańczowy, Biało-Brązowy, Brązowy. Weryfikację przeprowadza się standardowym testerem okablowania.

## 2. Konfiguracja rutera (MikroTik)

**Rozwiązanie:**
Konfiguracja sprzętu MikroTik z poziomu CLI (lub WinBox). W przykładzie wykorzystano interfejs tekstowy RouterOS.

**Konfiguracja CLI:**

```routeros
# Zmiana hasła (jeśli wymuszona przy pierwszym logowaniu)
/user set admin password=NoweHaslo!

# 1. Adres IP dla LAN
/ip address add address=192.168.1.1/24 interface=bridge_lan

# 2. Wyłączenie serwera DHCP
/ip dhcp-server remove [find]

# 3. Włączenie obsługi VLAN 802.1Q oraz utworzenie interfejsu VLAN o ID=2 na porcie 2 (tryb trunk)
/interface vlan add name=vlan2 vlan-id=2 interface=ether2
# 4. Adres IP dla interfejsu VLAN o ID=2
/ip address add address=192.168.2.100/24 interface=vlan2

# 5. Utworzenie interfejsu VLAN o ID=3 na porcie 2 (tryb trunk)
/interface vlan add name=vlan3 vlan-id=3 interface=ether2
# 6. Adres IP dla interfejsu VLAN o ID=3
/ip address add address=192.168.0.100/24 interface=vlan3

# 7. Włączenie routingu między sieciami VLAN
# W systemie RouterOS nadanie adresów IP na interfejsach będących w stanie RUNning (czyli m.in. utworzonych VLAN-ach na działającym interfejsie) powoduje automatyczne dodanie tras (Connected Routes) do tablicy routingu, co umożliwia natywny ruch między podsieciami, o ile nie ograniczają tego reguły Firewalla.
```

**Odczyt konfiguracji rutera (Weryfikacja):**

```routeros
/interface vlan print
/ip address print
/ip dhcp-server print
/ip route print
```

## 3. Konfiguracja przełącznika (MikroTik)

**Rozwiązanie:**
Przełącznik konfigurujemy, definiując w pierwszej kolejności bridge z VLAN filteringiem.

**Konfiguracja CLI:**

```routeros
# Utworzenie bridge'a bez włączonego z góry vlan-filteringu
/interface bridge add name=bridge1 vlan-filtering=no

# 1. Przypisanie portów do bridge'a ze zdefiniowanymi PVID dla trybów dostępu (untagged)
# Port 1 (Trunk) pozostaje z pvid=1 domyślnie, port 2 jest przypisany do VLAN 2, porty 3 i 4 do VLAN 3
/interface bridge port add bridge=bridge1 interface=ether1
/interface bridge port add bridge=bridge1 interface=ether2 pvid=2
/interface bridge port add bridge=bridge1 interface=ether3 pvid=3
/interface bridge port add bridge=bridge1 interface=ether4 pvid=3

# 2. Konfiguracja tagowania i nietagowania na moście dla 802.1Q
# VLAN ID=2 (Port 1 z tagowaniem trunk, Port 2 bez tagowania dostęp)
/interface bridge vlan add bridge=bridge1 vlan-ids=2 tagged=ether1 untagged=ether2

# VLAN ID=3 (Port 1 z tagowaniem trunk, Porty 3 i 4 bez tagowania dostęp)
/interface bridge vlan add bridge=bridge1 vlan-ids=3 tagged=ether1 untagged=ether3,ether4

# 3. Włączenie filtrowania VLAN
/interface bridge set bridge1 vlan-filtering=yes

# 4. Skonfigurowanie adresu IP dla przełącznika zarządzalnego na głównym interfejsie mostka
/ip address add address=192.168.1.144/24 interface=bridge1

# 5. Skonfigurowanie bramy domyślnej
/ip route add dst-address=0.0.0.0/0 gateway=192.168.1.1
```

**Odczyt konfiguracji przełącznika (Weryfikacja):**

```routeros
/interface bridge port print
/interface bridge vlan print
/ip address print
/ip route print
```

## 4. Połączenie urządzeń sieciowych

*(Zadanie pominięto zgodnie z uwagą A w pliku README).* Należałoby połączyć poszczególne urządzenia według schematu i podłączyć je do prądu.

## 5. Konfiguracja serwera

(W rozwiązaniach używany jest symbol stanowiska X. Dla uproszczenia kodu powołujemy się na `X=1`).

### D1 - Windows Server 2022

**Rozwiązanie:**

1. **Konfiguracja Karty `LAN_WWW` i interfejsów:**
   - Otwieramy narzędzie `ncpa.cpl`.
   - Zmieniamy nazwę aktywnej karty sieciowej, podłączonej wg schematu pod przełącznik, na `LAN_WWW`.
   - Drugi, niepodpięty interfejs całkowicie wyłączamy prawym przyciskiem myszy ("Wyłącz").
   - We Właściwościach IPv4 interfejsu `LAN_WWW` ustawiamy parametry (zakładając x=1 z `192.168.0.200+X`):
     - IP: `192.168.0.201`
     - Maska: `255.255.255.0`
     - Brama domyślna: `192.168.0.100` (Rutera)
     - Preferowany serwer DNS: `127.0.0.1` (localhost).

2. **Przygotowanie plików dla witryny WWW:**
   - Na dysku zakładamy nowy folder instalacyjny z powłoki Windows: `C:\www`.
   - Przechodzimy na zawartość przenośnego napędu USB do folderu `DOKUMENTACJA/PROGRAMY/PLIKI`, stamtąd kopiujemy `index.html`.
   - Wklejamy go do utworzonego folderu `C:\www` i zmieniamy mu nazwę na wymaganą przez zadanie `egzamin.html`.

3. **Konfiguracja Serwera DNS:**
   - W Menedżerze Serwera, poprzez panel dodawania ról i funkcji, instalujemy "Serwer DNS".
   - Wchodzimy do narzędzia `dnsmgmt.msc` i klikamy prawym przyciskiem w "Strefy wyszukiwania do przodu" tworząc "Nową strefę". Wybieramy Strefę podstawową.
   - Podajemy nazwę `egzamin.local` i zatwierdzamy kreator.
   - W nowej strefie klikamy prawym, po czym dodajemy "Nowy host (A lub AAAA)...".
   - Nazwa dla hosta to `www`, adres IP wskazujący to IP lokalnego interfejsu: `192.168.0.201`.

4. **Konfiguracja IIS:**
   - W kreatorze dodawania Ról dogrywamy "Serwer sieci Web (IIS)".
   - Wchodzimy w aplikację do zarządzania infrastrukturą IIS - `inetmgr`.
   - Rozwijamy kaskadę naszego serwera i w zakładce "Witryny" usuwamy domyślną witrynę z wpiętego portu 80.
   - Klikamy prawym -> "Dodaj witrynę internetową...".
   - Nazwa: `egzamin`. Ścieżka fizyczna kieruje na katalog: `C:\www`.
   - Typ: HTTP. Adres IP przypisujemy z interfejsu `LAN_WWW` (`192.168.0.201`), port: 80.
   - Klikamy na powstałą witrynę, przechodzimy po środku do "Dokument domyślny". Jeśli nie jest dziedziczona po starej domyślnej witrynie w IIS, dodajemy tu wpis `egzamin.html` i przesuwamy go na samą górę stosu.

**Polecenia do weryfikacji ustawień (PowerShell):**

```powershell
Get-NetIPAddress -InterfaceAlias LAN_WWW
Get-NetAdapter | Format-Table Name, Status
Resolve-DnsName -Name www.egzamin.local
Import-Module WebAdministration
Get-Website
```

---

### D2 - Linux Ubuntu Server

**Rozwiązanie:**

1. **Konfiguracja sieci (netplan):**
   - W terminalu modyfikujemy plik `/etc/netplan/01-netcfg.yaml`:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      set-name: LAN_WWW
      addresses: [192.168.0.201/24] # Założono X=1
      routes:
        - to: default
          via: 192.168.0.100
      nameservers:
        addresses: [127.0.0.1]
    ens34:
      activation-mode: off # Wyłączenie drugiego interfejsu
```

- Aplikacja zmian: `sudo netplan apply`

1. **Kopiowanie zasobów HTML:**
   - Tworzymy plik korzenia serwera WWW: `sudo mkdir /www`
   - Kopiujemy z pendrive'a wkład pod nową nazwą: `sudo cp /media/pendrive/DOKUMENTACJA/PROGRAMY/PLIKI/index.html /www/egzamin.html`

2. **Konfiguracja BIND9 (DNS) i Apache (WWW):**
   - Paczki: `sudo apt update && sudo apt install -y bind9 bind9utils apache2`
   - Dodajemy strefę w `/etc/bind/named.conf.local`:

```text
zone "egzamin.local" {
    type master;
    file "/etc/bind/db.egzamin.local";
};
```

- Skonstruowanie rekordu strefy, kopiujemy z pliku lokalnego: `sudo cp /etc/bind/db.local /etc/bind/db.egzamin.local`. Na końcu tegoż pliku modyfikujemy wpisy domenowe dodając: `www IN A 192.168.0.201`.
- `sudo systemctl restart bind9`
- Utworzenie nowej v-host witryny `sudo nano /etc/apache2/sites-available/egzamin.conf`:

```apache
<VirtualHost 192.168.0.201:80>
    ServerName egzamin.local
    DocumentRoot /www
    DirectoryIndex egzamin.html
</VirtualHost>
```

- Przełączamy witryny i resetujemy serwer HTTP:
     `sudo a2dissite 000-default.conf`
     `sudo a2ensite egzamin.conf`
     `sudo systemctl restart apache2`

**Weryfikacja parametrów (Bash):**

```bash
ip a show LAN_WWW
dig @127.0.0.1 www.egzamin.local
systemctl status bind9 apache2
```

## 6. Konfiguracja stacji roboczej

### C1 - Windows 10/11

**Rozwiązanie:**

1. **Adresacja interfejsu `LAN_SR`:**
   - Narzędziem `ncpa.cpl` zmieniamy nazwę karty LAN na `LAN_SR`.
   - Otwieramy Właściwości -> Protokół IPv4.
   - Adres IP: `192.168.2.10`
   - Maska: `255.255.255.0`
   - Brama: `192.168.2.100` (Ruter w tym VLAN-ie).
   - DNS: Adres IP serwera, u nas `192.168.0.201`.

2. **Wymóg logowania CTRL+ALT+DEL oraz komunikat logowania:**
   - W polu powłoki uruchom wpisujemy program: `secpol.msc` (Zasady Zabezpieczeń Lokalnych).
   - "Zasady lokalne" -> "Opcje zabezpieczeń".
   - Prawym na regułę: `Logowanie interakcyjne: nie wymagaj naciśnięcia klawiszy CTRL+ALT+DEL`. Wymuszamy aby jej stan zmienić na **Wyłączone** (Zaznaczamy to pole kropką). W ten sposób włączymy konieczność klikania w skrót przy logowaniu.
   - W tej samej sekcji na regule `Logowanie interakcyjne: tekst komunikatu dla użytkowników próbujących się zalogować` wpisujemy: `Uwaga! Logujesz się do komputera egzaminacyjnego!`. Tytuł obok komunikatu (`tytuł komunikatu dla...`) możemy uzupełnić czymś opcjonalnym, chociażby spacją.

3. **Przeglądarka internetowa:**
   - Odpalamy systemową przeglądarkę (np. Chrome/Edge) na koncie i przechodzimy na stronę `http://www.egzamin.local`. Dokument `egzamin.html` powinien załadować się na ekranie bez błędów odczytując odpowiedź DNSa do postawionego serwera HTTP na komputerze obok. Zgłaszamy Egzaminatorowi.

**Polecenia do weryfikacji ustawień (CMD/PowerShell):**

```powershell
ipconfig /all
Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "DisableCAD"
Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "legalnoticetext"
```

---

### C2 - Linux Ubuntu Stacja

**Rozwiązanie:**

1. **Adresacja dla stacji roboczej (`LAN_SR`):**
   - Poprzez polecenia Network Managera z CLI:
   - `nmcli connection modify "Połączenie przewodowe 1" connection.id LAN_SR`
   - `nmcli connection modify LAN_SR ipv4.addresses 192.168.2.10/24 ipv4.gateway 192.168.2.100 ipv4.dns 192.168.0.201 ipv4.method manual`
   - `nmcli connection up LAN_SR`

2. **Odpowiednik Komunikatu w środowisku logowania Linux (np. GDM) i konsolowym:**
   - Systemy Linux potrafią wyświetlać dany komunikat w menedżerze logowania.
   - Dla GDM/Gnome wprowadzamy dane modyfikując bazę. Utworzenie profilu i banera w `/etc/dconf/db/gdm.d/01-banner-message`:

```text
[org/gnome/login-screen]
banner-message-enable=true
banner-message-text='Uwaga! Logujesz się do komputera egzaminacyjnego!'
```

- Dopełniamy komendą aplikującą `sudo dconf update`. Dodatkowo, plik motd:
     `echo "Uwaga! Logujesz się do komputera egzaminacyjnego!" | sudo tee /etc/issue.net /etc/issue`

1. **Strona WWW:**
   - Komendą np. z pakietu Curl: `curl -I http://www.egzamin.local` sprawdzamy, po czym pokazujemy okienkową wersję w przeglądarce Egzaminatorowi.

**Weryfikacja parametrów (Bash):**

```bash
nmcli dev show
cat /etc/dconf/db/gdm.d/01-banner-message
```

## 7. Test komunikacji (Serwer)

**Rozwiązanie:**
Aby serwer (np. Windows) miał możliwość bezproblemowej komunikacji, puszczamy testy po protokole ICMP. Jeżeli od serwera nie wychodzi sygnał do stacji lub nie odpowiada, należy umożliwić zaporze Windows odpowiedź echa w opcjach zapory (`wf.msc`) stacji roboczej zaznaczając reguły wejścia ("Udostępnianie plików i drukarek (Żądanie echa)").

Otwieramy konsole na Serwerze i wpisujemy serię adresów:

```cmd
ping 192.168.2.10    (Interfejs Stacji roboczej za routerem z Vlan)
ping 192.168.0.100   (Brama routera)
ping 192.168.0.200   (Drukarka sieciowa pod portem E-X)
```

Jeżeli dostaniemy wszystkie cztery odpowiedzi bez ubytku, operacja komunikacji jest pomyślnie zestawiona.

## 8. Identyfikacja podzespołów

Zadanie opiera się na odnalezieniu poszczególnych komponentów komputera i uzupełnieniu nimi wpisów na dołączonej Tablicy 1 "Specyfikacja stacji roboczej". Odczyty udokumentuj zrzutem ekranu wykonanym za pomocą np. systemowego narzędzia Wycinanie w systemie Windows lub narzędzi Linux i zapisz na zewnętrznym dysku USB jako JPEG z nazewnictwem podanym w poleceniu.

### C1 - Windows 10/11

Program z narzędziami dla CPU/Mobo (Procesor, Płyta Główna): W menu start wpisujemy "Informacje o Systemie" (`msinfo32`). Mamy podany procesor (np. Intel i5-7400), i Producent Płyty (Gigabyte).
Karta graficzna to odnoga "dxdiag" w polu `Uruchom` w zakładce Ekran. (Lub `devmgmt.msc` Menedżer Urządzeń).
Dysk to Menedżer Urządzeń `devmgmt.msc` i klikamy w Dyski podając jego model identyfikacyjny producenta. Ewentualnie z wiersza za pomocą składni WMIC.

### C2 - Linux Ubuntu Stacja

Program do wyświetlania na konsoli: `sudo lshw` i `lscpu`. Procesor widnieje na samym topie logów. Model płyty w zakładce z Motherboard i UUID, karta poprzez wyodrębnienie Display (`lshw -class display` / `lspci | grep VGA`).
Do tego przydadzą się polecenia typu:

- Procesor: `cat /proc/cpuinfo`
- Płyta Główna: `sudo dmidecode -t baseboard`
- Karta i dyski (`sudo lshw -class disk` lub `lsblk -o name,model,size`).

Wpisanie parametrów na kartę to konieczność odnotowania:

- CPU: Intel Core i5
- Płyta: MSI B250M
- Grafika: NVIDIA GeForce 1050
- Dysk: SEAGATE (Lub wpis z modelu z `WDC / ST`).
