# Rozwiązanie egzaminu INF.02 - Sesja Czerwiec 2025, wersja arkusza 04 SG

## 1. Wykonaj montaż okablowania sieciowego

**Rozwiązanie:**
Zgodnie z wytycznymi A w pliku `README.md` to zadanie jest celowo pomijane. Warto jednak wiedzieć, że wykonanie kabla krosowego prostego U/UTP w sekwencji T568B wymaga użycia wtyków 8P8C (RJ-45) zarobionych po obu stronach zgodnie z ułożeniem: Biało-Pomarańczowy, Pomarańczowy, Biało-Zielony, Niebieski, Biało-Niebieski, Zielony, Biało-Brązowy, Brązowy. Weryfikację przeprowadza się standardowym testerem okablowania.

## 2. Identyfikacja parametrów bezprzewodowej karty sieciowej

W arkuszu widnieje polecenie identyfikacji dla systemu Windows, z kolei polecenie C nakazuje opracowanie rozwiązań dla dwóch platform (C1 Windows, C2 Linux/Ubuntu). Zrzuty z wynikami zapisujemy w katalogu `Identyfikacja` podpinając napęd USB o nazwie `Egzamin-x`.

### C1 - Windows 10/11

Dane można pozyskać graficznie z Panelu Sterowania lub z linii poleceń.
**Polecenia do odczytu ustawień (Weryfikacja):**

```cmd
netsh wlan show interfaces
```

lub w PowerShell:

```powershell
Get-NetAdapter | Where-Object {$_.InterfaceDescription -match "Wi-Fi|Wireless"} | Select-Object Name, MacAddress, InterfaceDescription
netsh wlan show drivers
```

- **Adres fizyczny (MAC):** Podany jako adres fizyczny, np. `00:1A:2B:3C:4D:5E`.
- **Producent:** Wyciągnięty z nazwy i opisu interfejsu (np. `Intel`, `Realtek`, `Qualcomm`).
- **Obsługiwane standardy IEEE 802.11:** Wylistowane po komendzie `netsh wlan show drivers` (np. `802.11n, 802.11ac, 802.11ax`).

### C2 - Linux Ubuntu Stacja

Dane karty sieciowej (WLAN) w Linuksie identyfikuje się za pomocą narzędzi wiersza poleceń.
**Polecenia do odczytu ustawień (Weryfikacja):**

```bash
# Weryfikacja adresu MAC interfejsu wlan0
ip link show wlan0

# Weryfikacja producenta karty WiFi
lspci | grep Network
# LUB
lshw -C network | grep -i wireless -A 10

# Weryfikacja obsługiwanych standardów (b/g/n/ac/ax) za pomocą `iw`
iw list
```

Z wyników odczytujemy MAC, markę oraz parametry fizyczne, a zrzuty (np. zrzut z terminala przyciskiem `PrtScn`) zapisujemy na USB. Wyniki z tabeli 2 wpisujemy do odpowiednich komórek arkusza.

## 3. Konfiguracja rutera / punktu dostępowego (MikroTik)

**Rozwiązanie:**
Konfigurację przeprowadzamy z poziomu CLI RouterOS, z wykorzystaniem wbudowanego modułu WLAN (lub oddzielnego AP, jak zaznacza uwaga na egzaminie). Do rozwiązania przyjęto zintegrowany RouterOS. Zmienne `XX` oznaczają nr stanowiska (w poniższym przykładzie załóżmy numer `05`).

**Konfiguracja CLI:**

```routeros
# Zmiana hasła (jeśli wymagana)
/user set admin password=NoweHaslo!

# 1. Adres IP dla LAN
/ip address add address=192.168.0.1/24 interface=bridge_lan

# 2. Serwer DHCP wyłączony
/ip dhcp-server remove [find]

# 3. Konfiguracja interfejsu WLAN (zakładając wlan1) na pasmo 2.4 GHz i wskazany kanał (dla stanowiska 5 -> kanał 5 to 2432 MHz).
# Stworzenie profilu zabezpieczeń (Security Profile) z kluczem WPA2 AES/CCMP:
/interface wireless security-profiles add name=sec_egzamin authentication-types=wpa2-psk wpa2-pre-shared-key=egzamin_05 mode=dynamic-keys

# Ustawienie i włączenie (enable) modułu radiowego 2.4GHz:
/interface wireless set wlan1 disabled=no mode=ap-bridge ssid=EGZAMIN_05 band=2ghz-b/g/n frequency=2432 security-profile=sec_egzamin
```

**Odczyt konfiguracji rutera (Weryfikacja):**

