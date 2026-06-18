# Rozwiązanie egzaminu INF.02 - Sesja Czerwiec 2025, wersja arkusza 08 SG

## 1. Wykonaj montaż okablowania sieciowego

**Rozwiązanie:**
Zgodnie z wytycznymi w pliku `README.md` (Zasada A), zadanie z montażem okablowania zostaje fizycznie pominięte.
Dla celów teoretycznych: wykonanie patchcordu U/UTP w standardzie T568A wymaga po obu stronach wtyków RJ-45 z kolejnością ułożenia żył: Biało-Zielony, Zielony, Biało-Pomarańczowy, Niebieski, Biało-Niebieski, Pomarańczowy, Biało-Brązowy, Brązowy. Weryfikację kabla potwierdza użycie testera, o co należy zgłosić się do Przewodniczącego Zespołu Nadzorującego.

## 2. Identyfikacja parametrów bezprzewodowej karty sieciowej

Odczyt należy zapisać w dostarczonej Tabeli 2 oraz sporządzić zrzuty w odpowiednim katalogu na dysku USB opisanym jako Egzamin-x. Poniżej przedstawiono rozwiązania dla platform określonych w instrukcji.

### C1 - Windows 10/11

Odczytu dokonujemy za pomocą terminala (CMD lub PowerShell).

**Polecenia CMD (Weryfikacja):**

```cmd
netsh wlan show interfaces
netsh wlan show drivers
```

- **Adres fizyczny:** Wynika z pola `Physical address` w sekcji informacji o interfejsie.
- **Producent:** Wyodrębniony z `Description` karty lub wyświetlany po wylistowaniu sterowników.
- **Obsługiwane standardy IEEE 802.11:** Możliwe do odczytu pod komendą `show drivers` w polu Typy Obsługi Radia (np. 802.11b, 802.11g, 802.11n, 802.11ac, 802.11ax).

Otrzymane w konsoli okna "wycinamy" korzystając z *Narzędzia Wycinanie* i zapisujemy na pulpicie/pendrive jako zrzut.

### C2 - Linux Ubuntu Stacja

Rozwiązanie z terminala bash.

**Polecenia do odczytu ustawień (Weryfikacja):**

```bash
# Adres Fizyczny i Producent sprzętu:
lshw -C network | grep -i wireless -A 10
# Ewentualnie adres MAC bezpośrednio:
ip link show wlan0

# Standardy łączności Wi-Fi z użyciem polecenia iw:
iw list | grep -i "Band" -A 10
# lub prościej odpytując wprost kartę z listami (802.11):
iwconfig
```

Dokonujemy zrzutu terminala kombinacją klawiszy PrtScn lub dedykowaną aplikacją i wypełniamy arkusz.

## 3. Konfiguracja rutera (MikroTik)

**Rozwiązanie:**
(Oparto na założeniu, że ruter i AP są w jednym urządzeniu opartym o RouterOS). Zakładamy stanowisko XX = 08.

**Konfiguracja CLI:**

```routeros
# Zmiana hasła zgodnie z polityką domyślną Mikrotika w nowych wersjach
/user set admin password=NoweHaslo!

# Skonfigurowanie adresu LAN rutera
/ip address add address=192.168.0.1/24 interface=bridge_lan

# Usunięcie (Wyłączenie) serwera DHCP
/ip dhcp-server remove [find]

# Konfiguracja sieci Bezprzewodowej
# Utworzenie profilu zabezpieczeń dla sieci z WPA2-PSK AES/CCMP:
/interface wireless security-profiles add name=sec_egzamin authentication-types=wpa2-psk wpa2-pre-shared-key=egzamin_08 mode=dynamic-keys

# Ustawienie karty wlan na 2.4GHz, wpisanie nazwy SSID, przypięcie kanału z nr stanowiska (Zakładamy np. Kanał 8 -> frequency=2447 MHz), z zabezpieczeniem profilu.
/interface wireless set wlan1 disabled=no mode=ap-bridge ssid=EGZAMIN_08 band=2ghz-b/g/n frequency=2447 security-profile=sec_egzamin
```

