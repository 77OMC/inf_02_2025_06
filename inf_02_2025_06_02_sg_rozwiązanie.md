# Rozwiązanie egzaminu INF.02 - Sesja Czerwiec 2025, wersja arkusza 02 SG

## 1. Wykonaj modernizację stacji roboczej

**Rozwiązanie:**
Należy wyłączyć stację roboczą i odłączyć kabel zasilający. Po upewnieniu się, że komputer jest pozbawiony napięcia, zdjąć panel obudowy. Zlokalizować na płycie głównej wolny bank pamięci RAM. Odchylić zaczepy zabezpieczające po obu stronach (lub z jednej, w zależności od modelu płyty) wolnego banku. Wyrównać moduł pamięci RAM do wcięcia w złączu (co zapobiega jego odwrotnemu włożeniu), a następnie docisnąć równomiernie z obu stron aż do samoczynnego zatrzaśnięcia się zaczepów. Zamknąć obudowę, podłączyć okablowanie i włączyć system.

## 2. Identyfikacja parametrów pamięci RAM

Zgodnie z poleceniem identyfikacja odbywa się na systemie Linux, jednakże w związku z instrukcją podano także alternatywę Windows.

### C1 - Windows 10/11

Do identyfikacji pamięci z wiersza poleceń można użyć PowerShell lub CMD.

**Polecenia do odczytu ustawień (Weryfikacja):**

```cmd
wmic memorychip get capacity, memorytype, speed, manufacturer, partnumber, serialnumber
```

Alternatywa w programie PowerShell:

```powershell
Get-CimInstance Win32_PhysicalMemory | Format-Table Capacity, MemoryType, Speed, Manufacturer, PartNumber, SerialNumber
```

- Pojemność: Wynik `Capacity` podany jest w bajtach (w celu zamiany na GB należy go podzielić przez 1024^3).
- Standard: Zwykle `MemoryType` w postaci liczbowej (np. 26 oznacza DDR4, 24 oznacza DDR3).
- Częstotliwość: Parametr `Speed` wyrażony w MHz.
- Producent: `Manufacturer`.
- Numer seryjny/produktu: `SerialNumber` / `PartNumber`.

**Zrzut ekranu:**
W Windows można go wykonać używając wbudowanego "Narzędzia Wycinanie" lub klawisza `PrtScn`, a sam plik zapisać w formacie `.jpg` pod nazwą `ram.jpg` na nośniku USB.

### C2 - Linux Ubuntu Stacja

W systemie Linux wykorzystuje się narzędzie `dmidecode` lub `lshw`.

**Polecenia do odczytu ustawień (Weryfikacja):**

```bash
sudo dmidecode --type memory
```

Z wyjścia komendy odczytujemy:

- Pojemność: Wartość pola `Size`.
- Standard: Wartość pola `Type` (np. DDR4).
- Częstotliwość: Wartość pola `Speed` (w MHz).
- Producent: Wartość pola `Manufacturer`.
- Numer seryjny/produktu: `Serial Number` lub `Part Number`.

**Zrzut ekranu:**
Za pomocą klawisza `PrtScn` lub aplikacji `gnome-screenshot` należy zapisać plik graficzny jako `ram.jpg` na pamięć przenośną (USB).

## 3. Konfiguracja rutera (MikroTik)

**Rozwiązanie:**
Routery firmy MikroTik konfiguruje się z wykorzystaniem wiersza poleceń (CLI) lub GUI (WinBox). Zakładając logowanie z ustawień fabrycznych.

**Konfiguracja CLI:**

```routeros
# Zmiana hasła (jeżeli urządzenie wymusi to po zalogowaniu)
/user set admin password=NoweHaslo123

# Adres IP interfejsu WAN (200.200.200.9/28), np. port ether1
/ip address add address=200.200.200.9/28 interface=ether1

# Brama domyślna interfejsu WAN (200.200.200.1)
/ip route add dst-address=0.0.0.0/0 gateway=200.200.200.1

# Serwer DNS interfejsu WAN (4.4.4.4 oraz 8.8.8.8)
/ip dns set servers=4.4.4.4,8.8.8.8 allow-remote-requests=yes

# Adres IP interfejsu LAN (192.168.0.1/24), np. port ether2
/ip address add address=192.168.0.1/24 interface=ether2

# Wyłączony serwer DHCP - domyślnie DHCP może być przypisane do bridge'a, w takim przypadku go usuwamy
/ip dhcp-server remove [find]
```

**Odczyt konfiguracji rutera (Weryfikacja):**

