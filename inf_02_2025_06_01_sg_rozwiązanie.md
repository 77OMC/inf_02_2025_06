# Rozwiązanie egzaminu INF.02 - Sesja Czerwiec 2025, wersja arkusza SG

## 1. Wykonaj modernizację stacji roboczej

**Rozwiązanie:**
Należy wyłączyć stację roboczą, odłączyć kabel zasilający. Zdjąć obudowę. Zlokalizować wolny slot pamięci RAM na płycie głównej. Odchylić zaczepy zabezpieczające po bokach slotu. Umieścić moduł pamięci RAM zgodnie z wycięciem w slocie (wcięcie zapobiega odwrotnemu montażowi). Docisnąć równomiernie z obu stron aż do zatrzaśnięcia się zaczepów. Zamknąć obudowę, podłączyć kable i uruchomić komputer.

## 2. Identyfikacja parametrów pamięci RAM

Zgodnie z poleceniem na egzaminie należy zidentyfikować parametry dla systemu Linux, natomiast poniżej znajduje się rozwiązanie dla obu systemów operacyjnych.

### C1 - Windows

Do sprawdzenia parametrów pamięci RAM można użyć systemowego Wiersza polecenia (cmd) lub programu PowerShell.

**Polecenia do odczytu ustawień (Weryfikacja):**

```cmd
wmic memorychip get capacity, memorytype, speed, manufacturer, partnumber, serialnumber
```

Alternatywnie w PowerShell:

```powershell
Get-CimInstance Win32_PhysicalMemory | Format-Table Capacity, MemoryType, Speed, Manufacturer, PartNumber, SerialNumber
```

- Pojemność (Capacity): Wartość podana w bajtach (należy przeliczyć na GB, dzieląc przez 1024^3).
- Standard: Zazwyczaj MemoryType (np. 20 to DDR, 24 to DDR3, 26 to DDR4).
- Częstotliwość (Speed): Wartość w MHz.
- Producent (Manufacturer)
- Numer seryjny (SerialNumber) / Numer produktu (PartNumber)

**Zrzut ekranu:**
Aby wykonać zrzut ekranu dokumentujący, można użyć narzędzia "Wycinanie" (Snipping Tool) lub nacisnąć `PrtScn`, a następnie wkleić i zapisać np. w programie Paint jako plik `ram.jpg` na nośniku USB.

### C2 - Linux Ubuntu Stacja

W systemie Linux wykorzystujemy komendę `dmidecode` lub `lshw`.

**Polecenia do odczytu ustawień (Weryfikacja):**

```bash
sudo dmidecode --type memory
```

Z wyników odczytujemy:

- Size: Pojemność
- Type: Standard (np. DDR4)
- Speed: Częstotliwość (MHz)
- Manufacturer: Producent
- Serial Number / Part Number: Numer seryjny/produktu

**Zrzut ekranu:**
W systemie Ubuntu można wykonać zrzut okna terminala aplikacją "Screenshot" lub klawiszem `PrtScn`. Plik graficzny należy nazwać `ram.jpg` i skopiować na pendrive.

## 3. Konfiguracja rutera (MikroTik)

**Rozwiązanie:**
Router MikroTik konfigurujemy z poziomu konsoli (CLI) lub WinBox.

**Konfiguracja CLI:**

```routeros
# Zmiana hasła (jeśli wymagana przy pierwszym logowaniu)
/user set admin password=NoweHaslo!

# 1. Adres IP interfejsu WAN (100.100.100.9/28), np. port ether1
/ip address add address=100.100.100.9/28 interface=ether1

# 2. Brama domyślna interfejsu WAN (100.100.100.1)
/ip route add dst-address=0.0.0.0/0 gateway=100.100.100.1

# 3. Serwer DNS interfejsu WAN (4.4.4.4, drugi 8.8.8.8)
/ip dns set servers=4.4.4.4,8.8.8.8 allow-remote-requests=yes

# 4. Adres IP interfejsu LAN (172.16.0.1/24), np. port ether2
/ip address add address=172.16.0.1/24 interface=ether2

# 5. Wyłączony serwer DHCP - brak konfiguracji serwera DHCP (domyślnie jest wyłączony, lub usuwamy jeśli istnieje).
/ip dhcp-server remove [find]
```

**Odczyt konfiguracji rutera MIKROTIK (Weryfikacja):**

```routeros
# Sprawdzenie adresacji IP:
/ip address print

# Sprawdzenie bramy domyślnej (routingu):
/ip route print

# Sprawdzenie serwerów DNS:
/ip dns print

# Sprawdzenie statusu serwerów DHCP:
/ip dhcp-server print
```