**Odczyt konfiguracji rutera (Weryfikacja):**

```routeros
/ip address print
/ip dhcp-server print
/interface wireless print
/interface wireless security-profiles print
```

## 4. Konfiguracja przełącznika (MikroTik)

**Rozwiązanie:**
Konfiguracja sprzętowa VLAN poprzez filtrowanie wewnątrz mostka L2 na MikroTiku.

**Konfiguracja CLI:**

```routeros
# Stworzenie wirtualnego bridge
/interface bridge add name=bridge1 vlan-filtering=no

# Przekazanie portów 1, 2 i 3 do bridge'a ze znakiem tagowania PVID w trybie dostępu dla żądanego ID 123
/interface bridge port add bridge=bridge1 interface=ether1 pvid=123
/interface bridge port add bridge=bridge1 interface=ether2 pvid=123
/interface bridge port add bridge=bridge1 interface=ether3 pvid=123

# Zabezpieczenie portów dla trybu untagged na vlan
/interface bridge vlan add bridge=bridge1 vlan-ids=123 untagged=ether1,ether2,ether3

# Załączenie filtrowania VLANów
/interface bridge set bridge1 vlan-filtering=yes

# Skonfigurowanie wirtualnego interfejsu VLAN 123 oraz nadanie na nim adresu IP
/interface vlan add name=vlan123 vlan-id=123 interface=bridge1
/ip address add address=192.168.0.5/24 interface=vlan123

# Brama domyślna dla przełącznika (skierowana na adres IP rutera)
/ip route add dst-address=0.0.0.0/0 gateway=192.168.0.1
```

**Odczyt konfiguracji przełącznika (Weryfikacja):**

```routeros
/interface bridge vlan print
/interface bridge port print
/ip address print
/ip route print
```

## 5. Połączenie urządzeń sieciowych

*(Pominięto, zgodnie z zasadą A. Przebieg zakłada okablowanie bezpośrednie urządzeń zgodnie z dostarczonym przez CKE schematem fizycznym).*

## 6. Konfiguracja serwera

(W rozwiązaniach używany jest symbol stanowiska XX=08).

### D1 - Windows Server 2022

**Rozwiązanie:**

1. **Płyty sieciowe:**
   - Poprzez wbudowany Menedżer `ncpa.cpl`.
   - Zmieniamy nazwę na `LAN_W` interfejsowi łączącemu serwer i Switch.
   - Właściwości -> Adres IPv4. Adres przypisany to `192.168.0.18` (10 + 08 z XX). Maska podsieci: `255.255.255.0`. Brama domyślna to `192.168.0.1`. Serwer DNS wskazany na Siebie: `127.0.0.1` (localhost).
   - Drugą kartę sieciową wyłączamy w jej Właściwościach z Menedżera, po kliknięciu Prawym klawiszem (Opcja `Wyłącz`).

2. **Drukarka Sieciowa RAW z udostępnianiem:**
   - Uruchamiamy opcje Panelu Sterowania i wyszukujemy "Urządzenia i drukarki" (lub z Narzędzi -> Zarządzanie drukowaniem).
   - "Dodaj drukarkę" -> Wprowadź jej IP `192.168.0.200` i wybierz opcję autodetekcji TCP/IP ustawiając Typ na Standardowy "Raw".
   - Podczas żądania sterowników używamy opcji "Z Dysku..." po czym podpinamy lokalizację `DRUKARKA` z przenośnego nosnika.
   - We Właściwościach zainstalowanego urządzenia w zakładce "Udostępnianie" zaptaszkowujemy opcję "Udostępnij tę drukarkę" i wypełniamy jej identyfikator: `drukarka_08`.