```routeros
# Sprawdzenie adresacji IP:
/ip address print
# Sprawdzenie serwera DHCP:
/ip dhcp-server print
# Sprawdzenie profilu bezpieczeństwa:
/interface wireless security-profiles print
# Sprawdzenie parametrów bezprzewodowych:
/interface wireless print advanced
```

## 4. Konfiguracja przełącznika (MikroTik)

**Rozwiązanie:**
Rozwiązanie z użyciem mostka VLAN (Bridge VLAN Filtering).

**Konfiguracja CLI:**

```routeros
# 1. Tworzenie sieci VLAN o ID=13
/interface bridge add name=bridge1 vlan-filtering=no

# Porty 1, 2, 3 w VLAN 13 jako dostępowe bez tagowania (untagged)
/interface bridge port add bridge=bridge1 interface=ether1 pvid=13
/interface bridge port add bridge=bridge1 interface=ether2 pvid=13
/interface bridge port add bridge=bridge1 interface=ether3 pvid=13

# Ustawienie tagowania/brak tagowania w tablicy mostu
/interface bridge vlan add bridge=bridge1 vlan-ids=13 untagged=ether1,ether2,ether3

# Włączamy filtering VLAN
/interface bridge set bridge1 vlan-filtering=yes

# 2. Adres IP dla VLAN 13 (przełącznik L3): 192.168.0.5/24
/interface vlan add name=vlan13 vlan-id=13 interface=bridge1
/ip address add address=192.168.0.5/24 interface=vlan13

# 3. Brama domyślna: adres IP rutera
/ip route add dst-address=0.0.0.0/0 gateway=192.168.0.1
```

**Odczyt konfiguracji przełącznika (Weryfikacja):**

```routeros
/interface bridge port print
/interface bridge vlan print
/ip address print
/ip route print
```

## 5. Połączenie urządzeń sieciowych

*(Zadanie w warstwie fizycznej okablowania do ominięcia według poleceń `README.md`)*

## 6. Konfiguracja serwera

(W rozwiązaniach używany jest symbol stanowiska X. Dla uproszczenia kodu powołujemy się na X=5, `XX`=05).

### D1 - Windows Server 2022

**Rozwiązanie:**

1. **Nazwy połączeń i konfiguracja IP:**
   - W narzędziu Płyty sieciowej `ncpa.cpl` zmieniamy nazwę karty na `LAN_W`. Drugą kartę całkowicie "Wyłączamy".
   - Wchodzimy we Właściwości IPv4 dla `LAN_W`.
   - Adres IP (192.168.0.10 + 5): `192.168.0.15`
   - Maska: `255.255.255.0`
   - Brama domyślna: `192.168.0.1`
   - DNS: `127.0.0.1` (localhost).

2. **Drukarka sieciowa (Port RAW):**
   - Menedżer Serwera -> Narzędzia -> Zarządzanie drukowaniem lub z Panelu Sterowania (Urządzenia i drukarki -> Dodaj drukarkę).
   - "Dodaj drukarkę używając adresu TCP/IP". Typ urządzenia: TCP/IP. Hostname/IP: `192.168.0.200`. Odznaczamy opcję automatycznego pytania o sterownik. Wybieramy standard karty: Raw.
   - Gdy pojawi się prośba o sterownik, klikamy "Z dysku" i wskazujemy lokalizację `DOKUMENTACJA/PROGRAMY/DRUKARKA` na pendrive. Instalujemy.
   - We właściwościach tej drukarki przechodzimy na zakładkę "Udostępnianie". Zaznaczamy udostępnianie z nazwą zasobu: `drukarka_05`.

3. **Active Directory (Nowa domena w nowym lesie):**
   - Instalujemy Rola "Usługi Domenowe Active Directory" w Menedżerze Serwera.
   - Po instalacji -> "Promuj serwer na kontroler domeny" -> "Dodaj nowy las".
   - Nazwa domeny: `zs05.local`.
   - Hasło DSRM: `egz@INF2`. Instalujemy i zgadzamy się na restart serwera.
   - Jeśli wymagana była zmiana konta administratora dla trybu pracy katalogów, należy pamiętać o haśle `zaq1@WSX`. Zwykły administrator loguje się kontem domenowym po instalacji.

