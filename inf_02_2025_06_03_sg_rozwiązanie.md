# Rozwiązanie egzaminu INF.02 - Sesja Czerwiec 2025, wersja arkusza 03 SG

## 1. Wykonaj montaż okablowania sieciowego

**Rozwiązanie:**
Zgodnie z wytycznymi w pliku `README.md` (Zasada A), pomijamy zadania dotyczące fizycznego montażu okablowania sieciowego.
W warunkach egzaminacyjnych należałoby przygotować patchcord UTP/STP z wtykami RJ-45 zarobionymi z obu stron zgodnie ze standardem T568B (Biało-Pomarańczowy, Pomarańczowy, Biało-Zielony, Niebieski, Biało-Niebieski, Zielony, Biało-Brązowy, Brązowy). Następnie okablowanie musiałoby zostać sprawdzone testerem.

## 2. Konfiguracja rutera (MikroTik)

**Rozwiązanie:**
Ruter konfigurujemy z wykorzystaniem interfejsu wiersza poleceń (CLI) w systemie RouterOS.

**Konfiguracja CLI:**

```routeros
# Zmiana hasła, jeśli jest wymuszona przez domyślną konfigurację
/user set admin password=NoweHaslo!

# 1. Adres IP interfejsu LAN: 192.168.1.1/24 (np. na ether1 lub bridge'u powiązanym z LAN)
/ip address add address=192.168.1.1/24 interface=ether1

# 2. Wyłączenie serwera DHCP
/ip dhcp-server remove [find]

# 3. Włączenie obsługi VLAN 802.1Q oraz utworzenie interfejsu VLAN o ID=2 na porcie 2 (tryb trunk)
/interface vlan add name=vlan2 vlan-id=2 interface=ether2
# 4. Adres IP dla interfejsu VLAN o ID=2
/ip address add address=172.16.100.100/24 interface=vlan2

# 5. Utworzenie interfejsu VLAN o ID=3 na porcie 2 (tryb trunk)
/interface vlan add name=vlan3 vlan-id=3 interface=ether2
# 6. Adres IP dla interfejsu VLAN o ID=3
/ip address add address=192.168.0.100/24 interface=vlan3

# 7. Włączenie routingu między sieciami VLAN
# (W systemie RouterOS, po nadaniu adresów IP na interfejsach VLAN, routing między nimi jako sieciami połączonymi bezpośrednio jest realizowany automatycznie i dodawany do tablicy routingu).
```

**Odczyt konfiguracji rutera (Weryfikacja):**

```routeros
# Sprawdzenie utworzonych interfejsów VLAN:
/interface vlan print
# Sprawdzenie zaadresowania IP interfejsów:
/ip address print
# Sprawdzenie tablicy routingu:
/ip route print
# Weryfikacja braku serwera DHCP:
/ip dhcp-server print
```

## 3. Konfiguracja przełącznika (MikroTik)

**Rozwiązanie:**
Przełącznik konfigurujemy, definiując w pierwszej kolejności bridge z VLAN filteringiem.

**Konfiguracja CLI:**

```routeros
# Utworzenie bridge'a
/interface bridge add name=bridge1 vlan-filtering=no

# Przypisanie portów do bridge'a ze zdefiniowanymi PVID dla trybów dostępu (untagged)
# Port 1 (Trunk) pozostaje z pvid=1 domyślnie, port 2 (VLAN 2), porty 3 i 4 (VLAN 3)
/interface bridge port add bridge=bridge1 interface=ether1
/interface bridge port add bridge=bridge1 interface=ether2 pvid=2
/interface bridge port add bridge=bridge1 interface=ether3 pvid=3
/interface bridge port add bridge=bridge1 interface=ether4 pvid=3

# Przypisywanie tagowania i nietagowania na moście
# VLAN ID=2 (Port 1 z tagowaniem, Port 2 bez tagowania)
/interface bridge vlan add bridge=bridge1 vlan-ids=2 tagged=ether1 untagged=ether2

# VLAN ID=3 (Port 1 z tagowaniem, Porty 3 i 4 bez tagowania)
/interface bridge vlan add bridge=bridge1 vlan-ids=3 tagged=ether1 untagged=ether3,ether4

# Włączenie filtrowania VLAN 802.1Q
/interface bridge set bridge1 vlan-filtering=yes

# Skonfigurowanie adresu IP dla przełącznika na głównym interfejsie bridge (lub dedykowanym VLANie zarządzania)
/ip address add address=192.168.1.144/24 interface=bridge1

# Skonfigurowanie bramy domyślnej
/ip route add dst-address=0.0.0.0/0 gateway=192.168.1.1
```