3. **Active Directory:**
   - W Menedżerze Serwera, z apletu "Zarządzanie" -> "Dodaj role i funkcje".
   - Klikamy na listę wyboru zaznaczając rolex usług domenowych Active Directory. Po restarcie/pobraniu zatwierdzamy by "Promować serwer do roli kontrolera domeny".
   - Nowy las z domeną: `zs08.local`.
   - Zgodnie z punktacją hasło odzyskiwania DSRM określamy na `egz@INF2`. W przypadku żądania z okienka przy wymuszonym resecie ustawiamy hasło administratora AD na docelowe: `zaq1@WSX`.

4. **Katalogi Profilowe z Zabezpieczeniem:**
   - Eksplorator dysku systemowego -> nowy folder `C:\ZSprofile`.
   - Poprzez prawy przycisk, opcję w Właściwości -> Udostępnianie włączamy jego widoczność jako `ZSprofile$`.
   - Pod opcjami okienka na Uprawnienia, wybieramy grupę `Wszyscy`, i zatwierdzamy dla niej `Pełna kontrola`.
   - Dla uprawnień samego folderu, przechodząc do "Zabezpieczeń", nadajemy analogicznie wszystkim grupom "użytkownicy" pełen stopień dostępu.

5. **Organizacja i Konta Użytkowników (dsa.msc):**
   - Po restarcie do roli, w programie `dsa.msc`, z prawej strony root folderu klikamy z listy w Jednostkę Organizacyjną (OU) by dodać `TECHNIKUM`. W `TECHNIKUM` kreujemy Nową Grupę Globalną - `UCZNIOWIE`.
   - Tworzymy kolejnego Użytkownika wewnątrz `TECHNIKUM`. Pełna nazwa (imię i nazwisko): `Jan Nowak`. Skrócony login (Nazwa logowania / SAMAccountName): `jnowak`.
   - Do logowania posłuży nam `jnow#1UCZ#`. Parametrem obłożonym we wlasciwościach jest dodatkowa funkcja zaznaczenia zakazu wygasania.
   - Przechodzimy we właściwości użytkownika `jnowak`. W zakładce "Profil" przy wskaźnikach katalogów, zmieniamy pole `Ścieżka profilu` adresując ją bezpośrednio pod: `\\Nasz_Hostname_lub_AD\ZSprofile$\%USERNAME%`. Na końcu dopisujemy go do nowo stworzonej grupy `UCZNIOWIE`.

6. **Zasady Grupy (GPO) dla Domenty:**
   - Skrót systemowy z Servera -> `gpmc.msc`. Znajdujemy swoją nowo wykreowaną dziedzinę domeny.
   - Pod korzeniem "Domeny" -> `zs08.local` naciskamy prawy i kreujemy "Utwórz Obiekt GPO". Nadajemy mu nazwę `bezpieczna_szkola`.
   - Klikamy weń by edytować. "Zasady -> Ustawienia Systemu Windows -> Ustawienia Zabezpieczeń -> Zasady Konta".
   - `Zasady haseł` - w opcji `Minimalna długość hasła` wpisujemy parametr: `8`.
   - `Zasady blokady konta` - opcja dot. Półokresów: Zmieniamy na `3 nieudane próby logowania`. Potwierdzając podrzędny wyrok, zmieniamy parametr zablokowania `Czas trwania blokady konta` na domyślny czas: `15` minut.
   - Wgrywamy od góry `gpupdate /force` z linii CMD.

**Weryfikacja parametrów (PowerShell):**

```powershell
ipconfig /all
Get-Printer -Name "Nazwa_Drukarki" | Format-List
Get-ADUser -Identity jnowak
Get-ADGroupMember -Identity UCZNIOWIE
Get-SmbShare -Name ZSprofile$
Get-Acl -Path C:\ZSprofile | Format-List
```

---

### D2 - Linux Ubuntu Server

**Rozwiązanie:**
(Linuxowe mapowanie parametrów wprost proporcjonalne do zaleceń M.S Server Windows).

1. **Konfiguracja Netplan i wyłączenie interfejsu drugiego:**
   - W otwartym z roota dokumencie `/etc/netplan/01-netcfg.yaml`:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      set-name: LAN_W
      addresses: [192.168.0.18/24] # Założono dla X=8
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses: [127.0.0.1]
    ens34:
      activation-mode: off
