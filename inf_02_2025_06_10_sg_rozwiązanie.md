# Rozwiązanie egzaminu INF.02 - Sesja Czerwiec 2025, wersja arkusza 10 SG

## 1. Wykonaj montaż okablowania sieciowego

**Rozwiązanie:**
Zgodnie z wytycznymi w pliku `README.md` (Zasada A), polecenia dotyczące fizycznego okablowania są omijane. W przypadku prac na środowisku praktycznym, wykonanie patchcordu U/UTP w układzie prostym według standardu T568A polega na użyciu wtyków 8P8C (RJ-45) przy ułożeniu pinów z obu stron kabla z zachowaniem sekwencji: Biało-Zielony, Zielony, Biało-Pomarańczowy, Niebieski, Biało-Niebieski, Pomarańczowy, Biało-Brązowy, Brązowy. Kabel zaciśnięty zaciskarką powinien zostać sprawdzony testerem pod kątem przejść między pinami.

## 2. Konfiguracja rutera (MikroTik)

**Rozwiązanie:**
Poniższe polecenia konfiguracyjne zostały utworzone w interfejsie konsolowym (CLI) dedykowanym systemowi MikroTik RouterOS. Przed ich wpisaniem należy upewnić się, czy wymagana jest zmiana hasła.

**Konfiguracja CLI:**

```routeros
# Opcjonalna zmiana hasła:
/user set admin password=NoweHaslo!

# 1. Konfiguracja interfejsu WAN (np. ether1): Adres 200.20.200.20/27 i Brama 200.20.200.1
/ip address add address=200.20.200.20/27 interface=ether1
/ip route add dst-address=0.0.0.0/0 gateway=200.20.200.1

# 2. Serwery DNS dla WAN
/ip dns set servers=7.7.7.7,6.6.6.6 allow-remote-requests=yes

# 3. Konfiguracja interfejsu LAN (np. bridge1 lub przypisany port switchowy) - adres 172.16.7.1/24
/ip address add address=172.16.7.1/24 interface=bridge1

# 4. Serwer DHCP na włączonym zakresie dzierżawy: 172.16.7.11 ÷ 172.16.7.20
/ip pool add name=pool_lan ranges=172.16.7.11-172.16.7.20
/ip dhcp-server add name=dhcp_lan interface=bridge1 address-pool=pool_lan disabled=no
/ip dhcp-server network add address=172.16.7.0/24 gateway=172.16.7.1 dns-server=7.7.7.7,6.6.6.6

# 5. Rezerwacja adresu IP 172.16.7.11 dla stacji roboczej (Dla przykładu użyto fikcyjnego adresu MAC)
/ip dhcp-server lease add address=172.16.7.11 mac-address=11:22:33:44:55:66 server=dhcp_lan
```

**Odczyt konfiguracji rutera (Weryfikacja):**

```routeros
# Sprawdzenie zaadresowania interfejsów, bramy DNS oraz DHCP
/ip address print
/ip route print
/ip dhcp-server print
/ip dhcp-server lease print
```

## 3. Konfiguracja przełącznika (MikroTik)

**Rozwiązanie:**
(Prosta konfiguracja Switcha zarządzalnego L2).

**Konfiguracja CLI:**

```routeros
# Przypisanie IP i bramy do interfejsu, by można było na niego wejść po L3 (do portów nietagowanych)
/ip address add address=172.16.7.2/24 interface=bridge1
/ip route add dst-address=0.0.0.0/0 gateway=172.16.7.1
```

**Odczyt konfiguracji przełącznika (Weryfikacja):**

```routeros
/ip address print
/ip route print
```

## 4. Połączenie urządzeń sieciowych

*(Zadanie pominięte zgodnie z uwagą A w pliku `README.md`). Schemat nakazywałby połączenie Rutera, Przełącznika, Serwera, Stacji kablem prostym po miedzianej infrastrukturze.*

## 5. Diagnostyka parametrów stacji roboczej (Windows)

