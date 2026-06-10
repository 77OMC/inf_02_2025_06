# Rozwiązanie egzaminu INF.02 - Sesja Czerwiec 2025, wersja arkusza 09 SG

## 1. Wykonaj montaż okablowania sieciowego

**Rozwiązanie:**
Zgodnie z wytycznymi w pliku `README.md` (Zasada A), polecenia dotyczące fizycznego okablowania są omijane. Dla informacji merytorycznej: wykonanie patchcordu U/UTP w układzie prostym według standardu T568B polega na użyciu wtyków 8P8C (RJ-45) przy ułożeniu pinów z obu stron kabla z zachowaniem odpowiedniej sekwencji (Biało-Pomarańczowy, Pomarańczowy, Biało-Zielony, Niebieski, Biało-Niebieski, Zielony, Biało-Brązowy, Brązowy). Kabel po zaciśnięciu weryfikuje się odpowiednim testerem pomiarowym.

## 2. Konfiguracja rutera (MikroTik)

**Rozwiązanie:**
Poniższe polecenia konfiguracyjne wykorzystują interfejs konsolowy (CLI) stosowany w MikroTiku (system RouterOS). W tym scenariuszu ruter działa standardowo jako kontroler połączenia bez zdefiniowanej na nim strefy punktu WLAN (jeżeli zadanie na PDFie tego nie nakłada).

**Konfiguracja CLI:**

```routeros
# Zmiana hasła dla nowej/zresetowanej maszyny w zależności od wersji systemu, standardowo dla konta z uprawnieniami
/user set admin password=NoweHaslo!

# 1. Konfiguracja interfejsu WAN (np. ether1): Adres, Maska /27 i Brama domyślna
/ip address add address=200.20.200.20/27 interface=ether1
/ip route add dst-address=0.0.0.0/0 gateway=200.20.200.1

# 2. Serwery DNS interfejsu
/ip dns set servers=7.7.7.7,6.6.6.6 allow-remote-requests=yes

# 3. Konfiguracja interfejsu LAN (np. ether2 lub prekonfigurowanego mostu bridge)
/ip address add address=192.168.7.1/24 interface=ether2

# 4. Serwer DHCP dla interfejsu LAN (Z ustawieniami zakresu ip na 192.168.7.11 - 192.168.7.20)
/ip pool add name=dhcp_pool_lan ranges=192.168.7.11-192.168.7.20
/ip dhcp-server add name=server_dhcp1 interface=ether2 address-pool=dhcp_pool_lan disabled=no
/ip dhcp-server network add address=192.168.7.0/24 gateway=192.168.7.1 dns-server=7.7.7.7,6.6.6.6

# 5. Rezerwacja adresu IP dla stacji roboczej (podano losowy przykładowy MAC: 1A:2B:3C:4D:5E:6F)
/ip dhcp-server lease add address=192.168.7.11 mac-address=1A:2B:3C:4D:5E:6F server=server_dhcp1
```

**Odczyt konfiguracji rutera (Weryfikacja):**

```routeros
# Sprawdzenie adresacji i domyślnej trasy
/ip address print
/ip route print
# Sprawdzenie serwera DHCP i listy wydzierżawionych adresów
/ip dhcp-server print
/ip dhcp-server lease print
```

## 3. Konfiguracja przełącznika (MikroTik)

**Rozwiązanie:**
(Przełącznik w prostej konfiguracji jako węzeł zarządzalny z IP dla strefy LAN).

**Konfiguracja CLI:**

```routeros
# Przypisanie IP i bramy na nadrzędny interfejs operacyjny zarządzania (przeważnie bridge domyślny)
/ip address add address=192.168.7.2/24 interface=bridge1
/ip route add dst-address=0.0.0.0/0 gateway=192.168.7.1
```

**Odczyt konfiguracji przełącznika (Weryfikacja):**

```routeros
/ip address print
/ip route print
```

## 4. Połączenie urządzeń sieciowych

*(Zadanie polegające na zapięciu kabli na podstawie schematu omijamy wedle obostrzeń na środowisko instrukcji A z README).*

## 5. Diagnostyka parametrów stacji roboczej (Windows)

Zadanie w arkuszu skierowane jest wprost pod system Windows i nie da się jednoznacznie przenieść oceny DirectX na natywny system C2 (Linux Ubuntu) - podano w tym kroku opis sprawdzania tych specyficznych danych dla O/S ze zlecenia oraz dla ewentualnego testu C2.