```

- Zaaplikowanie planu: `sudo netplan apply`

1. **Przygotowanie środowiska druku w CUPS:**
   - Instalacja `sudo apt update && sudo apt install cups`.
   - Serwer dołączamy w konfigu (lub nakładką `cupsctl`): w `/etc/cups/cupsd.conf` zmieniamy serwowane porty (Port 631) oraz uprawnienia odczytu, logując `Allow from 192.168.0.*`.
   - Wpięcie maszyny raw przez polecenie systemowe i zaznaczenie współdzielenia z SMB (Samba) lub natywne `lpadmin`:
     `sudo lpadmin -p drukarka_08 -E -v socket://192.168.0.200:9100 -m raw`
   - Opcja wymuszenia dzielenia z AD zostanie omówiona w wpisie poniżej.

2. **Active Directory za pomocą Samba Tool:**
   - Podstawowa weryfikacja bibliotek to instalacja: `samba smbclient krb5-user winbind`.
   - Ukrycie kopii standardowego smb pod starą nazwę i utworzenie nowego domyślnego podsieci: `sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak`.
   - Procedura prowizjonowania domeny bez serwera proxy (Hasło Administratora w Samba/Kerberos):
     `sudo samba-tool domain provision --use-rfc2307 --realm=ZS08.LOCAL --domain=ZS08 --adminpass=zaq1@WSX`
   - Kopiujemy bibliotekę serwerową: `sudo cp /var/lib/samba/private/krb5.conf /etc/`. Uruchamiamy AD DC: `sudo systemctl restart samba-ad-dc`.

3. **Jednostka OU i polityki grupy Użytkowników:**
   - Instalacja folderów: `sudo samba-tool ou create "OU=TECHNIKUM,DC=zs08,DC=local"`
   - Utworzenie Grupy `UCZNIOWIE`: `sudo samba-tool group add UCZNIOWIE --groupou="OU=TECHNIKUM"`
   - Parametry pracownicze dla Jana Nowaka (wraz z hasłem):
     `sudo samba-tool user create jnowak jnow#1UCZ# --given-name=Jan --surname=Nowak --userou="OU=TECHNIKUM"`
   - Połączenie Jana do grupy uczniowskiej: `sudo samba-tool group addmembers UCZNIOWIE jnowak`

4. **Przestrzeń folderu profilów:**
   - Tworzymy pustą połać w korzeniu: `sudo mkdir -m 777 -p /ZSprofile`
   - Edycja pliku konfiguracyjnego `smb.conf` o dopisy domowe (udziały dla `ZSprofile$` z pełnym RW) jak i drukarek CUPSowych.

```ini
[ZSprofile$]
    path = /ZSprofile
    read only = no

[printers]
   comment = Drukarka_08_Raw_Share
   browseable = yes
   path = /var/spool/samba
   printable = yes
   guest ok = yes
   read only = yes
```

- Zaaplikowanie drogi z konta użytkownika do profilu po udziale `zs08`:
     `sudo samba-tool user setprofilepath jnowak "\\\\PCserwer\\ZSprofile$\\%USERNAME%"` (lub `\ZSprofile$` bez backslashy dla zmiennych w GUI Menedzera windows). Resety usługi smb.

1. **Definicja reguł (GPO z Samba Tool):**
   - Ustalamy parametry hasłowe domeny przy użyciu odpowiednich narzędzi poleceń systemu Linux z Samba.
   - Minimalna długość 8: `sudo samba-tool domain passwordsettings set --min-pwd-length=8`
   - Zasady resetu przy 3 wtopach: `sudo samba-tool domain passwordsettings set --account-lockout-threshold=3`
   - Czas na kwadrans: `sudo samba-tool domain passwordsettings set --account-lockout-duration=15`

**Weryfikacja parametrów (Bash):**

```bash
ip a show LAN_W
sudo samba-tool user show jnowak
sudo samba-tool domain passwordsettings show
lpstat -p
testparm -s
```

