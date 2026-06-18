# Rozwiązanie egzaminu INF.02 - Sesja Czerwiec 2025, wersja arkusza 05 SG

## 1. Wykonaj montaż okablowania sieciowego

**Rozwiązanie:**
Zgodnie z wytycznymi w pliku `README.md` (Zasada A), pomijamy zadania dotyczące fizycznego montażu okablowania sieciowego. W realiach egzaminacyjnych należałoby przygotować patchcord U/UTP z wtykami RJ-45 (8P8C) skrosowany (z przeplotem). W tym celu z jednej strony używa się sekwencji T568A, a z drugiej T568B.

## 2. Konfiguracja rutera (MikroTik)

**Rozwiązanie:**
Konfiguracja sprzętu MikroTik z poziomu CLI (lub WinBox). W przykładzie wykorzystano CLI systemu RouterOS.
*(Zmienna `XX` oznacza numer stanowiska egzaminacyjnego, do celów testowych przyjęto stanowisko `05`).*

**Konfiguracja CLI:**

```routeros
# Zmiana hasła (jeśli wymuszona przy pierwszym logowaniu)
/user set admin password=NoweHaslo123

# Adres IP interfejsu WAN (ether1) i brama domyślna:
/ip address add address=100.0.0.4/24 interface=ether1
/ip route add dst-address=0.0.0.0/0 gateway=100.0.0.1

# Serwer DNS dla interfejsu WAN:
/ip dns set servers=4.4.4.4,4.4.5.5 allow-remote-requests=yes

# Adres IP interfejsu LAN1 (ether2 lub zintegrowany bridge, zakładamy LAN na bridge'u):
/ip address add address=192.168.1.1/28 interface=bridge_lan

# Serwer DHCP - wyłączony
/ip dhcp-server remove [find]

# Konfiguracja sieci bezprzewodowej 2.4 GHz (wlan1):
# Utworzenie profilu zabezpieczeń z uwierzytelnianiem WPA2-PSK i odpowiednim szyfrowaniem:
/interface wireless security-profiles add name=sec_egzamin authentication-types=wpa2-psk wpa2-pre-shared-key=Qwerty_05 mode=dynamic-keys

# Ustawienie karty sieciowej wlan1: (zakładamy numer kanału 5 co odpowiada 2432 MHz).
/interface wireless set wlan1 disabled=no mode=ap-bridge ssid=Egzamin_05 band=2ghz-b/g/n frequency=2432 security-profile=sec_egzamin
```

**Odczyt konfiguracji rutera (Weryfikacja):**

```routeros
# Sprawdzenie adresacji IP:
/ip address print
# Sprawdzenie routingu:
/ip route print
# Sprawdzenie konfiguracji DNS:
/ip dns print
# Sprawdzenie serwera DHCP:
/ip dhcp-server print
# Sprawdzenie ustawień sieci WiFi oraz jej profilu security:
/interface wireless security-profiles print
/interface wireless print advanced
```

## 3. Połączenie urządzeń sieciowych

*(Zadanie pominięto zgodnie z uwagą A w pliku README).*

## 4. Identyfikacja parametrów

Zadanie w arkuszu skierowane jest wprost pod system Windows. Ze względu na wariantowość egzaminu podajemy również rozwiązanie dla systemu Linux Ubuntu. Wszystkie zrzuty zapisywane są na nośnik przenośny do folderu `Identyfikacja`.

### C1 - Windows 10/11

Należy pozyskać dane dotyczące producenta dysku twardego i jego pojemności, a także dane RAM.
**Polecenia CMD / PowerShell (Weryfikacja):**

```cmd
:: Dysk twardy (Producent i pojemność)
wmic diskdrive get Manufacturer, Model, Size

:: Pamięć RAM (Pojemność i częstotliwość)
wmic memorychip get capacity, speed
```

Zapisujemy wyjście komend narzędziem 'Wycinanie' na dysk `Egzamin-x`. Rozmiary z bajtów na dysku lub RAM przeliczamy na GB dzieląc odpowiednio przez `1024^3` (RAM) lub standardowo wg podziałów 1000/1024. Wartości umieszczamy w Tabeli 1.

### C2 - Linux Ubuntu Stacja

Dane podzespołów w linuksie pozyskuje się za pomocą dedykowanych programów linii poleceń.
**Polecenia do odczytu ustawień (Weryfikacja):**