### C1 - Windows 10/11

Dane diagnostyczne, o które proszono pobierane są poprzez załączone programy i narzędzia systemowe. Wykonane na nich zrzuty ekranu trafiają do katalogu `Windows` umiejscowionego na napędzie USB.
**Sposoby odczytu (Weryfikacja):**

1. **System operacyjny (Wersja i kompilacja):**
   Uruchamiamy okno Uruchamiania (`Win + R`) i wklepujemy polecenie: `winver`. Drugą graficzną opcją jest wejście w "Ustawienia -> System -> Informacje" gdzie opisane są wersja podsystemu Windows i precyzyjny Build (kompilacja systemu).
2. **Procesor (Nazwa, Oznaczenie, Taktowanie bazowe):**
   Można go odczytać używając wpisu `msinfo32` w aplecie Uruchom. Taktowanie podstawowe znajduje się tamże, obok nazwy układu. Identyczne informacje uzyskamy z Menedżera zadań na karcie Wydajność, opcji "Procesor CPU".
3. **Karta graficzna (Rozmiar VRAM, Wersja DirectX):**
   Otwieramy Narzędzie Diagnostyczne DirectX poleceniem `dxdiag`.
   Zakładka System - widnieje "Wersja programu DirectX".
   Zakładka Ekran - pole "Pamięć ekranu (VRAM)". Oceniany komponent graficzny musi mieć wpis 3 GB+ (czyli m.in. 3072 MB lub więcej z odczytu). W wierszu polecenia (CMD) jest ucięte do innej karty komend (nie `dxdiag`), ale można wywołać podsumowanie `dxdiag /t diag.txt` a po tym przeszukać plik wpisem tekstowym.

Na bazie tych trzech punktów, należy sprawdzić, czy komputer spełnia wymagania z Tabeli 1 i na tej podstawie wprowadzić słowną, potwierdzoną uzasadnieniem Ocenę z testu, zapisując w arkuszu "Spełnia / Nie spełnia minimalnych wymogów, albowiem... [posiada za mało RAM Grafiki / stary system operacyjny itd]".

### C2 - Linux Ubuntu Stacja (Ekwiwalent poboru sprzętowego)

Chcąc pozyskać tożsame dane sprzętowe nie wykorzystując natywnie niedostępnego uwarunkowania pod kod DirectX:

```bash
# System, kompilacja Kernela
hostnamectl
uname -a

# Procesor (Taktowanie bazowe i model)
lscpu | grep "Model name"
lscpu | grep "CPU max MHz"

# Karta graficzna (Ilość dedykowanego VRAM i model, przy instalacji hwinfo i pakietów wsparcia)
sudo lshw -C display
glxinfo | grep -i "Video"
# Aby zbadać odpowiednik technologii dla 3D tj, wsparcie pod Vulkan lub OpenGL (Odpowiednik sprawdzania DX):
vulkaninfo | grep apiVersion
glxinfo -B | grep "OpenGL version string"
```

Uzyskane dane (zrzuty ekranów z terminali) archiwizuje się podobnie na docelowy pendrive.

## 6. Konfiguracja Stacji Roboczej

W poleceniu stacja to O/S Linux (Zgodnie z naszym trybem jest to rozwiązanie systemu C2, stąd podajemy na C1 odpowiednik).

### C2 - Linux Ubuntu Stacja

**Rozwiązanie:**

1. **Konfiguracja sieci (IP, DHCP):**
   - Po podpięciu kabla pod sieć do przełącznika, odświeżamy DHCP z routera za pomocą usługi w warstwie poleceń.
   - Identyfikujemy połączony profil komendą np. `nmcli connection show`. Prawdopodobnie "Połączenie przewodowe 1".
   - `nmcli connection modify "Połączenie przewodowe 1" connection.id "LAN K"`
   - Adres serwera DNS ustawiamy jako ręczny dla tego profilu (z uwzględnieniem faktu że Serwer z zadania będzie miał `192.168.7.3`):
   - `nmcli connection modify "LAN K" ipv4.dns 192.168.7.3 ipv4.ignore-auto-dns yes`
   - Odświeżamy połączenie i włączamy ponowny pobór IP za sprawą klienta DHCP: `nmcli connection down "LAN K" && nmcli connection up "LAN K"`
   - lub `sudo dhclient -r` a po zwolnieniu odpalając `sudo dhclient`.