```routeros
# Sprawdzenie interfejsów IP i maski:
/ip address print

# Sprawdzenie bramy (routingu):
/ip route print

# Sprawdzenie konfiguracji DNS:
/ip dns print

# Sprawdzenie czy usługa DHCP nie jest aktywna:
/ip dhcp-server print
```

## 4. Konfiguracja przełącznika (MikroTik)

**Rozwiązanie:**
Skonfigurujemy zarządzalny przełącznik MikroTik z wykorzystaniem najnowszej formy `Bridge VLAN Filtering`.

**Konfiguracja CLI:**

```routeros
# 1. Utworzenie mostu i portów
/interface bridge add name=bridge1 vlan-filtering=no
/interface bridge port add bridge=bridge1 interface=ether1 pvid=200
/interface bridge port add bridge=bridge1 interface=ether2 pvid=200
/interface bridge port add bridge=bridge1 interface=ether3 pvid=200
/interface bridge port add bridge=bridge1 interface=ether4 pvid=200

# 2. Utworzona sieć VLAN o ID=200 i porty przypisane bez tagowania (untagged)
/interface bridge vlan add bridge=bridge1 vlan-ids=200 untagged=ether1,ether2,ether3,ether4

# 3. Włączenie filtrowania VLAN na utworzonym moście
/interface bridge set bridge1 vlan-filtering=yes

# 4. Adres IP dla przełącznika L3 (192.168.0.2/24) przypisany do interfejsu VLAN 200
/interface vlan add name=vlan200 interface=bridge1 vlan-id=200
/ip address add address=192.168.0.2/24 interface=vlan200

# 5. Brama domyślna: adres IP rutera
/ip route add dst-address=0.0.0.0/0 gateway=192.168.0.1
```

**Odczyt konfiguracji przełącznika (Weryfikacja):**

```routeros
# Sprawdzenie interfejsu IP i adresu:
/ip address print
# Sprawdzenie bramy domyślnej:
/ip route print
# Sprawdzenie portów przypisanych do VLAN:
/interface bridge port print
# Sprawdzenie VLANów na bridge:
/interface bridge vlan print
```

## 5. Połączenie urządzeń sieciowych

**Rozwiązanie:**
Zgodnie z wytycznymi A (pomijamy zadanie dotyczące montażu okablowania sieciowego na etapie fizycznym). Urządzenia powinny być w rzeczywistości połączone kablami skrętką z wtykami RJ-45 wedle podanego na arkuszu schematu.

## 6. Konfiguracja serwera

### D1 - Windows Server 2022

**Rozwiązanie:**

1. **Nazwa połączeń i interfejs LAN12 (NIC Teaming):**
   - W menedżerze kart sieciowych (`ncpa.cpl`) zmieniamy nazwy fizycznych interfejsów na `LAN1` i `LAN2`.
   - Z "Menedżera serwera" -> "Serwer lokalny" wybieramy opcję tworzenia "Zespołu kart sieciowych".
   - W nowym oknie tworzymy nowy zespół o nazwie `LAN12` i przypisujemy do niego karty `LAN1` i `LAN2`.
   - W zakładce Właściwości Dodatkowe wybieramy Tryb: "Niezależny od przełącznika", Równoważenie obciążenia: "Dynamiczny". Wszystkie karty w sekcji stanu powinny być oznaczone jako aktywne.

2. **Adresacja IP dla nowej karty LAN12:**
   - W oknie `ncpa.cpl` wchodzimy we właściwości interfejsu zespołu `LAN12`. W protokole IPv4 podajemy:
     - Adres IP: `192.168.0.4`
     - Maska: `255.255.255.0`
     - Brama domyślna: `192.168.0.1`
     - Serwer DNS: `127.0.0.1` (localhost)

3. **Nazwa komputera:**
   - Klikając prawym na Start -> System -> Zmień nazwę tego komputera. Wprowadzamy `SERWER-XX`. Konieczny restart.

4. **Instalacja i konfiguracja Active Directory:**
   - Poprzez Menedżera serwera -> "Dodaj role i funkcje" wybieramy rolę "Usługi domenowe Active Directory".
   - Po instalacji promujemy serwer na kontroler domeny.
   - Wybieramy opcję: "Dodaj nowy las" i wpisujemy domenę główną: `informatyk.local`.
   - W sekcji opcji kontrolera wpisujemy hasło DSRM: `ZAQ!2wsx`. Proces zakończy się po restarcie komputera.