Ocenę stacji roboczej na arkuszu nakazuje się zrobić korzystając z środowiska Windows, jako C1 opisujemy weryfikację na Windows. Względem polecenia `C` i alternatywnych wymogów (np. co by było gdyby systemem był Ubuntu Linux na C2) przygotowano opcje zastępcze. Zdjęcia po zrzucie (za pomocą systemowego "Wycinanie") zapisujemy do katalogu `Windows` na podpiętym przez USB przenośnym nośniku (pendrive) i uzupełniamy "Tabelę 2" na arkuszu egzaminu.

### C1 - Windows 10/11

Odczyt do zrzutów:

1. **Karta Graficzna (DirectX / Pamięć VRAM):**
   Wciskamy `Win + R` -> `dxdiag`. Przechodzimy w zakładkę Ekran.
   - Pamięć VRAM w Mb przeliczona z VRAM (np. 3072 MB = 3GB).
   - Obsługiwana Wersja DirectX wyświetla w oknie System lub pod funkcjami akceleracyjnymi.
2. **Procesor (Nazwa / Taktowanie):**
   Uruchamiamy okno wiersza poleceń `CMD` i piszemy `wmic cpu get name, MaxClockSpeed`. Wynik ukazuje nazwę modelu CPU oraz Taktowanie podstawowe MHz.
   (Można to zweryfikować z poziomu `msinfo32.exe` podając pole Procesor).
3. **System Operacyjny (Nazwa i typ / Wersja kompilacji):**
   Uruchamiamy `winver` dla samej kompilacji wersji, oraz "Informacje o Systemie" (`msinfo32` -> opcja Nazwa systemu operacyjnego oraz Typ systemu x64-based PC).

Oceną komputera, zgodnie z Tabelą 1, musi być jednoznaczny wniosek: "Spełnia / Nie spełnia minimalnych wymogów, ponieważ..." odnosząc się bezpośrednio do porównania wymaganych wartości do odczytanych na ekranie.

### C2 - Linux Ubuntu Stacja (Ekwiwalent poboru sprzętowego)

Chcąc pozyskać tożsame dane sprzętowe pod odpowiednik C2 na Linuks:

```bash
# Karta Graficzna (Podsumowanie dedykowanego VRAM i wersji renderowania - brak czystego DX)
sudo lshw -C display
glxinfo -B | grep "OpenGL version string"

# Procesor (Model, taktowanie Max/Min/Bazowe)
lscpu | grep "Model name"
lscpu | grep "MHz"

# System, kompilacja Kernela
uname -a
hostnamectl
cat /etc/os-release
```

Z tych komend dokonujemy zrzutu terminala i ewaluujemy tabelę.

## 6. Konfiguracja Stacji Roboczej

W poleceniu stacja to O/S Linux (Zgodnie z naszym trybem jest to rozwiązanie systemu C2, stąd podajemy na C1 odpowiednik).

### C2 - Linux Ubuntu Stacja

**Rozwiązanie:**

1. **Konfiguracja sieci (`LAN K` z przydziałem automatycznym):**
   - Po podpięciu stacji kablem, wykorzystujemy terminal z NetManger by zarządzić nową nazwą interfejsu (lub zmieniamy to nakładką).
   - `nmcli connection modify "Połączenie przewodowe 1" connection.id "LAN K"`
   - Zmiana opcji ustawiania DNS tak aby, po przypisaniu IP przez DHCP, resolver czerpał informacje o hostach bezpośrednio z podanego dla Serwera w LAN_Z.
   - `nmcli connection modify "LAN K" ipv4.dns 172.16.7.3 ipv4.ignore-auto-dns yes`
   - Odświeżamy DHCP zmuszając adapter do zrzucenia przypisanego IP i odzyskania ponownej opcji z Rutera (Powinien przydzielić stały wpis zarezerwowany 172.16.7.11 dla stacji).
   - `sudo dhclient -r && sudo dhclient` lub polecenie w nmcli: `nmcli connection down "LAN K" && nmcli connection up "LAN K"`.