## 7. Konfiguracja stacji roboczej

### C1 - Windows 10/11

**Rozwiązanie:**

1. **Adresacja interfejsu WLAN (`LAN_K`):**
   - Po wyłączeniu kabli przechodzimy do modułu `ncpa.cpl`. Zmieniamy adapter z modułem radiowym na `LAN_K`.
   - Sprawdzamy, czy w protokole Właściwości IPv4 zaznaczona jest na sztywno odpowiednia opcja "Uzyskaj adres IP i DNS - statycznie" - my konfigurujemy te ręczne wedle zadań (nie łączymy się po DHCP).
   - Wpisujemy IP: `192.168.0.101`. Maska to `255.255.255.0`.
   - Brama domyślna dla modułu wi-fi: `192.168.0.1`.
   - Serwer DNS stacji łączony do Serwera: `192.168.0.18`.
   - Odnajdujemy sieć `EGZAMIN_08` z panelu po prawej i łączymy uwierzytelniając ją podanym na routerze kluczem `egzamin_08`.

2. **Podłączenie stacji do kontrolera:**
   - Przed podłączeniem sprawdzamy ewentualne komunikaty pingu. `Win + Pause` (Panel Sysdm), wybieramy Domena i podajemy `zs08.local`. Pamiętamy by nadać sobie uwierzytelnienie od Administratorów z DC. Po powiadomieniu o pomyślnym przyjęciu odczekujemy do restartu stacji i przelogowujemy.

3. **Autoryzacja użytkownika, drukarka:**
   - Logujemy na `jnowak` hasłem `jnow#1UCZ#`. Stacja zacznie pobierać (jeśli poprawnie powiązana) profil Roamingowy (jeżeli opcje na serwerze nie blokują dysku).
   - Menu start i przechodzimy przez Pasek uruchom `\\192.168.0.18`. Klikamy prawym w zasób pokazany jak ikona `drukarka_08` -> opcja `Połącz`. System zainstaluje sterownik ze stacji serwerowej. Następnie otwieramy Panel z urządzeniami - prawym kliknięciem wybieramy "Właściwości Drukarki" i drukujemy próbną treść strony by ukazać ją dla Komisji.

**Weryfikacja parametrów (CMD/PowerShell):**

```powershell
ipconfig /all
systeminfo | findstr /B /C:"Domain"
Get-Printer
```

---

### C2 - Linux Ubuntu Stacja

**Rozwiązanie:**

1. **Zarządzanie połączeniem WiFi w terminalu `LAN_K`:**
   - Zmiana interfejsów Network Managera. Wbudowane pakiety w konsoli radzą sobie w miarę szybko. Zmiana nazwy profilu:
   - `nmcli connection add type wifi con-name LAN_K ifname wlan0 ssid EGZAMIN_08`
   - `nmcli connection modify LAN_K wifi-sec.key-mgmt wpa-psk wifi-sec.psk egzamin_08`
   - Oznaczamy stały IP: `nmcli connection modify LAN_K ipv4.addresses 192.168.0.101/24 ipv4.gateway 192.168.0.1 ipv4.dns 192.168.0.18 ipv4.method manual`
   - Uruchomienie połączenia wprowadzonymi parametrami: `nmcli connection up LAN_K`.

2. **Zapis do domeny z poziomu pakietu Reamld:**
   - Doinstalowanie pakietów integracji: `sudo apt install realmd sssd sssd-tools adcli samba-common-bin`.
   - `sudo realm join -U Administrator zs08.local`. Autoryzacja dla poświadczeń logowania.

3. **Drukowanie próbne dla podłączonego dysku SMB po zmapowaniu i zalogowaniu z konta użytkownika jnowak:**
   - Otwarcie graficznego zarządzania drukarkami systemu Ubuntu (lub przeglądarka pod polem z adresem `localhost:631` CUPS-a).
   - Wpisując pole i poświadczenie podajemy autoryzację do udostępnionej sieci drukującej: lokalizacja sieci to m.in. `smb://192.168.0.18/drukarka_08`.
   - Do zgłoszenia i weryfikacji zadania drukujemy odpowiedni raport wydruków na kartkę.