4. **Organizacja i użytkownicy (dsa.msc):**
   - Prawym na `zs05.local` -> Nowy -> Jednostka organizacyjna. Nazwa: `TECHNIKUM`.
   - W `TECHNIKUM` dodajemy Nową Grupę -> nazwa `UCZNIOWIE`, zakres "Globalny", typ zabezpieczeń.
   - W `TECHNIKUM` dodajemy Nowego Użytkownika. Imię: Karol, Nazwisko: Nowak, Logowanie: `knowak`.
   - Hasło: `know#1UCZ#`. Opcja "Użytkownik nie może zmienić hasła" i "Hasło nigdy nie wygasa". Następnie dodajemy go do grupy `UCZNIOWIE`.

5. **Folder udostępniony z profilem mobilnym:**
   - Zakładamy na dysku folder `C:\ZSprofile`.
   - Właściwości -> "Udostępnianie" -> "Zaawansowane". Udostępnij jako `ZSprofile$`.
   - Klikamy Uprawnienia i dajemy `Wszyscy` - Pełna Kontrola.
   - Przechodzimy do zakładki "Zabezpieczenia" -> Edytuj -> nadajemy `Wszyscy` lub `Użytkownicy` pełną kontrolę na poziomie systemu plików NTFS.
   - W Active Directory otwieramy właściwości profilu użytkownika `knowak`. W ścieżce profilu dajemy: `\\SERWER_HOSTNAME\ZSprofile$\%USERNAME%`. (Upewnij się co do poprawności HOSTNAME w ścieżce).

6. **Zasady Grupy (GPO):**
   - Z Menedżera otwieramy `gpmc.msc` (Zarządzanie Zasadami Grupy).
   - Rozwijamy "Las: zs05.local" -> "Domeny" -> `zs05.local`. Prawym myszy na ten element i klikamy "Utwórz obiekt zasad grupy w tej domenie...". Nazywamy go `bezpieczna_szkola`.
   - Prawym na nową regułę `bezpieczna_szkola` -> Edytuj.
   - Konfiguracja komputera -> Zasady -> Ustawienia systemu Windows -> Ustawienia zabezpieczeń -> Zasady konta:
     - W "Zasady haseł" modyfikujemy: `Minimalna długość hasła` na `8 znaków`.
     - W "Zasady blokady konta" modyfikujemy: `Próg blokady konta` na `3 nieudane próby logowania`. Następnie akceptujemy, by automatycznie zatwierdził `Czas trwania blokady konta` m.in. na `15 minut`.
   - Aktualizujemy zasady komendą `gpupdate /force` z Wiersza Poleceń.

**Weryfikacja parametrów (PowerShell):**

```powershell
Get-NetIPAddress -InterfaceAlias LAN_W
Get-ADUser -Identity knowak
Get-ADGroupMember -Identity UCZNIOWIE
Get-SmbShare -Name ZSprofile$
Get-Printer | Select Name, Shared, ShareName
```

---

### D2 - Linux Ubuntu Server

**Rozwiązanie:**

1. **Konfiguracja IP i dezaktywacja LAN (netplan):**
   - `sudo ip link set <stary_interfejs1> name LAN_W` (opcjonalne mapowanie sprzętowe przez systemd/netplan).
   - `/etc/netplan/01-netcfg.yaml`:

```yaml
network:
  version: 2
  ethernets:
    ens33: # Podłączone do LAN_W
      addresses: [192.168.0.15/24] # Założono X=5 -> 10+5
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses: [127.0.0.1]
    ens34: # Drugi wyłączony
      activation-mode: off
```

- `sudo netplan apply`

1. **Serwer Wydruku CUPS i udostępnianie surowe (RAW):**
   - Instalujemy: `sudo apt install cups`
   - Otwieramy plik `sudo nano /etc/cups/cupsd.conf`. Zmieniamy opcję z `Listen localhost:631` na `Port 631`, a także wprowadzamy modyfikacje dostępu, pozwalając adresacji `Allow from 192.168.0.*`. `Browsing On`.
   - Dodajemy raw drukarkę (kolejka bez sterownika graficznego, przekazuje zadania na port JetDirect serwera 192.168.0.200):
     `sudo lpadmin -p drukarka_05 -E -v socket://192.168.0.200:9100 -m raw`
   - Sprawdzamy ustawienia samby by współdzieliła wydruki z CUPS przez udziały CIFS. W `/etc/samba/smb.conf`:

```ini
[printers]
   comment = All Printers
   browseable = yes
   path = /var/spool/samba
   printable = yes
   guest ok = yes
   read only = yes
   create mask = 0700
```

- Reset Samby.