2. **Konto i prawa, powłoka:**
   - Tworzenie konta, zmiana powłoki oraz ustanawianie loginu w systemie:
   - `sudo useradd -m -s /bin/sh asystent`
   - Nakładamy docelowe hasło wywołując pipe: `echo "asystent:Syst@my1!" | sudo chpasswd`
   - Zezwolenie na logikę uruchamiania wszystkich procesów root w trybie visudo:
     Wywołujemy `sudo visudo`. W otwartym skrypcie nano/vi pod sekcją wpisów (gdzie np. widnieje root ALL=(ALL:ALL) ALL) dopisujemy komendę by nadać najwyższe pozwolenie userowi:
     `asystent ALL=(ALL:ALL) ALL`

3. **Praca na plikach i archiwum Tar:**
   - Powrót na podane konto (względem zadania: `su - administrator`).
   - `mkdir /home/administrator/Kopie`
   - Podłączamy nośnik USB. Mountujemy partycję sdb1/sdd1 i z poziomu USB odszukujemy folder z testowymi plikami. Następnie podrzucamy je w rzeczone miejsce.
     `cp /media/usb/DOKUMENTACJA/PROGRAMY/PLIKI/inf02.txt /home/administrator/Kopie/`
     `cp /media/usb/DOKUMENTACJA/PROGRAMY/PLIKI/inf03.txt /home/administrator/Kopie/`
   - Załączamy archiwizator poleceniem na plik typu .tar z flagą -v ukazującą z czym pracuje:
     `cd /home/administrator/Kopie`
     `tar -cvf informatyk.tar inf02.txt inf03.txt`
   - W ramach weryfikacji należy do **Tabeli 3** wklepać właśnie te wywołanie z wszystkimi wpisanymi opcjami, czyli: `tar -cvf informatyk.tar inf02.txt inf03.txt`.

4. **Test z pingu:**
   Z poziomu zintegrowanego terminala wysyłamy:

   ```bash
   # Ping na LAN Rutera:
   ping 172.16.7.1
   # Ping na Przełącznik:
   ping 172.16.7.2
   ```

**Weryfikacja poleceń (Bash):**

```bash
ip addr show
grep asystent /etc/passwd
sudo -l -U asystent
ls -l /home/administrator/Kopie
tar -tvf /home/administrator/Kopie/informatyk.tar
```

---

### C1 - Windows 10/11

**Rozwiązanie (Odpowiednik dla Stacji C1, gdyby polecenie zakładało Windows):**

1. W Menedżerze kart sieciowych `ncpa.cpl` zmieniamy opcje z nazwy Połączenie lokalne na `LAN K`. Ustawiamy w IPv4 Automatyczne pobieranie adresu IP. Konfigurację na stałe serwera DNS aplikujemy w tym samym polu wprowadzając ręcznie preferowany wpis Serwera D1, czyli np `172.16.7.3`. Piszemy w CMD `ipconfig /release` i `ipconfig /renew`.
2. W Opcjach systemu otwieramy zarządzanie przez okno poleceń `compmgmt.msc`. W menu lokalnych "Użytkownicy", nowe okno wprowadzamy nazwę "asystent", podpinamy hasło `Syst@my1!`. Dodajemy użytkownika do grupy "Administratorzy" nadającej wysokie przywileje odpalania procesów UAC. Powłoka systemowa domyślnie jest obsługiwana graficznie/z CMD a z sh powiązano jedynie polecenia środowiska programistycznego linuks z subsystem.
3. Kopiowanie z przenośnego pliku nośnikowego (jak USB/DVD) do `C:\Users\Administrator\Kopie` ręcznie.
4. Archiwizacja przez terminal (Win 10+) wykonuje te samo połączenie modułu tar co w Unix, ewentualnie opcją do zamiany staje się system PowerShell Compress-Archive. Dla czystej tabeli: uruchomić wiesz i użyć `tar -cvf informatyk.tar inf02.txt inf03.txt`.

## 7. Konfiguracja Serwera (Windows Serwer D1 i Linux D2)

### D1 - Windows Server 2022

**Rozwiązanie:**