```bash
# Dysk twardy:
lsblk -d -o name,model,size
# lub szczegółowo:
sudo lshw -class disk

# Pamięć RAM:
sudo dmidecode --type memory | egrep "Size|Speed"
```

Uzyskane dane ze zrzutów okna ekranu zachowujemy na dysku USB i wprowadzamy do Tabeli 1.

## 5. Konfiguracja stacji roboczej

(Zmienna stanowiska dla ułatwienia przyjęta jako `XX` = 05).

### C1 - Windows 10/11

**Rozwiązanie:**

1. **Nazwa komputera, sieć i domena:**
   - Panel sterowania -> Centrum sieci i udostępniania -> Zmień ustawienia karty sieciowej (`ncpa.cpl`).
   - Modyfikacja nazwy karty Wi-Fi na `W-LAN`.
   - Zmiana opcji protokołu IPv4 na adres statyczny:
     - IP: `192.168.1.2`
     - Maska dla /28: `255.255.255.240`
     - Brama: `192.168.1.1`
     - DNS: `4.4.4.4`
   - Połączenie się z wyświetloną po wyszukaniu siecią z rutera `Egzamin_05` i uwierzytelnienie hasłem `Qwerty_05`.
   - Zmiana nazwy komputera w `O komputerze` -> `Zaawansowane ustawienia systemu` -> `Nazwa komputera` na `PCuczen`. System musi zostać zrestartowany.

2. **Zarządzanie kontami użytkowników i grupami:**
   - Otwieramy Zarządzanie Komputerem (`compmgmt.msc`) -> Narzędzia systemowe -> Użytkownicy i grupy lokalne.
   - W sekcji *Użytkownicy* znajdujemy zablokowane konto `uczen`.
   - Prawy klik -> Właściwości. Odznaczamy "Konto jest wyłączone".
   - Zaznaczamy "Użytkownik nie może zmienić hasła" oraz "Hasło nigdy nie wygasa". Zatwierdzamy.
   - Prawy klik na użytkowniku `uczen` -> Ustawianie hasła -> Nadajemy hasło `QweRTY!1`.
   - W sekcji *Grupy*, klikamy na pustym polu prawym -> Nowa grupa -> Nazwa grupy: `egzamin`. Dodajemy w menu tworzenia członka wpisując `uczen`.

3. **Przydziały dyskowe (Konta Disk Quotas):**
   - Wchodzimy do "Ten Komputer". Prawym kliknięciem na dany dysk instalacyjny (np. `C:`) wybieramy Właściwości.
   - Przechodzimy na zakładkę "Przydział" -> "Pokaż ustawienia przydziału".
   - Zaznaczamy opcje: "Włącz zarządzanie przydziałami" oraz "Odmów miejsca na dysku użytkownikom przekraczającym limit przydziału". Następnie klikamy "Wpisy przydziałów".
   - W nowym oknie -> "Nowy wpis przydziału..." -> wpisujemy użytkownika `uczen`.
   - Ograniczamy mu miejsce na dysku do obliczonych 20% pojemności dysku stacji roboczej (np. dla dysku 50 GB ustawiamy limit na 10 GB).

4. **Wymóg Ctrl+Alt+Del:**
   - Wywołujemy w polu Uruchom: `netplwiz`. Przechodzimy do zakładki "Zaawansowane". Zaznaczamy na dole opcję "Wymagaj od użytkowników naciśnięcia klawiszy Ctrl+Alt+Delete". Zatwierdzamy.

**Polecenia do weryfikacji ustawień (CMD/PowerShell):**

```powershell
# Interfejs i adresy IP
ipconfig /all
# Ustawienia hasła i restrykcje dla konta
net user uczen
# Grupa
net localgroup egzamin
# Reguła CTRL+ALT+DEL
Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "DisableCAD"
```

---

### C2 - Linux Ubuntu Stacja

**Rozwiązanie:**

1. **Ustawienia sieci WLAN (`W-LAN`):**
   - Wykorzystując komendę menadżera sieci:
   - `nmcli connection add type wifi con-name W-LAN ifname wlan0 ssid Egzamin_05`
   - `nmcli connection modify W-LAN wifi-sec.key-mgmt wpa-psk wifi-sec.psk Qwerty_05`
   - `nmcli connection modify W-LAN ipv4.addresses 192.168.1.2/28 ipv4.gateway 192.168.1.1 ipv4.dns 4.4.4.4 ipv4.method manual`
   - `nmcli connection up W-LAN`