**Odczyt konfiguracji przełącznika (Weryfikacja):**

```routeros
# Sprawdzenie adresacji przełącznika i bramy
/ip address print
/ip route print
# Sprawdzenie ustawień portów względem bridge i PVID:
/interface bridge port print
# Sprawdzenie tablicy VLAN bridge'a (tagged/untagged):
/interface bridge vlan print
```

## 4. Połączenie urządzeń sieciowych

**Rozwiązanie:**
Zgodnie ze schematem urządzenia łączy się kablami (patchcordami). Porty dostępowe muszą zostać wpięte do urządzeń klienckich i serwera, a port trunk (np. port 1 switcha) do wyznaczonego portu na ruterze (ether2).

## 5. Konfiguracja serwera WWW/DNS

### D1 - Windows Server 2022

**Rozwiązanie:**

1. **Konfiguracja interfejsów:**
   - Wchodzimy do `ncpa.cpl`.
   - Zmieniamy nazwę pierwszej karty podłączonej do przełącznika (port 3) na `LAN_WWW`.
   - Zmieniamy nazwę drugiej karty (podłączonej do port 4) i całkowicie ją wyłączamy (prawy klik -> Wyłącz).
   - Otwieramy Właściwości -> Protokół IPv4 dla `LAN_WWW`.
   - Adres IP: `192.168.0.201` (przyjmując przykładowy numer stanowiska X=1).
   - Maska: `255.255.255.0`
   - Brama domyślna: `192.168.0.100`
   - Preferowany serwer DNS: `127.0.0.1` (localhost).

2. **Role Serwera i DNS:**
   - Z poziomu Menedżera Serwera wybieramy "Dodaj role i funkcje".
   - Instalujemy role "Serwer DNS" oraz "Serwer sieci Web (IIS)".
   - Otwieramy narzędzie menedżera DNS (`dnsmgmt.msc`).
   - Rozwijamy nazwę serwera -> "Strefy wyszukiwania do przodu" -> prawy przycisk -> "Nowa strefa...". Wybieramy strefę podstawową.
   - Nazwa strefy: `egzamin.local`. Zgadzamy się na utworzenie standardowego pliku strefy.
   - Po utworzeniu wchodzimy do strefy `egzamin.local`, klikamy prawym -> "Nowy host (A lub AAAA)...".
   - Nazwa: `www`, Adres IP: `192.168.0.201` (IP interfejsu LAN_WWW). Klikamy "Dodaj hosta".

3. **Pliki i konfiguracja WWW (IIS):**
   - W Eksploratorze Windows tworzymy nowy folder: `C:\www`.
   - Wchodzimy na pendrive egzaminacyjny `DOKUMENTACJA/PROGRAMY/PLIKI`, kopiujemy plik `index.html` do katalogu `C:\www`. Następnie zmieniamy jego nazwę na `egzamin.html`.
   - Otwieramy "Menedżer internetowych usług informacyjnych (IIS)" (`inetmgr`).
   - W lewym panelu rozwijamy serwer, klikamy na "Witryny" i "Dodaj witrynę internetową...".
   - Nazwa witryny: `egzamin`.
   - Ścieżka fizyczna: wybieramy folder `C:\www`.
   - Powiązanie: Typ `http`, Adres IP przypisany to `192.168.0.201` (LAN_WWW), Port `80`. Klikamy OK.
   - Następnie klikamy na utworzoną witrynę, w głównym panelu wybieramy "Dokument domyślny". Dodajemy nową pozycję `egzamin.html` i przesuwamy ją na samą górę, by miała najwyższy priorytet. Opcjonalnie można zatrzymać stary "Default Web Site" działający na tym samym porcie i IP.

**Polecenia do weryfikacji ustawień (PowerShell):**