1. **Adresacja interfejsu (LAN Z):**
   - Otwieramy opcje połączeń Sieciowych `ncpa.cpl`.
   - Identyfikujemy po MACu kartę która łączy serwer ze switchem. Nadajemy opcję zmiany "Zmień nazwę" i ustalamy jako `LAN Z`.
   - Z jej "Właściwości" obieramy Protokół IPv4. Oznaczamy stały IP:
     - IP: `172.16.7.3`
     - Maska: `255.255.255.0`
     - Brama: `172.16.7.1` (Z adresu IP rutera/bramy sieci docelowej, należy upewnić się o połącznie między LANem z routingiem między pulami przez co Windows Server połączy pakiety. Tu 172.16.7.1 według zaleceń testowych sieci WAN/LAN i braku innej drogi).
     - DNS z opcją na: `127.0.0.1` (localhost).
   - Drugą sprzętową, przewodową płytkę deaktywujemy z opcji jako Prawy Klawisz -> Wyłącz.

2. **Katalog strony C:\Web i powiązanie z Menedżerem IIS:**
   - Zakładamy na Systemowym roocie Windows folder o nazwie `C:\Web`.
   - Kopiujemy do niego testowe pliki strony wprost ze zmontowanej napędu: `test.html`.
   - Na serwerze otwieramy "Zarządzanie Serverem". Dodajemy Role serwera web (IIS) oraz rolę wirtualnej rozgłośni DNS. Restart nieobowiązkowy.

3. **Strefa Serwera DNS (`dnsmgmt`):**
   - Otwieramy Menedżera z "Narzędzia" w konsoli głównej serwera. Tworzymy strefę w bloku "Wyszukiwania do Przodu". "Nowa Strefa" na prawym kliku myszki.
   - Prowadzimy instalację na tzw. Strefę Podstawową.
   - Piszemy w okienku kreatora frazę: `egzamin.local` i zatwierdzamy pule.
   - Do otwartej świeżej "teczki strefowej" z prawej strony wrzucamy (Prawy klik -> Nowy host) wiersz Host A.
   - W polu Nazwa wpisujemy `www`, i przyrównujemy ten wyraz do adresu powiązanego IP karty z LAN Z Serwera: tj. `172.16.7.3`.

4. **Konfigurowanie witryny HTTP w Serwerze Web (IIS):**
   - Włączamy `inetmgr` (W menedżerze serwera).
   - Po otwarciu Menedżera IIS wybieramy stację serwera i w lewym panelu usuwamy wpisy domyślne ("Default Web Site").
   - Prawym na Witryny -> "Dodaj witrynę internetową...".
   - Formularz Wypełnień: Nazwa do ujęcia = `testowa`. Ścieżka fizyczna dokumentu (gdzie leży nasz zrzut pliku) to `C:\Web`. W polu Powiązania wybieramy IP jako `172.16.7.3` na wbudowany Port z usługą http `80`.
   - W wolne pole "Nazwa hosta" (Nagłówek) należy wpisać skonstruowany do sieci link <www.egzamin.local> (bez prefixu http).
   - Zatwierdzamy. Pośrodku pulpitu IIS używając pola włączamy opcję "Dokument Domyślny" klikając i załączając prawym kliknięciem `Dodaj...`. Piszemy tu precyzyjną nazwę pobraną z pliku domowego -> `test.html`. Położenie podnosi sie samą opcją strzałek by wskoczył na same the top pierwszeństwa listingu witryn.
   - Rozpoczynamy z poziomu Serwera przeglądarkę i pokazujemy egzaminatorowi podstronę, testując złącze <http://www.egzamin.local>.

**Weryfikacja parametrów (PowerShell):**

```powershell
Get-NetIPAddress -InterfaceAlias "LAN Z"
Resolve-DnsName -Name www.egzamin.local
Import-Module WebAdministration
Get-Website
```

---

### D2 - Linux Ubuntu Server

**Rozwiązanie (Odpowiednik):**

1. **Konfiguracja Netplan i IP na sieci `LAN Z`:**
   - W pliku yaml (`/etc/netplan/01-netcfg.yaml`):

```yaml
network:
  version: 2
  ethernets:
    ens33:
      set-name: "LAN Z"
      addresses: [172.16.7.3/24]
      routes:
        - to: default
          via: 172.16.7.1
      nameservers:
        addresses: [127.0.0.1]
    ens34:
      activation-mode: off
```