## 4. Konfiguracja przełącznika (MikroTik)

**Rozwiązanie:**
Zakładamy przełącznik L3 firmy MikroTik (Cloud Router Switch - CRS) pracujący z systemem RouterOS.

**Konfiguracja CLI:**

```routeros
# 1. Utworzenie sieci VLAN 802.1Q o ID=2
/interface vlan add name=vlan2 vlan-id=2 interface=bridge1
# Alternatywnie w nowym Bridge VLAN Filtering:
/interface bridge add name=bridge1 vlan-filtering=no
/interface bridge port add bridge=bridge1 interface=ether1 pvid=2
/interface bridge port add bridge=bridge1 interface=ether2 pvid=2
/interface bridge port add bridge=bridge1 interface=ether3 pvid=2
/interface bridge port add bridge=bridge1 interface=ether4 pvid=2
/interface bridge vlan add bridge=bridge1 vlan-ids=2 untagged=ether1,ether2,ether3,ether4

# Włączamy filtrowanie VLAN na bridge:
/interface bridge set bridge1 vlan-filtering=yes

# W tym przypadku IP nadajemy interfejsowi vlan przypisanemu do bridge'a
/interface vlan add name=vlan2 interface=bridge1 vlan-id=2

# 2. Adres IP dla przełącznika L3: 172.16.0.2/24 przypisany do VLAN o ID=2
/ip address add address=172.16.0.2/24 interface=vlan2

# 3. Brama domyślna: adres IP rutera
/ip route add dst-address=0.0.0.0/0 gateway=172.16.0.1
```

**Odczyt konfiguracji przełącznika MIKROTIK (Weryfikacja):**

```routeros
# Sprawdzenie interfejsów VLAN:
/interface vlan print
# Sprawdzenie portów w Bridge i ich PVID:
/interface bridge port print
# Sprawdzenie konfiguracji Bridge VLAN:
/interface bridge vlan print
# Sprawdzenie adresu IP:
/ip address print
# Sprawdzenie bramy:
/ip route print
```

## 5. Połączenie urządzeń sieciowych

**Rozwiązanie (zgodnie z instrukcją A: pomijamy to zadanie w warstwie fizycznej):**
*(Zadanie pominięte zgodnie z wytycznymi w pliku README)*

Gdyby było konieczne: Należy zgodnie ze schematem połączyć porty używając okablowania UTP/STP z wtykami RJ-45.

## 6. Konfiguracja serwera

### D1 - Windows Server 2022

**Rozwiązanie:**

1. **Nazwa połączeń i NIC Teaming:**
   - W systemie wciskamy `Win + R`, `ncpa.cpl`. Zmieniamy nazwy interfejsów na `LAN1` i `LAN2`.
   - W Menedżerze serwera wybieramy "Serwer lokalny", znajdujemy opcję "Tworzenie zespołu kart interfejsu sieciowego" (NIC Teaming) i klikamy.
   - W sekcji ZESPOŁY klikamy "Zadania" -> "Nowy zespół".
   - Nazwa zespołu: `LAN12`.
   - Zaznaczamy karty `LAN1` i `LAN2`.
   - Właściwości dodatkowe: Tryb tworzenia zespołu - "Niezależny od przełącznika" (Switch Independent), Tryb równoważenia obciążenia - "Dynamiczny", Karty rezerwowe - "Brak (wszystkie karty są aktywne)".

2. **Adresacja IP dla LAN12:**
   - W `ncpa.cpl` otwieramy właściwości interfejsu `LAN12` -> IPv4.
   - IP: `172.16.0.4`, Maska: `255.255.255.0` (dla /24)
   - Brama domyślna: `172.16.0.1`
   - Preferowany serwer DNS: `127.0.0.1` (localhost)

3. **Nazwa komputera:**
   - W Menedżerze serwera "Serwer lokalny", klikamy na nazwę komputera, zmieniamy na `SERWER-XX`. Restartujemy system.

4. **Usługa Active Directory i konfiguracja:**
   - Menedżer serwera -> "Dodaj role i funkcje" -> "Usługi domenowe Active Directory" -> Instaluj.
   - Po instalacji: "Promuj ten serwer na kontroler domeny".
   - Dodaj nowy las: domena `informatyk.local`.
   - Hasło trybu DSRM: `ZAQ!2wsx`. Reszta domyślnie, klikamy Instaluj.
   - System zrestartuje się.