```powershell
# Weryfikacja IP
Get-NetIPAddress -InterfaceAlias LAN_WWW
Get-NetAdapter | Format-Table Name, Status

# Weryfikacja DNS
Resolve-DnsName -Name www.egzamin.local

# Weryfikacja stron IIS (przy zainstalowanym module WebAdministration)
Import-Module WebAdministration
Get-Website
```

---

### D2 - Linux Ubuntu Server

**Rozwiązanie:**

1. **Konfiguracja sieci (`netplan` lub `nmcli`):**
   - Identyfikujemy interfejsy np. `ens33` i `ens34`.
   - Zmieniamy nazwę na `LAN_WWW` (np. przez reguły udev) lub nazywamy interfejs w pliku konfiguracyjnym Netplan `/etc/netplan/01-netcfg.yaml`:

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
      dhcp4: false
      activation-mode: off # Odpowiednik wyłączenia drugiej karty
```

- Aktywujemy zmiany: `sudo netplan apply`.

1. **Instalacja usług DNS (bind9) i WWW (apache2 lub nginx):**
   - `sudo apt update && sudo apt install bind9 bind9utils bind9-doc apache2`

2. **Konfiguracja DNS (bind9):**
   - W pliku `/etc/bind/named.conf.local` dodajemy strefę do przodu:

```text
zone "egzamin.local" {
    type master;
    file "/etc/bind/db.egzamin.local";
};
```

- Kopiujemy podstawowy plik strefy i go edytujemy:
     `sudo cp /etc/bind/db.local /etc/bind/db.egzamin.local`
     Edycja `/etc/bind/db.egzamin.local` (aktualizacja SOA, dodanie rekordu www):

```text
$TTL    604800
@       IN      SOA     localhost. root.localhost. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      localhost.
www     IN      A       192.168.0.201
```

- Restart usługi DNS: `sudo systemctl restart bind9`.

1. **Konfiguracja WWW (Apache):**
   - `sudo mkdir /www`
   - Kopiujemy plik i zmieniamy nazwę: `sudo cp /media/pendrive/DOKUMENTACJA/PROGRAMY/PLIKI/index.html /www/egzamin.html`
   - Definiujemy nową stronę w `/etc/apache2/sites-available/egzamin.conf`:

```apache
<VirtualHost 192.168.0.201:80>
    ServerName egzamin.local
    DocumentRoot /www
    DirectoryIndex egzamin.html
</VirtualHost>
```

- Nadajemy prawa odczytu katalogowi `/www`: `sudo chmod -R 755 /www`.
- Wyłączamy domyślną stronę i włączamy nową:
     `sudo a2dissite 000-default.conf`
     `sudo a2ensite egzamin.conf`
- Restart Apache: `sudo systemctl reload apache2`.

**Polecenia do weryfikacji ustawień (Bash):**

```bash
ip addr show LAN_WWW
ip addr show ens34 # powinno być wyłączone
dig @127.0.0.1 www.egzamin.local
systemctl status bind9 apache2
```

## 6. Konfiguracja stacji roboczej

### C1 - Windows 10/11

**Rozwiązanie:**

1. **Adresacja IP dla LAN_SR:**
   - W narzędziu `ncpa.cpl` zmieniamy nazwę włączonej karty sieciowej na `LAN_SR`.
   - Właściwości -> Protokół IPv4:
     - IP: `172.16.100.10`
     - Maska: `255.255.255.0`
     - Brama: `172.16.100.100`
     - DNS: `192.168.0.201` (adres IP serwera)

2. **Wymóg logowania CTRL+ALT+DEL oraz komunikat ekranowy:**
   - Otwieramy narzędzie Zasad Zabezpieczeń Lokalnych wciskając `Win + R` i wpisując `secpol.msc`.
   - Rozwijamy: "Zasady lokalne" -> "Opcje zabezpieczeń".
   - Znajdujemy polisę: `Logowanie interakcyjne: nie wymagaj naciśnięcia klawiszy CTRL+ALT+DEL`. Otwieramy i zmieniamy jej stan na **Wyłączone** (co uaktywnia wymóg ich naciskania).
   - Następnie znajdujemy dwie powiązane polisy:
     - `Logowanie interakcyjne: tekst komunikatu dla użytkowników próbujących się zalogować`.
     - `Logowanie interakcyjne: tytuł komunikatu dla użytkowników próbujących się zalogować`.
   - W polu tekstu (zgodnie z zadaniem) wpisujemy: `Uwaga! Logujesz się do komputera egzaminacyjnego!`. (Tytuł można zostawić pusty, wpisać spację, lub dowolny inny zwrot jak "Informacja").

3. **Wyświetlenie strony internetowej:**
   - Otwieramy dowolną przeglądarkę (np. Microsoft Edge) i wpisujemy w pasek adresu: `http://www.egzamin.local`. Po wczytaniu się zawartości dokumentu `egzamin.html`, można poprosić egzaminatora.