- Komenda wprowadzająca do powłoki konfigurację `sudo netplan apply`.

1. **Kopiowanie folderu Web na O/S Linuks:**
   - `sudo mkdir /Web`
   - Skopiowanie zapytania przez np. napęd zewnętrzny flash do utworzonej sekcji:
     `sudo cp /media/pendrive/DOKUMENTACJA/PROGRAMY/PLIKI/test.html /Web/`

2. **Moduły Web Apache2 i rozgłośnia Bind9:**
   - Uruchamiamy aktualizację paczek `sudo apt update && sudo apt install -y bind9 bind9utils apache2`.
   - Rekord dns. Otwarcie strefowej przestrzeni `sudo nano /etc/bind/named.conf.local`:

```text
zone "egzamin.local" {
    type master;
    file "/etc/bind/db.egzamin.local";
};
```

- Skonstruowanie domeny HostA: (Pobieramy jako wzorzec z db local):
     `sudo cp /etc/bind/db.local /etc/bind/db.egzamin.local`
     W `db.egzamin.local` wstawiamy wpis A: `www IN A 172.16.7.3`. Zrestartuj daemon pod procesem serwisu: `sudo systemctl restart bind9`.
- Moduł Apache. Tworzymy nowy plik z definicją strony wirtualnej `sudo nano /etc/apache2/sites-available/testowa.conf`:

```apache
<VirtualHost 172.16.7.3:80>
    ServerName www.egzamin.local
    DocumentRoot /Web
    DirectoryIndex test.html
</VirtualHost>
```

- Odpalamy system na nowej strukturze `sudo a2dissite 000-default.conf` następnie wprowadzamy flagę aktywacyjną `sudo a2ensite testowa.conf`. Następuje przeładowanie Apache - `sudo systemctl restart apache2`. Następnie `curl http://www.egzamin.local`.

**Weryfikacja poleceń (Bash):**

```bash
ip a show "LAN Z"
dig @127.0.0.1 www.egzamin.local
cat /etc/apache2/sites-available/testowa.conf
```

## 8. Plik Wsadowy BAT na Serwerze (D1 - Windows)

**Rozwiązanie:**
(Pracując na koncie administracyjnym w Serwerze odpalamy pole Notatnika `notepad.exe`).

- Zadaniem skryptu jest zapętlenie wykonywanego polecenia nad folderami (stworzenie w folderze `TEST` dokładnie dziesięciu nowych wpisów katalogowych).

Zawartość pliku wsadowego:

```bat
@echo off
echo tworzenie folderow dla studentow
mkdir C:\TEST
cd /d C:\TEST
for /l %%A in (1, 1, 10) do (
    mkdir STUDENT%%A
)
```

Tłumaczenie: Polecenie wpierw pozbawia plik echa wywoływanych poleceń na samej górze. Używa słowa wyświetlania w linii 2 do podania żądanego tekstu. Linie 3 i 4 służą wykonaniu korzenia systemu dysku katalogowego i wejściu do utworzonego katalogu `C:\TEST`. Następnie blok w pętli `for` opiera inkrementację licznika jako `(START: 1, KROK: 1, MAX: 10)` po zliczeniach tworząc rekursywną wykreowaną nazwę podfolderu `STUDENT1`, `STUDENT2`... itd..
Na końcu pliku zapisujemy go koniecznie zmieniając standardowy typ rozszerzenia pliku w prawym dolnym logu Zapisz Jako na wszystkie wpisy "Wszystkie pliki" i oznaczając w korzeniu partycji C:\ wprost nazwą `plik1.bat`. Po kliknięciu okienko włączy się, przeprowadzi pętle i zamknie a wynikowy `TEST` na dysku w głównym C otrzyma zbiór pustych miejsc na dokumenty dla grup docelowych.
*(Pod ekwiwalent stacji Linux (Skrypt np. bashowy) plik wykonywalny przybrałby nazwę skrypt.sh a jego postać w wierszu `echo "tworzenie folderow dla studentow" && mkdir -p /TEST && for i in {1..10}; do mkdir /TEST/STUDENT$i; done`).*