5. **Organizacja i użytkownik:**
   - Wchodzimy do "Użytkownicy i komputery usługi Active Directory" (`dsa.msc`).
   - Prawym na `informatyk.local` -> Nowy -> Jednostka Organizacyjna. Nazwa: `Serwisanci`.
   - Prawym na `Serwisanci` -> Nowy -> Użytkownik. Nazwa, imię, nazwisko: `Adam Abacki`, Login: `abackia`.
   - Hasło logowania to `12!@EWSX`. Ustawiamy, aby hasło nigdy nie wygasało.

6. **Zasób profilowy i uprawnienia:**
   - Na dysku z systemem (lub innym) tworzymy folder `C:\Profile`.
   - Prawym kliknięciem -> Właściwości -> Udostępnianie -> Zaawansowane udostępnianie -> "Udostępnij folder".
   - Nadajemy nazwę udziału jako `profile$`.
   - W "Uprawnieniach" sieciowych (Share permissions) usuwamy wszystkich, zostawiamy i dajemy pełną kontrolę dla `Wszyscy`.
   - W zakładce "Zabezpieczenia" (NTFS permissions) w "Zaawansowane" wyłączamy dziedziczenie (kopiując wpisy). Usuwamy wszystko, pozostawiając tylko uprawnienia do "Pełnej Kontroli" dla `Administratorzy` oraz `Użytkownicy`.

7. **Profil mobilny dla konta Adam Abacki:**
   - Otwieramy właściwości użytkownika `abackia` w `dsa.msc`, w zakładce "Profil", ścieżka profilu to:
     `\\SERWER-XX\profile$\%USERNAME%`

**Weryfikacja parametrów (PowerShell):**

```powershell
Get-NetLbfoTeam
Get-NetIPAddress -InterfaceAlias LAN12
hostname
Get-ADDomain
Get-ADUser -Identity abackia
Get-SmbShare -Name profile$
Get-Acl -Path C:\Profile
```

---

### D2 - Linux Ubuntu Server

**Rozwiązanie:**

1. **Utworzenie połączenia NIC Teaming (Bonding LAN1 i LAN2 do LAN12):**
   - Należy zidentyfikować interfejsy (np. `ip a` pokaże m.in. `ens33` i `ens34`).
   - Modyfikujemy adresację w programie konfiguracyjnym Netplan (plik np. `/etc/netplan/01-netcfg.yaml`):

```yaml
network:
  version: 2
  renderer: networkd
  bonds:
    LAN12:
      interfaces: [ens33, ens34]  # Tu podmieniamy właściwe nazwy
      addresses:
        - 192.168.0.4/24
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses:
          - 127.0.0.1
      parameters:
        mode: balance-alb
```

- *Tryb `balance-alb` to odpowiednik "Niezależnego od przełącznika, dynamicznego obciążenia".*
- Po edycji wykonujemy polecenie: `sudo netplan apply`.

1. **Nazwa komputera (Hostname):**
   - Komendą: `sudo hostnamectl set-hostname SERWER-XX`. Dodatkowo, modyfikujemy `/etc/hosts` dopisując:
     `192.168.0.4 SERWER-XX.informatyk.local SERWER-XX`

2. **Active Directory w systemie Linux (Samba AD DC):**
   - Pobieramy niezbędne pakiety za pomocą `sudo apt update && sudo apt install -y samba smbclient krb5-user winbind libpam-winbind libnss-winbind`.
   - Zatrzymujemy tymczasowo Sambę i ukrywamy dotychczasową konfigurację: `sudo systemctl stop smbd nmbd winbind` oraz `sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak`.
   - Provisioning kontrolera AD: `sudo samba-tool domain provision --use-rfc2307 --realm=INFORMATYK.LOCAL --domain=INFORMATYK --adminpass=ZAQ!2wsx`.
   - Podmieniamy pliki konfiguracyjne np. Kerberosa: `sudo cp /var/lib/samba/private/krb5.conf /etc/`. Uruchamiamy usługę: `sudo systemctl unmask samba-ad-dc && sudo systemctl enable --now samba-ad-dc`.

3. **Konta, struktura (OU) w Active Directory oraz profil mobilny (Samba Tool):**
   - Dodajemy jednostkę `Serwisanci`:
     `sudo samba-tool ou create "OU=Serwisanci,DC=informatyk,DC=local"`
   - Tworzymy konto wewnątrz OU z hasłem `12!@EWSX`:
     `sudo samba-tool user create abackia 12!@EWSX --given-name=Adam --surname=Abacki --userou="OU=Serwisanci"`

4. **Folder udostępniony i profil:**
   - Katalog lokalny: `sudo mkdir -p /Profile` oraz poprawa uprawnień NTFS pod Linux: `sudo chmod 777 /Profile`.
   - Dodanie udostępnionego katalogu `profile$` z pełną kontrolą na końcu w edytowanym pliku `/etc/samba/smb.conf`:

```ini
[profile$]
    path = /Profile
    read only = no
```

- Zmiana profilu w konfiguracji AD:
     `sudo samba-tool user setprofilepath abackia "\\SERWER-XX\profile$\%USERNAME%"`
- Akceptacja pliku konfiguracji: `sudo smbcontrol all reload-config`.

**Weryfikacja parametrów (Bash):**

```bash
cat /proc/net/bonding/LAN12
ip a show LAN12
hostnamectl
sudo samba-tool user show abackia
testparm -s
```

## 7. Konfiguracja stacji roboczej

### C1 - Windows 10/11

**Rozwiązanie:**

1. **Konfiguracja sieci (IP, Brama, DNS) dla LAN3:**
   - W narzędziu Połączeń Sieciowych `ncpa.cpl` zmieniamy najpierw nazwę wykorzystywanej karty sieciowej na `LAN3`.
   - Otwieramy Właściwości -> Protokół IPv4, następnie wprowadzamy stałe ustawienia:
     - IP: `192.168.0.3`
     - Maska: `255.255.255.0`
     - Brama: `192.168.0.1`
     - DNS (Adres IP lokalnego serwera z zadania): `192.168.0.4`

2. **Zmiana nazwy i dodanie do domeny:**
   - Prawym przyciskiem myszy na Start -> System -> "Zaawansowane ustawienia systemu", zakładka "Nazwa komputera".
   - Klikamy "Zmień" -> Nazwa komputera to `SERWIS`. Wybieramy przynależność do Domeny jako `informatyk.local`. Zostaniemy poproszeni o uwierzytelnienie konta administratora sieci: `Administrator` i hasło z zadania. Uruchamiamy ponownie system.

3. **Pliki i szyfrowanie EFS (Encrypting File System):**
   - Po restarcie i zalogowaniu lokalnym, tworzymy katalog `C:\PLIKI`.
   - Odczytujemy zdjęcie na nośniku instalatora z odpowiedniego katalogu i wklejamy (kopiujemy) plik `tapeta.jpg` do naszej nowo założonej ścieżki.
   - Prawym przyciskiem myszy klikamy na skopiowany plik `tapeta.jpg` -> Właściwości.
   - Wybieramy zakładkę Ogólne -> przycisk "Zaawansowane". Zaznaczamy w Atrybutach opcję "Szyfruj zawartość, aby zabezpieczyć dane". Na oknie z pytaniem klikamy na zastosowanie tylko i wyłącznie do tego pliku (nie dla folderu).

4. **Logowanie domenowe:**
   - Wylogowujemy się w Menu Start, na okienku logowania dajemy "Inny użytkownik".
   - Wprowadzamy konto `abackia` oraz hasło. Potwierdzamy, że pliki profilowe (Roaming) pobierają się poprawnie z Serwera. Po zalogowaniu do pulpitu wylogowujemy się ponownie.

**Weryfikacja parametrów (CMD/PowerShell):**

```powershell
ipconfig /all
systeminfo | findstr /B /C:"Domain" /C:"Host Name"
cipher /c C:\PLIKI\tapeta.jpg
```

---

### C2 - Linux Ubuntu Stacja

**Rozwiązanie:**

1. **Adresacja IP, DNS, Brama z narzędzia nmcli w przypadku karty podłączonej do Switcha i zmiana nazwy na LAN3:**
   - W terminalu podnosimy profil połączenia o nazwie `LAN3` przy użyciu wbudowanego menedżera.
   - `nmcli connection modify <stara_nazwa> connection.id LAN3`
   - `nmcli connection modify LAN3 ipv4.addresses 192.168.0.3/24 ipv4.gateway 192.168.0.1 ipv4.dns 192.168.0.4 ipv4.method manual`
   - Uruchomienie połączenia wprowadzonymi parametrami: `nmcli connection up LAN3`.

2. **Modyfikacja nazwy komputera:**
   - Wykorzystując komendę: `sudo hostnamectl set-hostname SERWIS`.

3. **Włączenie systemu do kontrolera AD serwera:**
   - Pobieramy paczki wymagane przy współpracy z AD z domyślnym logowaniem profilowym: `sudo apt install realmd sssd sssd-tools adcli samba-common-bin`.
   - Wykorzystujemy program z paczki the `realmd` logując sie poświadczeniami DC:
     `sudo realm join -U Administrator informatyk.local`. Wpisujemy hasło kontrolera.
   - Aktywujemy tworzenie domyślnych profili automatycznie przez PAM dodając linijkę w pliku konfiguracji:
     `echo "session optional pam_mkhomedir.so skel=/etc/skel umask=077" | sudo tee -a /etc/pam.d/common-session`