2. **Zmiana nazwy komputera:**
   - `sudo hostnamectl set-hostname PCuczen`

3. **Konta, grupy, hasła (Linux):**
   - Prawdopodobnie konto ma zmodyfikowaną datę wygaśnięcia z przeszłości lub ma postać zablokowaną wykrzyknikiem. Odblokowujemy:
     `sudo usermod -U uczen` ewentualnie zmieniamy shell jeśli ustawiony był jako `nologin`: `sudo usermod -s /bin/bash uczen`.
   - Hasło ustawiamy komendą: `echo "uczen:QweRTY!1" | sudo chpasswd`
   - Data ważności konta/hasła: `sudo chage -m 0 -M 99999 -I -1 -E -1 uczen`.
   - Tworzymy grupę i dodajemy użytkownika:
     `sudo groupadd egzamin`
     `sudo usermod -a -G egzamin uczen`

4. **Konta Disk Quotas na dysku root (`ext4`):**
   - Opcja włączona w partycji: Edycja `/etc/fstab` -> dodanie atrybutu `usrquota` np. dla `/`.
   - Przeładowanie opcji przydziałów: `sudo mount -o remount /`.
   - Inicjalizacja sprawdzania i utworzenie plików kwot dyskowych: `sudo quotacheck -cum /`.
   - Włączenie systemu kwot dyskowych: `sudo quotaon -v /`.
   - Konfiguracja restrykcji dla `uczen` z ograniczeniem na np. pojemności rootfs na 20%:
     `sudo edquota -u uczen` (Otworzy edytor i należy w rubrykach dysku ustawić `hard limit` na wielkość odpowiadającą 20% limitu bloku w kilobajtach, co spowoduje nałożenie bezwzględnego bloku "odmowy miejsca").

5. **Kombinacja GDM/Logowanie (Ekwiwalent):**
   - W klasycznym Linuksie logowanie za pomocą klawiatury na pełnym środowisku GNOME wymaga modyfikacji pliku lub nakładki powitania. Często stosuje się blokadę i baner ekranowy, ale brak w nim domyślnego flagowania wciśnięcia kombinacji klawiszy jak w Windows (z racji na środowisko graficzne). W takich scenariuszach weryfikowane jest jedynie zabezpieczenie na Windows, jednak konfigurację modyfikacji w GDM załatwia paczka `dconf`.

**Weryfikacja parametrów (Bash):**

```bash
nmcli connection show W-LAN
hostnamectl
grep egzamin /etc/group
sudo chage -l uczen
sudo repquota /
```

## 6. Konfiguracja serwera

### D1 - Windows Server 2022

**Rozwiązanie:**
(Przetłumaczenie powłok bash do natywnych rozwiązań w Windows Server zgodnie z poleceniem)

1. **Interfejsy i nazwa komputera:**
   - W `ncpa.cpl` zmieniamy nazwę na `LAN1`.
   - IPv4 w Właściwościach: IP `192.168.1.3`, Maska `255.255.255.240` (dla /28), Brama `192.168.1.1`, Serwer DNS `4.4.4.4`.
   - Zmiana nazwy serwera we Właściwościach systemu (`sysdm.cpl`) na `PCserwer`.

2. **Katalogi i uprawnienia z Bash na NTFS:**
   - Tworzymy w głównym dysku instalacyjnym katalog (ponieważ w windows nie używamy `/home` a domyślnie `C:\Users` dla stacji profilowych, utworzymy dedykowany folder np na dysku C): `C:\home\zadanie`.
   - Utworzenie pliku: Otwieramy systemowy notatnik. Wpisujemy tekst `INF.02`. Zapisujemy jako plik tekstowy w lokalizacji `C:\home\zadanie\dane.txt` (lub bez rozszerzenia).
   - Uprawnienia dostępu - plik `dane`: Odpowiednik kodu `660` (Właściciel: odczyt+zapis, Grupa: odczyt+zapis, Inni: brak dostępu).
   - Katalog `zadanie`: `770` (Właściciel: Pełna kontrola, Grupa: Pełna kontrola, Inni: brak).