**Polecenia do weryfikacji ustawień (CMD/PowerShell):**

```powershell
ipconfig /all
# Ustawienia logowania i CTRL+ALT+DEL widoczne są w rejestrze
Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "DisableCAD"
Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "legalnoticecaption"
Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "legalnoticetext"
```

---

### C2 - Linux Ubuntu Stacja

**Rozwiązanie:**

1. **Adresacja IP dla interfejsu stacji:**
   - W terminalu modyfikujemy aktywne połączenie:
   - `nmcli connection modify "Połączenie przewodowe 1" connection.id LAN_SR`
   - `nmcli connection modify LAN_SR ipv4.addresses 172.16.100.10/24 ipv4.gateway 172.16.100.100 ipv4.dns 192.168.0.201 ipv4.method manual`
   - `nmcli connection up LAN_SR`

2. **Wymóg interaktywnych skrótów i komunikat na ekranie powitalnym (GDM):**
   - Systemy Linux z menedżerem okien GDM3 nie mają natywnego wymogu Ctrl+Alt+Del dla wszystkich ekranów, tak jak Windows. By zrealizować wyświetlanie "Banera" (komunikatu ostrzegawczego) przed zalogowaniem wpisujemy w terminalu jako użytkownik z prawami roota:
   - Dodajemy wiadomość dla SSH/Terminali w `/etc/issue.net` oraz `/etc/issue`
     `echo "Uwaga! Logujesz się do komputera egzaminacyjnego!" | sudo tee /etc/issue.net /etc/issue`
   - W przypadku GDM (ekran graficzny) modyfikujemy plik profilu GDM: w katalogu `/etc/dconf/profile/gdm` upewniamy się, że istnieje wpis, a do bazy wprowadzamy dconf:
     Tworzymy plik banera `/etc/dconf/db/gdm.d/01-banner-message`:

```text
[org/gnome/login-screen]
banner-message-enable=true
banner-message-text='Uwaga! Logujesz się do komputera egzaminacyjnego!'
```

- Aktualizujemy dconf systemowy wykonując: `sudo dconf update`. Po wylogowaniu się, nad listą użytkowników pojawi się ten tekst.

1. **Wyświetlenie witryny:**
   - W przeglądarce (np. Firefox) wchodzimy na stronę `http://www.egzamin.local`. Pamiętamy o uprzednim odblokowaniu reguł ufw lub wyczyszczeniu resolvera przeglądarki jeśli są z nim problemy (DNS via HTTPS powinno być w FF na czas zadania wyłączone).

**Polecenia do weryfikacji ustawień (Bash):**

```bash
nmcli dev show
cat /etc/dconf/db/gdm.d/01-banner-message
curl -I http://www.egzamin.local
```

## 7. Test komunikacji

**Rozwiązanie:**
Aby serwer (np. 192.168.0.201) swobodnie komunikował się ze stacją roboczą z systemem Windows (172.16.100.10), należy upewnić się, czy zapora stacji roboczej pozwala na wchodzące żądania ping (ICMPv4 Echo).

- Windows: `wf.msc` -> Reguły ruchu przychodzącego -> zlokalizować "Udostępnianie plików i drukarek (Żądanie Echa - ruch przychodzący ICMPv4)" i wybrać opcję Włącz regułę.
- Linux (UFW): `sudo ufw allow icmp` (domyślnie ufw często wpuszcza ruch ICMP, ale poleceniem można upewnić sie co do innych ruchów).

Z poziomu serwera wykonujemy test wykorzystując Wiersz poleceń lub Terminal z następującymi pakietami:

```cmd
ping 172.16.100.10     (Stacja robocza)
ping 192.168.1.1       (Router interfejs główny)
ping 192.168.0.200     (Drukarka)
```

Po uzyskaniu poprawnych zwrotów zgłaszamy fakt przewodniczącemu.

## 8. Identyfikacja podzespołów

Zadanie wymaga identyfikacji za pomocą narzędzi wbudowanych dla stacji roboczej (podane jako Windows, ale załączamy dla obu, na wypadek egzaminu C2). Należy sporządzić dokumentację zrzutami ekranowymi.

### C1 - Windows 10/11

Odczytu dokonujemy za pomocą graficznych aplikacji wbudowanych lub komend CMD.

- **CPU:** Menedżer Zadań (`taskmgr`) w zakładce "Wydajność" -> "Procesor" (widnieje w prawym górnym rogu) lub w CMD komenda: `wmic cpu get name`.
- **Płyta główna (Producent i model):** Narzędzie "Informacje o systemie" (`msinfo32.exe`) -> "Producent płyty głównej", "Produkt płyty głównej" (Model) lub CMD: `wmic baseboard get product,Manufacturer`.
- **Karta graficzna:** Narzędzie "Narzędzie diagnostyczne DirectX" (`dxdiag.exe`), zakładka Ekran, pole Nazwa i Producent. Ewentualnie w CMD: `wmic path win32_VideoController get name`.
- **Dysk twardy:** "Zarządzanie dyskami" (`diskmgmt.msc`), po kliknięciu prawym na dysk zerowy, wybierając właściwości, mamy podany jego identyfikator (np. WDC WD500...). Można również użyć Menedżera Urządzeń (`devmgmt.msc`) w sekcji Dyski, lub z CMD: `wmic diskdrive get model`.

Wyniki przypisujemy do Tabeli nr 1 w dokumencie egzaminacyjnym. Zrzuty każdego okienka przenosimy na dysk USB o nazwie `Egzamin-x`.

### C2 - Linux Ubuntu Stacja

W systemie można z powodzeniem wykorzystać wbudowane graficzne narzędzia (jak Informacje o systemie w Ustawieniach) lub program `lshw`.

- **Z wiersza poleceń:**
  - CPU: `lscpu` (Pole "Model name").
  - Płyta Główna: `sudo dmidecode -t baseboard` (Pole "Manufacturer" i "Product Name").
  - Karta graficzna: `lspci | grep VGA` (Wyświetla nazwę oraz producenta rdzenia).
  - Dysk twardy: `sudo lshw -class disk` lub `lsblk -d -o name,model`.

Aplikacje okienkowe dla Linuksa można również zrzucić za pomocą aplikacji Screenshot i odpowiednio zapisać na USB.
Zidentyfikowane parametry należy zapisać w dostarczonej Tabeli nr 1 na arkuszu egzaminacyjnym, np.:

- CPU: Intel Core i5-10400F
- Płyta główna: Gigabyte B460M DS3H
- Karta graficzna: NVIDIA GeForce GTX 1660
- Dysk twardy: Kingston SA400S37240G

## 9. Harmonogram prac

**Rozwiązanie:**
W wybranym darmowym lub komercyjnym programie jak Excel albo LibreOffice Calc realizujemy układ zgodny z tabelą 2 na arkuszu.

1. Wypełniamy strukturę danymi pod nagłówkiem `Nazwa czynności` oraz czas w formacie minut w kolumnie `Czas wykonania [min]`. Jeżeli np. Zadanie z Montażem okablowania nie miało miejsca to stawiamy `0`.
2. W podsumowaniu dla rzędu `RAZEM:` korzystamy z reguły logicznej zliczania komórek powyżej: `=SUMA(B2:B7)`.
3. Aktywujemy formatowanie warunkowe dla tejże komórki:
   - Formuła: mniejsze lub równe `150`. Format -> Karta Wypełnienie kolorem Niebieskim.
   - Nowa reguła komórki: Formuła: większe od `150`. Format -> Karta Wypełnienie kolorem Zielonym.
4. Zapisujemy zrzut pracy w pliku `harmonogram` do odpowiedniego formatu i wkładamy go na pamięć przenośną (USB).