2. **Konto i prawa:**
   - Dodajemy usera, tworząc u niego katalog domowy `-m`, wybierając mu narzuconą powłokę domyślną na sh `-s /bin/sh`:
   - `sudo useradd -m -s /bin/sh asystent`
   - Wywołujemy procedurę nadania hasła podanemu z polecenia `Syst@my1!`:
     `echo "asystent:Syst@my1!" | sudo chpasswd`
   - Włączenie logiki praw poleceń "root" przez pakiet Sudo:
     Wywołujemy `sudo visudo`. Pod sekcją uprawnień z root wklepujemy tożsamą składnię (bądź dopisujemy do grupy `sudo` o ile nie naruszy to standardowych obostrzeń):
     `asystent ALL=(ALL:ALL) ALL`

3. **Praca na plikach i archiwum Tar:**
   - Logujemy się z powrotem bądź przechodzimy na dysk twardy na prawa konta "administrator" wskazanego w zadaniu (konto to istnieje na początku O/S lub dodano w zleceniach C).
   - `mkdir /home/administrator/Kopie`
   - Kopiujemy docelowe obiekty z przygotowanego Pendrive / Napędu USB, który zamontowano w `/media/` (ścieżka jest zależna od wyciągu dystrybucji):
     `cp /media/usb/DOKUMENTACJA/PROGRAMY/PLIKI/inf02.txt /home/administrator/Kopie/`
     `cp /media/usb/DOKUMENTACJA/PROGRAMY/PLIKI/inf03.txt /home/administrator/Kopie/`
   - Tworzenie i zachowanie wyciągu archiwum w żądanym katalogu, ze standardem widzialności plików archiwizowanych (flaga `v`), ze wsparciem nazewnictwa dla poszczególnych danych wejściowych:
     `cd /home/administrator/Kopie`
     `tar -cvf informatyk.tar inf02.txt inf03.txt`
   - Zapis do Arkusza Tabela 3. Polecenie do tworzenia archiwum wygląda w opcji pełnej następująco: `tar -cvf informatyk.tar inf02.txt inf03.txt`

4. **Test z pingu:**
   Z poziomu zintegrowanego terminala:

   ```bash
   # Wysyłka paczki do modemu / rutera LAN:
   ping 192.168.7.1
   # Wysyłka na interfejs przełącznika:
   ping 192.168.7.2
   ```

**Weryfikacja poleceń (Bash):**

```bash
ip addr show
cat /etc/passwd | grep asystent
sudo -l -U asystent
ls -l /home/administrator/Kopie
tar -tvf /home/administrator/Kopie/informatyk.tar
```

---

### C1 - Windows 10/11

**Rozwiązanie (Odpowiednik dla Stacji C1, gdyby polecenie zakładało Windows):**

1. W Menedżerze kart `ncpa.cpl` zmieniamy nazwę na `LAN K`. Ustawiamy w IPv4 Automatyczne pobieranie adresu IP. Konfigurację na stałe serwera DNS aplikujemy w tym samym polu wprowadzając na sztywno pole Preferowanego Serwera jako `192.168.7.3`. Piszemy w CDM `ipconfig /renew`.
2. W Opcjach systemu otwieramy zarządzanie poprzez polecenie `compmgmt.msc`. W "Użytkownicy", nowe okno wprowadzamy nazwę "asystent", ustalamy hasło `Syst@my1!`. Dodajemy tego użytkownika do Lokalnej grupy "Administratorzy" jako odwzorowanie możliwości odpalania procesów o wysokich uprawnieniach z poleceń (równoznaczne uprawnieniom sudo). Powłoką logowania na Win przeważnie jest domyślnie `cmd` na plikach starszych systemów, w Windows nowszych PowerShell, brak tu pełnego równoznacznika do trybu tekstowego sh jako takiego, bez środowiska WSL.
3. Kopiowanie do ścieżki (C:\Users\Administrator\Kopie) robimy skrótem menedżera okien Windows.
4. Archiwizacja przez natywne polecenie okien (tar został zaimportowany w Windows 10 kompilacjach nowszych z basha), polega ona na włączeniu wiersza CMD, i identycznej składni w katalogu co z Linux: `tar -cvf informatyk.tar inf02.txt inf03.txt`.

## 7. Konfiguracja Serwera