1. **Active Directory (Samba AD DC):**
   - Instalacja `samba` i powiązanych `krb5-user`, `winbind`.
   - Ukrycie starej konfiguracji `/etc/samba/smb.conf`.
   - Provisioning nowej domeny (`zs05.local`):
     `sudo samba-tool domain provision --use-rfc2307 --realm=ZS05.LOCAL --domain=ZS05 --adminpass=zaq1@WSX`
   - *(DSRM Pass z Windows serwera odnosi się do Kerberosa/katalogu)*. Opcjonalnie podmieniamy hasła Administratorowi. `sudo samba-tool user setpassword Administrator --newpassword=zaq1@WSX`.
   - Rozpoczęcie usługi AD-DC.

2. **Jednostka organizacyjna, grupa i użytkownik (Samba Tool):**
   - `sudo samba-tool ou create "OU=TECHNIKUM,DC=zs05,DC=local"`
   - `sudo samba-tool group add UCZNIOWIE --groupou="OU=TECHNIKUM"`
   - `sudo samba-tool user create knowak know#1UCZ# --given-name=Karol --surname=Nowak --userou="OU=TECHNIKUM"`
   - `sudo samba-tool group addmembers UCZNIOWIE knowak`

3. **Folder zasobów `ZSprofile$` i mobilny:**
   - `sudo mkdir -m 777 /ZSprofile`
   - Edycja udziału w `smb.conf`:

```ini
[ZSprofile$]
    path = /ZSprofile
    read only = no
```

- Przypisanie profilu mobilnego:
     `sudo samba-tool user setprofilepath knowak "\\\\SERWER_HOSTNAME\\ZSprofile$\\%USERNAME%"` (w zależności od formatowania shella).
- Reset AD: `sudo smbcontrol all reload-config`

1. **Zasada Grupy (Polityka Haseł w Samba AD DC):**
   - `sudo samba-tool domain passwordsettings set --min-pwd-length=8`
   - `sudo samba-tool domain passwordsettings set --account-lockout-threshold=3`
   - `sudo samba-tool domain passwordsettings set --account-lockout-duration=15`

**Weryfikacja parametrów (Bash):**

```bash
ip a show ens33
sudo samba-tool user show knowak
sudo samba-tool domain passwordsettings show
lpstat -p
```

## 7. Konfiguracja stacji roboczej (WLAN i Domena)

### C1 - Windows 10/11

**Rozwiązanie:**

1. **Adresacja IP dla interfejsu WLAN (`LAN_K`):**
   - W narzędziu `ncpa.cpl` modyfikujemy nazwę połączenia bezprzewodowego na `LAN_K`.
   - W jej właściwościach w zakładce IPv4 wprowadzamy:
     - IP: `192.168.0.101`
     - Maska: `255.255.255.0`
     - Brama: `192.168.0.1`
     - DNS (IP Serwera np.): `192.168.0.15`
   - Następnie po zdefiniowaniu interfejsu, znajdujemy sieć `EGZAMIN_05` w panelu prawym dolnym "Sieci Wi-Fi", łączymy się podając hasło: `egzamin_05`.

2. **Podłączenie do domeny `zsXX.local`:**
   - `Win + Pause` -> Zmień nazwę komputera / domeny. Zaznaczamy Domena i podajemy `zs05.local`.
   - Wyskoczy powiadomienie autoryzacyjne konta Administrator.
   - Komputer wykonuje restart.

3. **Drukarka Sieciowa z udziału po zalogowaniu na konto `knowak`:**
   - Logujemy się poświadczeniami ucznia w nowej domenie.
   - W wierszu start uruchamiamy powłokę (ikona lupy / uruchom): `\\192.168.0.15` (lub inna nazwa).
   - Otwieramy współdzieloną ikonę z drukarką `drukarka_05`. Klikamy prawym i wybieramy "Połącz". Sterowniki powinny zostać ściągnięte z serwera AD.
   - Wciskamy "Wydruk strony testowej" we właściwościach drukarki i sygnalizujemy powiadomienie przez podniesienie ręki dla egzaminatora.

**Weryfikacja parametrów (CMD/PowerShell):**

```powershell
ipconfig /all
systeminfo | findstr /B /C:"Domain"
# Po wylistowaniu drukarek:
Get-Printer
```

---

### C2 - Linux Ubuntu Stacja

**Rozwiązanie:**