5. **Użytkownicy i struktura organizacyjna:**
   - Otwieramy `dsa.msc` (Użytkownicy i komputery usługi Active Directory).
   - Klikamy prawym na domenę `informatyk.local` -> Nowy -> Jednostka organizacyjna. Nazwa: `Serwisanci`.
   - Prawym na `Serwisanci` -> Nowy -> Użytkownik.
   - Imię: Adam, Nazwisko: Abacki, Nazwa logowania: `abackia`.
   - Hasło: `12!@EWSX`. Wyłączamy wymóg zmiany hasła przy logowaniu, wybieramy, by hasło nigdy nie wygasało.

6. **Zasób udostępniony i uprawnienia:**
   - W Eksploratorze plików tworzymy `C:\Profile`.
   - Prawym klik -> Właściwości -> Udostępnianie -> Zaawansowane udostępnianie.
   - Udostępnij ten folder. Nazwa udziału: `profile$`.
   - Przycisk Uprawnienia: Usuwamy istniejące wpisy, dodajemy `Wszyscy`, zaznaczamy "Pełna kontrola".
   - Karta Zabezpieczenia (Uprawnienia NTFS): Klikamy Zaawansowane -> Wyłącz dziedziczenie (Konwertuj dziedziczone uprawnienia). Usuwamy wszystkie pozycje oprócz `Administratorzy` i `Użytkownicy`. Dla obu upewniamy się, że mają "Pełna kontrola".

7. **Profil mobilny:**
   - W `dsa.msc` prawym na konto Adam Abacki -> Właściwości -> zakładka Profil.
   - W polu "Ścieżka profilu" wpisujemy: `\\SERWER-XX\profile$\%USERNAME%` (lub `abackia`).

**Polecenia do weryfikacji ustawień (PowerShell):**

```powershell
# Weryfikacja NIC Teaming
Get-NetLbfoTeam
Get-NetLbfoTeamMember

# Weryfikacja IP
Get-NetIPAddress -InterfaceAlias LAN12
Get-NetIPConfiguration

# Weryfikacja Hostname
hostname

# Weryfikacja AD (las i użytkownik)
Get-ADDomain
Get-ADUser -Identity abackia

# Weryfikacja udostępnionego folderu i uprawnień Share
Get-SmbShare -Name profile$
Get-SmbShareAccess -Name profile$

# Weryfikacja uprawnień NTFS
Get-Acl -Path C:\Profile | Format-List
```

---

### D2 - Linux Ubuntu Server

**Rozwiązanie:**

1. **NIC Teaming (Bonding):**
   - Interfejsy zmieniamy poleceniem `ip link set dev <stara_nazwa> name LAN1` lub przy użyciu `netplan`.
   - Konfigurujemy z wykorzystaniem pliku `/etc/netplan/01-netcfg.yaml`:

```yaml
network:
  version: 2
  ethernets:
    LAN1:
      match:
        macaddress: <mac_lan1>
    LAN2:
      match:
        macaddress: <mac_lan2>
  bonds:
    LAN12:
      interfaces: [LAN1, LAN2]
      addresses: [172.16.0.4/24]
      gateway4: 172.16.0.1
      nameservers:
        addresses: [127.0.0.1]
      parameters:
        mode: balance-alb # adaptive load balancing, niezależne od switcha
```

- Aplikujemy: `sudo netplan apply`

1. **Nazwa komputera:**
   - `sudo hostnamectl set-hostname SERWER-XX`
   - Aktualizacja `/etc/hosts`: dopisać `172.16.0.4 SERWER-XX.informatyk.local SERWER-XX`

2. **Instalacja i konfiguracja kontrolera domeny (Samba AD DC):**
   - Instalacja pakietów: `sudo apt install samba smbclient krb5-user winbind libpam-winbind libnss-winbind`
   - Promocja na DC (zatrzymanie usług):
     `sudo systemctl stop smbd nmbd winbind`
     `sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak`
   - Provisioning domeny:
     `sudo samba-tool domain provision --use-rfc2307 --realm=INFORMATYK.LOCAL --domain=INFORMATYK --adminpass=ZAQ!2wsx`
   - Kopiujemy konfigurację Kerberosa: `sudo cp /var/lib/samba/private/krb5.conf /etc/`
   - Startujemy Sambę: `sudo systemctl unmask samba-ad-dc`, `sudo systemctl enable samba-ad-dc`, `sudo systemctl start samba-ad-dc`