W rozwiązaniach wykorzystywany jest podawany wyżej adres serwera LAN z opcji. Serwer na stacji instalacyjnej 09 w poleceniu ma zainstalowany system Windows Serwer.

### D1 - Windows Server 2022

**Rozwiązanie:**

1. **Adresacja interfejsu (LAN Z):**
   - Na urządzeniu otwieramy Panel Kart i zmieniamy wytypowaną przewodową płytę pod przełącznik jako `LAN Z`. Właściwości -> IPv4:
     - IP: `192.168.7.3`
     - Maska: `255.255.255.0`
     - Brama: `192.168.7.1`
     - DNS (lokalny do rozwiązywania stref we własnym serwerze): `127.0.0.1` (localhost).
   - Inny sprzętowy przewodowy interfejs deaktywujemy wciskając Prawy -> "Wyłącz".

2. **Przygotowanie strony (Web) z nośnika:**
   - Tworzymy w głównym korzeniu katalog `C:\Web`.
   - Z przenośnego dysku Pendrive kopiujemy zasób z dokumentacji / plików, mianowicie podaną witrynę zapisaną dla zadania `test.html` i wklejamy ją prosto we wcześniej ustalony folder `C:\Web`.

3. **Konfiguracja Rol serwerowych (IIS oraz Menedżera DNS):**
   - Poprzez wbudowany z lewej kreator instalacji Narzędzi oraz ról dołączamy "Serwer sieci Web (IIS)" oraz drugą pozycję "Serwer DNS". Uruchamiamy pobranie do końca.
   - Klikamy menu "Narzędzia -> DNS" (`dnsmgmt.msc`).
   - Pod Serwerem, prawy na "Strefy wyszukiwania do przodu". Klikamy pole na nową strefę, tworząc na ustawieniach Podstawową instalację bez połączenia z Active Directory o wybranej z zadania nazwie strefowej: `egzamin.local`.
   - Po jej zapisaniu podwójnym kliknięciem otwieramy listę i na białym tle wciskamy by opublikować dla Niej docelowy "Nowy Host (A lub AAAA)". Adres do którego strzelamy (Powiązanie z lokalnym LAN_Z): IP to `192.168.7.3`. Nazwa Hosta obok z boku widnieje jako skrót strefy z witryną czyli: `www`.

4. **Moduł web IIS (Zarządzanie Inetmgr):**
   - W pasku Start odpalamy IIS Manager (`inetmgr`).
   - Lewy spadochron menu ze zrzutami witryn, na podfolder "Witryny" włączamy z prawego opcję "Dodaj nową witrynę internetową".
   - Konfigurator:
     - Nazwa: `testowa`
     - Podanie Ścieżki Fizycznej z folderem źródłowym: Wybieramy lokalizację `C:\Web`.
     - W powiązaniach adres IP: wybieramy interfejs karty `192.168.7.3`, standardowy protokół HTTP łączy serwowanie danych poprzez Port 80.
     - Nazwa hosta musi zostać wypleniona w okienku: `www.egzamin.local`
   - Wracamy po zamknięciu do wewnątrz menu "testowa". Z ustawień odnajdujemy zbiór na dokumenty priorytetów zwany "Dokument Domyślny".
   - W okienku Dokumentów wciskamy pole widoczne obok i Dodajemy do priorytetu naszą nazwę odwołującą strone do pobrania czyli plik web z foldera pt. `test.html`. By aplikacja serwowała stronę poprawnie, przemieszczamy z listy pozycje w górę (wyżej).

5. **Przeglądarka internetowa:**
   - Z komputera serwera z użyciem przeglądarki Edge pod paskiem witryn wejdź w adres i skontroluj dostęp z zapytaniem: `http://www.egzamin.local`. Jeśli załadowano kod `test.html` operacja powiodła się.

**Weryfikacja parametrów (PowerShell):**

```powershell
Get-NetIPAddress -InterfaceAlias "LAN Z"
Resolve-DnsName -Name www.egzamin.local
Import-Module WebAdministration
Get-Website
```

---

### D2 - Linux Ubuntu Server

**Rozwiązanie (Odpowiednik systemu powłok na C2):**

1. **Konfiguracja IP i dezaktywacja innych adapterów z użyciem narządzia configu (Netplan):**
   - Zmiana w `/etc/netplan/01-netcfg.yaml`:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      set-name: LAN Z
      addresses: [192.168.7.3/24]
      routes:
        - to: default
          via: 192.168.7.1
      nameservers:
        addresses: [127.0.0.1]
    ens34:
      activation-mode: off