3. **Użytkownicy i przynależności (lokalne):**
   - Otwieramy Zarządzanie komputerem `compmgmt.msc`. W menu zarządzania Użytkownikami lokalnymi tworzymy użytkownika z logiem `egzamin`, hasłem `Egz@min12!`. Data wygaśnięcia nie może zostać bezpośrednio dodana z panelu GUI w przypadku logowania domowego, lecz da się założyć w Active Directory (brak roli AD w zadaniu). Datę 15 luty 2029 dla konta stacji narzuca się poprzez polecenie konsoli: `net user egzamin /expires:15/02/2029`.
   - We Właściwościach katalogu `C:\home\zadanie` klikamy na Zabezpieczenia -> Zaawansowane -> Właściciel. Zmieniamy go na użytkownika `egzamin`. W przypadku grupy dodajemy w panelu grupę docelową (Zamiast root będzie to grupa Administratorzy na OS Windows) i przypisujemy jej właściwe odczyty. Wyłączamy dziedziczenie. Pozostali użytkownicy ("Wszyscy") są z tego katalogu całkowicie usunięci.

**Weryfikacja parametrów (PowerShell):**

```powershell
Get-NetIPAddress -InterfaceAlias LAN1
hostname
net user egzamin
Get-Acl -Path C:\home\zadanie | Format-List
Get-Acl -Path C:\home\zadanie\dane.txt | Format-List
```

---

### D2 - Linux Ubuntu Server

**Rozwiązanie:**

1. **Konfiguracja IP (`netplan`) i Hostname:**
   - Plik konfiguracyjny (np. `/etc/netplan/01-netcfg.yaml`):

```yaml
network:
  version: 2
  ethernets:
    ens33:
      set-name: LAN1
      addresses: [192.168.1.3/28]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [4.4.4.4]
```

- Zaaplikowanie planu: `sudo netplan apply`
- Ustawienie nazwy komputera: `sudo hostnamectl set-hostname PCserwer` i zmiana pliku `hosts`.

1. **Katalogi, pliki oraz uprawnienia:**
   - `sudo mkdir -p /home/zadanie`
   - `echo "INF.02" | sudo tee /home/zadanie/dane`
   - Modyfikacja praw dostępu do zasobu w systemie octal (Ósemkowym):
     `sudo chmod 770 /home/zadanie`
     `sudo chmod 660 /home/zadanie/dane`

2. **Użytkownik, wygaśnięcie i atrybuty właściciela:**
   - Tworzenie: `sudo useradd -m -s /bin/bash egzamin`
   - Hasło: `echo "egzamin:Egz@min12!" | sudo chpasswd`
   - Ustalenie daty wygasania konta (na YYYY-MM-DD): `sudo chage -E 2029-02-15 egzamin`
   - Modyfikacja właściciela i grupy (`chown` owner:group):
     `sudo chown egzamin:root /home/zadanie`

**Weryfikacja parametrów (Bash):**

```bash
ip a show LAN1
hostnamectl
ls -ld /home/zadanie
ls -l /home/zadanie/dane
sudo chage -l egzamin
```

## 7. Test komunikacji

**Rozwiązanie:**
Zadanie wymaga wykonania polecenia pingu za pomocą terminala ze stacji roboczej połączonej bezprzewodowo. Egzaminator po zawołaniu winien zobaczyć wynik z terminala po wykonaniu tych poleceń.
Na stacji roboczej (Dla Windowsa CMD/PowerShell, dla Linuxa terminal) wydajemy polecenie:

```cmd
ping 192.168.1.3    (Serwer PCserwer)
ping 192.168.1.1    (Interfejs rutera)
```

Po uzyskaniu prawidłowych odpowiedzi ICMPv4 i otrzymaniu informacji zwrotnej (np. `Odpowiedź z...`), możemy powiadomić egzaminatora o gotowości do ponownego puszczenia testu.

## 8. Tabela z adresacją sieci LAN

Na podstawie dostarczonego i ustanowionego wcześniej środowiska, wypełniona "Tabela 2. Adresacja sieci LAN" za pomocą kalkulatora IPv4 dla IP 192.168.1.1 z zadaną maską CIDR /28.

| Parametr | Rozwiązanie |
| --- | --- |
| Adres sieci | `192.168.1.0` |
| Maska sieci w notacji kropkowo-dziesiętnej | `255.255.255.240` |
| Adres rozgłoszeniowy | `192.168.1.15` |
| Maksymalna liczba hostów w sieci | `14` (wynik z potęgi 2^4 - 2) |
| Adres IP interfejsu LAN1 rutera | `192.168.1.1` |
| Adres IP interfejsu W-LAN stacji roboczej | `192.168.1.2` |
| Adres IP interfejsu LAN1 serwera | `192.168.1.3` |