3. **Jednostka organizacyjna i użytkownik:**
   - Tworzenie OU: `sudo samba-tool ou create "OU=Serwisanci,DC=informatyk,DC=local"`
   - Tworzenie użytkownika w OU z podaniem hasła:
     `sudo samba-tool user create abackia 12!@EWSX --given-name=Adam --surname=Abacki --userou="OU=Serwisanci"`

4. **Folder udostępniony (profile) i profil mobilny:**
   - Tworzenie folderu: `sudo mkdir -m 777 /Profile`
   - Zabezpieczenia NTFS (Linux ACL): `sudo chmod 777 /Profile`
   - Konfiguracja profilu w AD:
     `sudo samba-tool user setprofilepath abackia "\\SERWER-XX\profile$\%USERNAME%"`
   - Dodanie udziału w `/etc/samba/smb.conf`:

```ini
[profile$]
    path = /Profile
    read only = no
    guest ok = no
    create mask = 0777
    directory mask = 0777
    browseable = no
```

- Przeładowanie samby: `sudo smbcontrol all reload-config`

**Polecenia do weryfikacji ustawień (Bash):**

```bash
# Weryfikacja bondingu i IP
ip addr show LAN12
cat /proc/net/bonding/LAN12

# Weryfikacja hostname
hostnamectl status

# Weryfikacja AD (Samba)
sudo samba-tool user list
sudo samba-tool user show abackia
sudo samba-tool ou list "DC=informatyk,DC=local"

# Weryfikacja zasobu Samba
testparm -s
ls -ld /Profile
```

## 7. Konfiguracja stacji roboczej

### C1 - Windows 10/11

**Rozwiązanie:**

1. **Adresacja IP dla LAN3:**
   - Przechodzimy do `ncpa.cpl`. Zmieniamy nazwę na `LAN3`.
   - We właściwościach IPv4 ustawiamy:
     - IP: `172.16.0.3`
     - Maska: `255.255.255.0`
     - Brama: `172.16.0.1`
     - DNS: `172.16.0.4` (IP serwera)

2. **Zmiana nazwy komputera i podłączenie do domeny:**
   - Można to zrobić z tego samego okna: Wciskamy `Win + Pause`, "Zmień nazwę tego komputera (zaawansowane)" lub "Zaawansowane ustawienia systemu" -> "Nazwa komputera" -> Zmień.
   - Nazwa: `SERWIS`.
   - Zaznaczamy "Domena" i wpisujemy `informatyk.local`. Zostaniemy poproszeni o poświadczenia administratora domeny (Administrator / ZAQ!2wsx). Po restarcie komputer będzie w domenie.

3. **Pliki i szyfrowanie EFS:**
   - Na dysku `C:\` tworzymy folder `PLIKI`.
   - Kopiujemy `tapeta.jpg` z nośnika do `C:\PLIKI`.
   - Prawym przyciskiem myszy na plik `tapeta.jpg` -> Właściwości -> Zaawansowane -> zaznaczamy "Szyfruj zawartość, aby zabezpieczyć dane" (EFS). Klikamy OK i Zastosuj. Na pytanie czy szyfrować plik i folder nadrzędny, wybieramy "Tylko plik".

4. **Logowanie domeny:**
   - Na ekranie logowania wybieramy "Inny użytkownik".
   - Wpisujemy `INFORMATYK\abackia` oraz hasło `12!@EWSX`.
   - Po pomyślnym załadowaniu pulpitu (utworzy się profil w systemie i na serwerze), wylogowujemy się.

**Polecenia do weryfikacji ustawień (CMD/PowerShell):**

```powershell
# Weryfikacja IP
ipconfig /all

# Weryfikacja domeny i nazwy
systeminfo | findstr /B /C:"Domain" /C:"Host Name"