4. **Struktura katalogów i alternatywne szyfrowanie "EFS" plików (np. przez fscrypt lub komendę szyfrującą gpg):**
   - Narzędzie GnuPG to doskonała alternatywa szyfrowania na poziomie plików (atrybuty na Linux).
   - Katalog: `sudo mkdir /PLIKI`
   - Skopiowanie: `sudo cp /media/<sciezka_do_pendrive>/DOKUMENTACJA/PROGRAMY/PLIKI/tapeta.jpg /PLIKI/tapeta.jpg`
   - Szyfrowanie pliku programem: `gpg -c /PLIKI/tapeta.jpg` co utworzy zahasłowany i zaszyfrowany kluczem plik typu `.gpg`. Stary niezaszyfrowany można usunąć komendą `sudo rm /PLIKI/tapeta.jpg`.

5. **Logowanie z konta:**
   - By potwierdzić autoryzację SSSD można zmienić konto logując się na domenę: `su - abackia@informatyk.local`, a następnie wrócić `exit`. Oraz spróbować bezpośrednio z menedżera graficznego środowiska.

**Weryfikacja parametrów (Bash):**

```bash
nmcli dev show
hostnamectl
realm list
ls -l /PLIKI/
```

## 8. Test komunikacji

**Rozwiązanie:**
W przypadku stacji roboczej opartej o system operacyjny Windows należy bezwzględnie dopuścić do odpowiedzi pakietów ping na stacji zapory Windows Defender z innej maszyny w sieci z polecenia testowego.

- Windows (Stacja C1): Wywołać z wiersza poleceń `wf.msc`, co spowoduje włączenie Zaawansowanej zapory (Zabezpieczenia Zaaawansowane). W nowym polu wybrać "Reguły ruchu przychodzącego". Znaleźć obydwie pozycje dotyczące nazwy `Udostępnianie plików i drukarek (Żądanie Echa - ruch przychodzący)` dla sieci IPv4 i włączyć obie reguły tak, aby zaznaczona była zielona ikona "ptaszka".
- Linux (Stacja C2): Najczęściej odpowiedzi zwrotne oparte na pakietach ICMPv4 puszczane są domyślnie po instalacji.

Z wiersza poleceń wywołać bezpośrednio komendy:

```cmd
ping 192.168.0.2
ping 192.168.0.1
ping 192.168.0.3
```

Każda z operacji na maszynach powiązanych musi w sposób cykliczny odpowiadać czasem opóźnień komunikacyjnych z poprawnym wynikiem `TTL=xx`. Po pozytywnym zweryfikowaniu należy zgłosić gotowość w obecności egzaminatora, unosząc dłoń.

## 9. Harmonogram prac

**Rozwiązanie:**
Harmonogram zadań wprowadzamy do dowolnego arkusza kalkulacyjnego np. Microsoft Excel, upewniając się, aby jego zawartość stanowiła logiczne odtworzenie struktury zawartej pod poleceniami, w oparciu o Tabela nr 2 na arkuszu egzaminacyjnym.

Procedura:

1. Skopiowanie i ustrukturyzowanie zawartości w pliku wraz z ramkami. Utworzenie nagłówków m.in. kolumny `Czas wykonania [min]`
2. Wypełnienie na podstawie orientacyjnych danych w poszczególnych komórkach pod spodem. Czas poświęcony na każdy etap (np. Montaż - 15 minut; Serwer - 45 itd.).
3. Poniżej użycie zintegrowanej formuły auto-podsumowania tj. funkcją sumowania dla powyższych komórek np: `=SUMA(B2:B7)` w komórce z RAZEM (B8).
4. Następnie użycie mechanizmów na w/w komórce B8 z polecenia: Na komórce zaznaczamy Wstążka -> Narzędzia Główne -> Formatowanie Warunkowe -> Reguły Wyróżniania Komórek / Nowa Reguła...
5. Komórka RAZEM ustawiona z "Formatuj tylko komórki zawierające -> Wartość Komórki". Ustalamy Mniejszą lub równą "150". Karta formatuj: kolor wypełnienia Niebieski.
6. Ustalamy nową regułę dla B8 formatując warunkowo jako "Większą niż" jako `150`. Format i tło na zielono.
7. Zapisać i umieścić dokument zgodnie z żądaniem na zmontowanym napędzie pod opisywanym schematem t.j.: `harmonogram.xlsx` (Excel) lub `.ods` (Calc).