**Weryfikacja parametrów (Bash):**

```bash
nmcli connection show LAN_K
realm list
lpstat -p
```

## 8. Test komunikacji

**Rozwiązanie:**
Odpalamy opcję puszczenia paczek komunikacyjnych testując złącze ze Stacją. Serwer, z odpowiednim adresem IP musi otrzymać informację.

Jeżeli takowa łączność z Windows na stacji odrzuca ping, otwieramy aplikacje obrony -> Zaawansowane Zabezpieczenia zaporowe Windows Defender (`wf.msc`). Znajdujemy i upewniamy się, że włączona jest opcja w regułach wchodzących dla IPv4 na wymóg `Udostępnianie plików i drukarek (Żądanie echa - ruch przychodzący)`.

Na stacji z wiersza plecenia:

```cmd
ping 192.168.0.1     (Router LAN interfejsu pod brama)
ping 192.168.0.18    (Serwer domeny na złączach)
```

Serwer:

```cmd
ping 192.168.0.200   (Drukarka sieciowa RAW)
```

Zgłaszamy egzaminatorom uzyskanie wszystkich odpowiedzi o wymiarze `TTL=...`.

## 9. Harmonogram i kosztorys prac - Kalkulacja

Do pracy wymaga się utworzenia tablicowego zapisu arkusza kalkulacyjnego w celu rozrachunkowym zgodnym z Cennikiem z poleceń z Tablic 1 oraz Tablicy Wzoru 3. Arkusz odpalić należy w środowisku programu np. MS Excel, po czym zachować jego postać w odpowiednim rozszerzeniu do dysku USB.

1. Wpisujemy wszystkie Nazwy oraz "Ceny netto" przyporządkowane manualnie do każdej konkretnej pozycji na dół, zgodnie z poleceniami. Do podanej `Ilość` wpisujemy "1".
2. Automatyzacja wyliczeń `VAT w zł` musi opierać się o stawkę stałą w naszym przypadku do wklejenia polecenia dla 2 rzędu: np. `=C2*0,23` (23% podatku). Wyniki przekopiować niżej przy pomocy przeciągania strzałką rożną.
3. Formuła na rozwiązanie wartości z rzędu na podstawie sum dla komórki w kolumnie `Cena brutto w zł`: z równania np. `=C2+D2` by zsumować.
4. Odpowiadając na żądanie podliczenia `Wartość brutto w zł` wymnażamy wyżej obliczoną Cenę x wiersz z Ilościami wg.: `=E2*F2`.
5. Sumaryzacja całego słupka roboczego przez opisy z użyciem dedykowanych funkcji matematycznych (przy wierszach u dołu ekranu). Komórka po prawej obok na dole: `ŁĄCZNA WARTOŚĆ USŁUG`: Formuła agregacyjna m.in. `=SUMA(G2:G9)` zakładająca pobór z wszystkich kolumn z wartością brutto u góry.
6. Określenie komórki rabatowej nałożeniem pola do `UDZIELONY RABAT` uwarunkowane logicznie: Ustawiamy komórkę (załóżmy że kolumna ŁĄCZNA znajduje się np w B10) `=JEŻELI(B10>450; B10*0,1; 0)`. Wynikiem będzie obniżenie o 10% zapłaty całkowitej z kwoty ogólnej, lub wypisanie zera przy braku jego posiadania w puli poniżej minimalnego zakresu 450zł.
7. Komórka `DO ZAPŁATY`: `=B10-B11` jako ujęcie od siebie obydwu.
8. Walutę dopasowujemy przez opcje formatowania prawego przysku "Formatuj komórki" dla kolumn, jako Wartość w systemie Polskim "Walutowe" o znaczniku ZŁ (lub PLN według oprogramowania instalatora systemu na ekranie).