# Weryfikacja szyfrowania EFS
cipher /c C:\PLIKI\tapeta.jpg
```

---

### C2 - Linux Ubuntu Stacja

**Rozwiązanie:**

1. **Adresacja IP dla LAN3:**
   - Korzystamy z `nmcli` lub z graficznego apletu sieciowego.
   - Komendą:
     `nmcli connection modify <nazwa_połączenia> connection.id LAN3`
     `nmcli connection modify LAN3 ipv4.addresses 172.16.0.3/24 ipv4.gateway 172.16.0.1 ipv4.dns 172.16.0.4 ipv4.method manual`
     `nmcli connection up LAN3`

2. **Zmiana nazwy komputera:**
   - `sudo hostnamectl set-hostname SERWIS`

3. **Podłączenie do domeny Active Directory:**
   - Używamy narzędzia `realmd` oraz `sssd`.
   - Instalacja: `sudo apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin`
   - Włączenie serwera DNS na interfejsie pozwoli znaleźć domenę.
   - Podłączanie: `sudo realm join -U Administrator informatyk.local`
   - Sprawdzamy status: `realm list`
   - Aktywujemy tworzenie katalogu domowego dla konta domenowego: w pliku `/etc/pam.d/common-session` upewniamy się, że jest `session optional pam_mkhomedir.so skel=/etc/skel umask=077`.

4. **Pliki i szyfrowanie EFS (GnuPG lub fscrypt w systemie Linux):**
   - Systemy Linux nie obsługują natywnie NTFS EFS. Ekwiwalentem dla pojedynczego pliku w zadaniach tego typu jest wykorzystanie np. GnuPG.
   - Tworzenie katalogu: `sudo mkdir /PLIKI`
   - Kopiowanie pliku (przy założeniu że nośnik montowany jest w `/media/`): `sudo cp /media/DOKUMENTACJA/PROGRAMY/PLIKI/tapeta.jpg /PLIKI/`
   - Szyfrowanie pliku symetryczne:
     `gpg -c /PLIKI/tapeta.jpg` (zostaniemy poproszeni o hasło, wygeneruje się `tapeta.jpg.gpg`). Oryginalny plik możemy usunąć: `sudo rm /PLIKI/tapeta.jpg`.

5. **Logowanie domeny:**
   - Użytkownika domenowego testujemy z terminala: `su - abackia@informatyk.local`
   - Sprawdzamy logowanie, polecenie np. `id`, po czym wylogowujemy się poleceniem `exit`.

**Polecenia do weryfikacji ustawień (Bash):**

```bash
# Weryfikacja IP
ip a show <interfejs>
nmcli dev show

# Weryfikacja nazwy komputera
hostname

# Weryfikacja domeny
realm list
id abackia@informatyk.local

# Weryfikacja plików i szyfrowania
ls -l /PLIKI/
```

## 8. Test komunikacji

**Rozwiązanie:**
Aby serwer mógł swobodnie komunikować się (odpowiadać na pakiety ping) ze stacją roboczą z systemem Windows, należy umożliwić na zaporze sieciowej żądania wejścia echa ICMPv4 na stacji roboczej.

- Na stacji C1 (Windows): Otwieramy `wf.msc` (Zapora Windows Defender z zabezpieczeniami zaawansowanymi). W Regułach ruchu przychodzącego włączamy zdefiniowane reguły `Udostępnianie plików i drukarek (Żądanie echa - ruch przychodzący ICMPv4)`.
- Na stacji C2 (Linux): Otwieramy zaporę ufw: `sudo ufw allow icmp` (domyślnie ufw pozwala na ping).

Wykonywanie testu:
Z poziomu serwera otwieramy wiersz polecenia i wpisujemy:

```cmd
ping 172.16.0.2
ping 172.16.0.1
ping 172.16.0.3
```

Jeżeli pojawią się opóźnienia `Czas=<XX>ms`, komunikacja jest weryfikowana i działa poprawnie. Należy to zgłosić Przewodniczącemu ZN.

## 9. Harmonogram prac

**Rozwiązanie:**
W programie arkusza kalkulacyjnego (np. MS Excel lub LibreOffice Calc):

1. Otwieramy nowy skoroszyt.
2. Przerysowujemy strukturę Tabeli 2 z arkusza egzaminacyjnego.
3. W kolumnie `Czas wykonania [min]` dla poszczególnych komórek (np. B2 do B7) wpisujemy minuty (np. 15, 10, 10, 30, 45, 30).
4. W komórce podsumowującej `RAZEM:` (np. B8) wpisujemy formułę funkcji sumy, czyli `=SUMA(B2:B7)`.
5. Ustawienie formatowania warunkowego na komórce z sumą (B8):
   - Zaznaczamy komórkę B8.
   - W Excelu: "Formatowanie warunkowe" -> "Nowa reguła" -> "Formatuj tylko komórki zawierające".
   - Warunek 1: "Wartość komórki", "mniejsza lub równa", `150`. Klikamy "Formatuj..." i na karcie "Wypełnienie" wybieramy kolor **niebieski**.
   - Dodajemy drugą regułę do komórki B8: "Formatowanie warunkowe" -> "Nowa reguła".
   - Warunek 2: "Wartość komórki", "większa niż", `150`. Klikamy "Formatuj..." i wybieramy kolor **zielony**.
6. Zapisujemy plik na pendrive opisanym `Egzamin-x` pod nazwą `harmonogram.xlsx` (lub `.ods`).