1. **Zarządzanie połączeniem WiFi w terminalu i zmiana NAZWY KARTY NA `LAN_K`:**
   - Wykorzystanie wbudowanego w ubuntu GUI lub przez terminal (np. interfejs wlan0):
   - Użycie komendy `nmcli connection add type wifi con-name LAN_K ifname wlan0 ssid EGZAMIN_05`
   - Modyfikacja hasła: `nmcli connection modify LAN_K wifi-sec.key-mgmt wpa-psk wifi-sec.psk egzamin_05`
   - Modyfikacja IP: `nmcli connection modify LAN_K ipv4.addresses 192.168.0.101/24 ipv4.gateway 192.168.0.1 ipv4.dns 192.168.0.15 ipv4.method manual`
   - Uruchomienie połączenia: `nmcli connection up LAN_K`.

2. **Podłączenie do domeny:**
   - Narzędzie w systemie: `realm join -U Administrator zs05.local`. Po poprawnym dołączeniu i autoryzacji usługa z systemem PAM winna zalogować użytkownika.

3. **Zmapowanie drukarki w CUPS klienckim dla usera knowak:**
   - Otwarcie graficznego zarządzania drukarkami systemu Ubuntu (lub przeglądarka i CUPS `localhost:631`).
   - Podpięcie nowej instalacji drukarki SMB/CIFS za pomocą lokalizacji: `smb://192.168.0.15/drukarka_05`.
   - Zaznaczenie odpowiedniego wydruku na stronie testowej i wywołanie zgłoszenia u Egzaminatora.

**Weryfikacja parametrów (Bash):**

```bash
nmcli connection show LAN_K
realm list
lpstat -p
```

## 8. Test komunikacji

**Rozwiązanie:**
Test komunikacji zakłada łączność sieci bezprzewodowych oraz sieci miedzianych z drukarką z Serwera.
Sprawdzamy zaporę systemu `Serwer`: Pamiętaj, aby Włączone (zielone strzałki na regułach) było udostępnianie plików i drukarek, ze szczególnym uwzględnieniem żądania `ICMPv4`.

Z wiersza poleceń stacji (C1/C2 Windows/Linux) po zalogowaniu:

```cmd
ping 192.168.0.1     (Interfejs rutera z brama)
ping 192.168.0.15    (Nasz przykładowy serwer 0.10+XX)
```

Z wiersza poleceń serwera (D1/D2 Windows/Linux) testujemy docelową drukarkę:

```cmd
ping 192.168.0.200   (Drukarka RAW egzaminacyjna)
```

Jeżeli parametry w pingu informują o pakietach wysłanych i odebranych - zgłaszamy poprawność działania komunikacji odpowiedniemu Zespołowi Nadzorującemu.

## 9. Harmonogram i kosztorys

Zadanie opiera się na przygotowaniu poprawnego formatowania dla kolumn z załączonych wytycznych na egzaminie. Należy wykonać to w MS Excel lub Calc LibreOffice.

1. Przerysować ramki z pliku do programu i umieścić dane (wypełnić wpisanymi na twardo np. Ceny netto).
2. W kolumnie `VAT w zł` należy wpisać formułę auto-wyliczeniową: komórka na poziomie `[Cena netto w zł]` x `0,23` lub `23%` dla każdego z poszczególnych wierszy w tabeli (wyciągnąć krzyżyk z zaznaczenia na cały dół).
3. W kolumnie `Cena brutto w zł` zastosować formułę, która polega na zsumowaniu danych z rzędu z komórkami np. dla wiersza pierwszego: `=B2+C2`.
4. Wypełnić sztuczną Ilość (np. 1).
5. Wyliczenie dla komórki `Wartość brutto w zł` równe jest `[Cena brutto] x [Ilość]` z odpowiedniego wiersza np. `=D2*E2`.
6. `ŁĄCZNA WARTOŚĆ USŁUG` po prawej stronie na dole: Użycie `=SUMA(F2:F9)`, gdzie wiersze dotyczą wszystkich obliczonych Wartości Brutto z rzędów górnych.
7. Komórka dla `UDZIELONY RABAT`: Funkcja warunkowa logiczna `=JEŻELI([adres ŁĄCZNA WARTOŚĆ USŁUG]>450; [adres ŁĄCZNA WARTOŚĆ USŁUG]*0,1; 0)`.
8. Komórka dla `DO ZAPŁATY`: `= [adres ŁĄCZNA WARTOŚĆ USŁUG] - [adres UDZIELONY RABAT]`.
9. Pamiętać należy, aby ustawić na wspomnianych wyżej polach oraz w całym kosztorysie format wyświetlania odpowiedni dla Waluty (`zł` lub `PLN`). Formatuje się poprzez Formatuj komórki -> Kategoria: Walutowe -> Symbol: zł.

Na końcu zapis pliku jako `Kosztorys.xlsx` na odpowiednim przygotowanym Dysku USB egzaminatora.