```

- Zaaplikowanie planu: `sudo netplan apply`

1. **Kopiowanie folderu Web na Linuks:**
   - `sudo mkdir /Web`
   - Kopiujemy z media pendrive'a test.html do utworzonej sekcji:
     `sudo cp /media/pendrive/DOKUMENTACJA/PROGRAMY/PLIKI/test.html /Web/`

2. **Serwer HTTP/DNS z wykorzystaniem pakietów Apache2 i BIND9:**
   - Pobieramy paczki `sudo apt update && sudo apt install -y bind9 bind9utils apache2`.
   - Moduł DNS. Otwarcie strefowej przestrzeni `sudo nano /etc/bind/named.conf.local` z wpisaniem na dół nowej logiki (do przodu):

```text
zone "egzamin.local" {
    type master;
    file "/etc/bind/db.egzamin.local";
};
```

- Opublikowanie danych HostA dla nowego modułu (Kopiujemy `db.local` jako wzór strefy):
     `sudo cp /etc/bind/db.local /etc/bind/db.egzamin.local`
     W `db.egzamin.local` wstawiamy wpis do sieci o budowie rekordu by skojarzył lokalny adres na: `www IN A 192.168.7.3`. Restarty usług serwerowych serwisu. `sudo systemctl restart bind9`.
- Web Server modyfikacja v-host w Apache. Tworzymy nowy plik z definicją `sudo nano /etc/apache2/sites-available/testowa.conf`:

```apache
<VirtualHost 192.168.7.3:80>
    ServerName www.egzamin.local
    DocumentRoot /Web
    DirectoryIndex test.html
</VirtualHost>
```

- Dezaktywowanie domyślnej usługi http web po tym interfejsie i wywołanie utworzonego hosta `sudo a2dissite 000-default.conf` następnie z załadowaną flagą `sudo a2ensite testowa.conf`. Wykonaj szybki restart na nową maszynę Apache. Weryfikacja przez `curl http://www.egzamin.local`.

**Weryfikacja poleceń (Bash):**

```bash
ip a show "LAN Z"
dig @127.0.0.1 www.egzamin.local
cat /etc/apache2/sites-available/testowa.conf
```

## 8. Utworzenie Skryptu Wsadowego BAT (Windows)

Zadanie opiewa o napisanie krótkiego pliku w korzeniu dysku Systemowego Serwera (C:\) będącym pętlą dla operacji modyfikacji folderów o danej inkrementacji.

**Rozwiązanie (kod wsadowy dla Windows `.bat`):**
Otwieramy Notatnik (w prawach administratora), w którym podajemy polecenia skryptowe z zadania, tak aby je wypełniały (Polecenia Wyświetlanie oraz Tworzenia). Kod zapisujemy pod lokalizacją z uprawnieniami na katalog główny dysku twardego Systemowego serwera, czyli C:\plik1.bat (zmieniając podczas zapisu opcje w Windowsie by zapis jako typ był widoczny na "Wszystkie Pliki *.*").

Zawartość pliku `.bat`:

```bat
@echo off
echo tworzenie folderow dla studentow
mkdir C:\TEST
cd /d C:\TEST
for /l %%X in (1, 1, 10) do (
    mkdir STUDENT%%X
)
```

Tłumaczenie: Na ekran wrzucony jest tekst "tworzenie...". Potem powstaje katalog TEST na dysku. Następuje polecenie FOR /L oznaczające w Windowsie pętle (Parametry: Początek czyli 1, Inkrementacja o 1, Zakończenie limitu jako zliczenie po wykonaniu 10). Wykonanie wewnątrz zawartości bloków jest stworzeniem katalogu STUDENT oraz sklejeniem ze stanem z licznika X pętli, tworząc wymóg dla folderów STUDENT1, STUDENT2 itp..
Po utworzeniu uruchamiamy plik wsadowy kliknięciem dwa razy aby zweryfikować czy katalog powziął wymagane przez dokument i pętle zadania.
*(W przypadku konieczności przygotowania pliku pod O/S C2 Ubuntu w formie `plik1.sh`: Kod to odpowiednik powłoki `mkdir -p /TEST && for i in {1..10}; do mkdir /TEST/STUDENT$i; done`).*
