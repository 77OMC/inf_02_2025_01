# Rozwiązanie egzaminu INF.02 (Styczeń 2025, wersja 03)

Poniżej znajduje się kompletna dokumentacja wykonania zadań egzaminacyjnych zgodnie z instrukcją inf_02_2025_01_03_sg.pdf oraz dodatkowymi wytycznymi z pliku README.

## 1. Montaż podzespołów

*Zgodnie z instrukcją polecenie wymaga modernizacji stacji roboczej poprzez montaż dodatkowego modułu pamięci RAM oraz dodatkowego dysku twardego. Zgodnie z wytyczną A (niezależnie od formy sprzętu, zazwyczaj pomija się zadania związane z fizycznym okablowaniem, tutaj mamy jednak do czynienia z komponentami bazowymi)*:

1. Należy odłączyć zasilanie stacji roboczej.
2. Zlokalizować wolny bank na płycie głównej, dopasować i wcisnąć dodatkowy moduł RAM zwracając uwagę na wcięcie pozycjonujące, aż zatrzaski zaskoczą.
3. Znaleźć miejsce na dysk w zatoce, zamocować go używając śrub i podłączyć zasilanie SATA oraz kabel sygnałowy SATA z płyty głównej.
4. Zgłosić zakończenie do Egzaminatora przed ostatecznym uruchomieniem systemu.

## 2. Konfiguracja rutera i sieci bezprzewodowej

*Zgodnie z wytyczną B:* Skonfigurowano ruter MikroTik (jako połączone urządzenie, np. hAP/hAP ac) z poziomu CLI (RouterOS).

**Polecenia konfigurujące ruter MikroTik:**

```routeros
# Reset ustawień fabrycznych (opcjonalnie)
/system reset-configuration no-defaults=yes skip-backup=yes

# 1. Konfiguracja WAN: adres IP 100.100.100.5/28 na interfejsie ether1 i brama domyślna
/ip address add address=100.100.100.5/28 interface=ether1
/ip route add distance=1 gateway=100.100.100.1

# 2. Konfiguracja serwerów DNS (8.8.8.8 i 4.4.4.4)
/ip dns set servers=8.8.8.8,4.4.4.4 allow-remote-requests=yes

# 3. Konfiguracja LAN: IP 192.168.1.1/24 na interfejsie ether2
/ip address add address=192.168.1.1/24 interface=ether2

# 4. Konfiguracja serwera DHCP z zakresem 192.168.1.50 - 192.168.1.55
/ip pool add name=dhcp_pool_lan ranges=192.168.1.50-192.168.1.55
/ip dhcp-server add address-pool=dhcp_pool_lan interface=ether2 name=dhcp_lan disabled=no
/ip dhcp-server network add address=192.168.1.0/24 dns-server=8.8.8.8 gateway=192.168.1.1

# 5. Rezerwacja DHCP (zakładając że adres MAC bezprzewodowej karty stacji roboczej to XX:XX:XX:XX:XX:XX)
/ip dhcp-server lease add address=192.168.1.51 mac-address=XX:XX:XX:XX:XX:XX

# 6. Konfiguracja Sieci Bezprzewodowej (Wireless AP) z SSID EGZAMIN_XX
# (Zakładając że interfejs WiFi nazywa się wlan1 i używamy numeru np. XX = 01)
/interface wireless security-profiles add name=profile_egzamin authentication-types=wpa2-psk mode=dynamic-keys wpa2-pre-shared-key=Qwerty_7 unicast-ciphers=aes-ccm group-ciphers=aes-ccm
/interface wireless set [ find default-name=wlan1 ] ssid=EGZAMIN_01 mode=ap-bridge security-profile=profile_egzamin frequency=auto band=2ghz-b/g/n disabled=no
# Ustawienie kanału "XX" np. 1 (W MikroTiku podaje się zwykle częstotliwość w MHz dla konkretnego kanału 2.4GHz)
/interface wireless set [ find default-name=wlan1 ] frequency=2412
```

**Jak odczytać dane z konfiguracji w MikroTik (Weryfikacja):**

- Przypisane adresy i interfejsy: `/ip address print`
- Brama (routing): `/ip route print`
- DNS: `/ip dns print`
- Pula i dzierżawy DHCP: `/ip pool print` oraz `/ip dhcp-server lease print`
- Parametry WLAN (SSID, Kanał): `/interface wireless print` oraz profil zabezpieczeń `/interface wireless security-profiles print`

## 3. Konfiguracja przełącznika

*Zgodnie z wytyczną B:* Konfiguracja zarządzalnego przełącznika (lub warstwy Bridge w RouterOS).

**Polecenia konfigurujące przełącznik MikroTik:**

```routeros
# Tworzenie interfejsu Bridge, który łączy porty przełącznika
/interface bridge add name=bridge_lan
/interface bridge port add bridge=bridge_lan interface=all

# 1. Przypisanie adresu IP do zarządzania przełącznikiem (192.168.1.2/24)
/ip address add address=192.168.1.2/24 interface=bridge_lan

# 2. Ustawienie bramy domyślnej na przełączniku (192.168.1.1)
/ip route add distance=1 gateway=192.168.1.1
```

**Jak odczytać dane z konfiguracji w MikroTik (Weryfikacja):**

- Przypisany adres zarządzania: `/ip address print`
- Sprawdzenie bramy domyślnej: `/ip route print`

## 4. Połączenie urządzeń

Należy połączyć urządzenia sieciowe skrętką (patchcordami) według schematu (lub opisu interfejsów):

1. Serwer należy podłączyć do Portu 2 Przełącznika.
2. Drukarkę sieciową należy podłączyć do kolejnego wolnego portu Przełącznika.
3. Ruter łączy się swoim interfejsem LAN z dowolnym portem Przełącznika, aby zapewnić routing do Internetu i łączność dla sieci lokalnej. (Port WAN rutera służy do podłączenia sieci nadrzędnej / dostawcy).
4. Stacja robocza (połączona przez WiFi) nie wymaga kablowego podłączenia, jeżeli zrealizowano bezprzewodowo.

## 5. Identyfikacja podzespołów w systemie Linux (Stacja Robocza)

Polecenie nakazuje wykonanie identyfikacji zamontowanych zasobów, a w Tabeli 1 znajdują się dysk twardy oraz pamięć RAM (pojemność, producent). Zgodnie z poleceniem na egzaminie w tym punkcie pracuje się w oparciu o stację roboczą z Linuxem.

**C2 - W systemie Linux (Identyfikacja z użyciem konsoli):**
Otwórz terminal (Ctrl+Alt+T) i wykonaj poniższe polecenia jako root lub używając `sudo`, aby poprawnie odczytać dane z urządzeń sprzętowych. Wyniki tych poleceń należy ująć na zrzutach ekranu, a pliki ze zrzutami umieścić w katalogu `Identyfikacja_zasobów_stacji`.

1. **Identyfikacja Dodatkowego dysku twardego:**
   Pojemność dysku i Producent:

   ```bash
   sudo lshw -class disk
   # lub
   lsblk -d -o NAME,SIZE,MODEL,VENDOR
   ```

   *Znajdź na liście odpowiednio drugi/dodatkowy dysk (np. `sdb` lub `nvme1n1`) i uzupełnij z niego Producenta i Pojemność.*

2. **Identyfikacja Dodatkowego modułu pamięci RAM:**
   Producent oraz Pojemność:

   ```bash
   sudo dmidecode -t memory | grep -E "Size|Manufacturer"
   ```

   *Spójrz na nowo podłączony bank pamięci (najczęściej ten, który nie jest "No Module Installed") i zrób zrzut ekranu odczytu.*

Utworzenie katalogu na zrzuty:

```bash
mkdir ~/Identyfikacja_zasobów_stacji
```

Zapisz w tym miejscu zrobione graficznym narzędziem systemowym (np. *gnome-screenshot*) zrzuty ekranowe.

## 6. Konfiguracja Stacji Roboczej

Zgodnie z wymaganiami przeprowadzana jest operacja nawiązania łączności przez WiFi, praca na plikach (Archiwizacja z hasłem) i zarządzanie dyskami i uprawnieniami.

### C1 - W systemie Windows

**1. Połączenie sieciowe:**

- Wejdź w Połączenia Sieciowe (`ncpa.cpl`).
- Znajdź interfejs bezprzewodowy (Wi-Fi), zmień jego nazwę na `WiFi`. Upewnij się, że ustawienia IPv4 są na "Uzyskaj adres IP automatycznie".
- Kliknij Prawym na Interfejs Przewodowy (Ethernet) i wybierz "Wyłącz".
- Z paska zadań (ikona sieci) znajdź sieć `EGZAMIN_XX` i połącz się, podając wygenerowane wcześniej w AP hasło `Qwerty_7`.

**2. Instalacja 7-Zip i utworzenie archiwum:**

- Zainstaluj z nośnika program 7-Zip.
- Odnajdź na nośniku (w folderze PLIKI) pliki `zd_1.jpg` i `zd_2.jpg`. Zaznacz je, kliknij prawym przyciskiem myszy -> 7-Zip -> Dodaj do archiwum...
- Nazwa archiwum: `kopia_zd.zip`
- W formacie archiwum zaznacz `zip`.
- W sekcji szyfrowanie, wpisz hasło: `ZAQ!@WSX`. Kliknij OK.

**3. Zarządzanie dyskami (Partycjonowanie):**

- Otwórz narzędzie Zarządzanie dyskami (`diskmgmt.msc`).
- Jeśli system zapyta o inicjalizację nowo zainstalowanego dysku, zatwierdź (np. GPT).
- Kliknij prawym na wolne miejsce (Nieprzydzielone) na dodatkowym dysku -> Nowy wolumin prosty.
- Rozmiar pierwszego woluminu określ na dokładnie 50% dostępnego wolnego miejsca. Literę dysku ustaw na **M**. Plik systemu: NTFS, Etykieta: **BACKUP1**. Sformatuj.
- Na pozostałej (drugiej) reszcie miejsca utwórz kolejny Wolumin prosty. Literę dysku przypisz na **Y**, System NTFS, Etykieta: **BACKUP2**. Sformatuj.

**4. Ochrona plików i użytkownik Praktykant:**

- Skopiuj / przenieś utworzony plik `kopia_zd.zip` z pierwotnej lokalizacji bezpośrednio na ścieżkę w dysku `M:\`.
- Otwórz konsolę `compmgmt.msc` -> Użytkownicy i grupy lokalne. Prawym na Użytkownicy -> Nowy użytkownik. Nazwa `Praktykant`, odznacz wszystkie hasła i przymusy ich zmian (Konto bez hasła jest zazwyczaj dozwolone, aczkolwiek może wymagać deaktywacji złożoności haseł w `secpol.msc` jeśli to system korporacyjny. Sprawdź ewentualnie lokalne polisy). Dodaj go do grupy Użytkownicy.
- Przejdź do Eksploratora Windows. Kliknij Prawym Przyciskiem Myszki na Dysk Lokalny **M:** -> Właściwości -> zakładka **Zabezpieczenia**.
- Kliknij Edytuj -> Dodaj... i wpisz `Praktykant`.
- Po dodaniu zaznacz na liście użytkownika `Praktykant` i w dolnej kolumnie "Odmów" (Deny) zaznacz "Pełna kontrola". Kliknij Zastosuj i OK.

### C2 - W systemie Linux Ubuntu (Wersja alternatywna stacji roboczej)

**1. Połączenie sieciowe:**
Za pomocą NetworkManagera (`nmcli`):

```bash
# Wyłączenie przewodowego interfejsu (np. eth0 / enp3s0)
nmcli connection down "Wired connection 1"
# Zmiana nazwy profilu na WiFi
nmcli connection modify "Wired connection 1" connection.id "WiFi"
# Zestawienie połączenia bezprzewodowego z terminala
nmcli device wifi connect "EGZAMIN_XX" password "Qwerty_7"
```

**2. Instalacja 7-Zip i tworzenie paczki:**

```bash
sudo apt-get install p7zip-full -y
# Przejdź do katalogu z plikami z pendrive
cd /media/Egzamin-x/DOKUMENTACJA/PLIKI
# Spakuj pliki do zip z nałożonym hasłem (-p)
7z a -p"ZAQ!@WSX" kopia_zd.zip zd_1.jpg zd_2.jpg
```

**3. Partycjonowanie:**
Używając komend (`fdisk` i `mkfs.ntfs`). Zakładamy, że nowy dysk to np. `/dev/sdb`.

```bash
sudo fdisk /dev/sdb
# -> Użyj komend 'n' (nowa partycja), przypisując +50% całości rozmiaru dysku (np. wpisując +50G jeśli dysk ma 100G)
# -> Użyj 'n' ponownie dla pozostałej reszty, potem 'w' do zapisu zmian.

# Instalacja narzędzi NTFS w Ubuntu
sudo apt-get install ntfs-3g
# Formatowanie i ustawienie etykiet
sudo mkfs.ntfs -f -L BACKUP1 /dev/sdb1
sudo mkfs.ntfs -f -L BACKUP2 /dev/sdb2

# Tworzenie punktów montowania "emulujących" litery dysków
sudo mkdir -p /mnt/M
sudo mkdir -p /mnt/Y
sudo mount /dev/sdb1 /mnt/M
sudo mount /dev/sdb2 /mnt/Y
```

**4. Zarządzanie przeniesieniem pliku i uprawnieniami (Linux ACLs):**

```bash
# Przeniesienie
sudo mv kopia_zd.zip /mnt/M/

# Tworzenie użytkownika
sudo useradd -m -s /bin/bash Praktykant
sudo passwd -d Praktykant

# Odebranie uprawnień (Linux domyślnie używa uprawnień plikowych lub list ACL)
# Nadanie odmowy dostępu przez setfacl do katalogu montowania
sudo setfacl -m u:Praktykant:- /mnt/M
```

## 7. Konfiguracja Serwera i Udostępnianie Drukarki

### D1 - Serwer z systemem Windows (Windows Server 2022)

**1. Konfiguracja interfejsów sieciowych (`ncpa.cpl`):**

- Wybierz pierwszy interfejs podłączony do przełącznika (Port 2). Zmień nazwę na `LAN1`. Właściwości -> IPv4:
  - Adres IP: `192.168.1.4`
  - Maska: `255.255.255.0`
  - Brama domyślna: `192.168.1.1`
  - Serwer DNS: `8.8.8.8`
- Wybierz drugi interfejs połączony z drukarką. Właściwości -> IPv4:
  - Adres IP: `192.168.0.101` (Zakładając że numer stanowiska X to np. 1. 100+1=101).
  - Maska: `255.255.255.0`
  - Puste pole Bramy i DNS.

**2. Konto i Usługi Druku:**

- W Zarządzaniu Komputerem (`compmgmt.msc`) stwórz nowego użytkownika w folderze "Użytkownicy" o nazwie `Drukarz`. Wpisz mu hasło `QWSAzx_12` oraz zaznacz, żeby przypisać go do grupy Użytkownicy, i odznacz żądanie zmiany hasła.
- Wejdź w Menedżer Serwera -> Zarządzanie -> Dodaj Role i Funkcje. Zaznacz **Usługi drukowania i zarządzania dokumentami** -> Serwer wydruku i zainstaluj funkcję.
- Przejdź do Panelu Sterowania -> Urządzenia i Drukarki -> **Dodaj drukarkę**.
- Wybierz "Drukarki, której szukam nie ma na liście", po czym: "Dodaj drukarkę korzystając z adresu TCP/IP".
- Typ urządzenia (Port): Wybierz `Urządzenie TCP/IP`, wpisz adres IP w polu Host: `192.168.0.200`. Odznacz ewentualne automatyczne pobieranie sterownika. Zakończ i w kreatorze ustawień portu sprawdź, że wykorzystywany jest protokół RAW (Zazwyczaj port 9100 - domyślnie TCP/IP zaznacza go poprawnie).
- Zainstaluj z dostępnych na liście w Windows jakikolwiek sterownik producenta Generic (np. Generic Text Only) albo wybierz konkretny, jeśli podany przez egzaminatora.
- Nazwa drukarki na liście, i w polu do udostępniania wpisz: `INF02`.

**3. Ustawienia Dostępności i Zabezpieczeń Drukarki:**

- W Panelu Sterowania kliknij prawym na nową drukarkę `INF02` -> Właściwości drukarki.
- Zakładka **Zaawansowane**: Zaznacz "Zawsze dostępne", jednak w polach Godzina zmień wartość dostępności na od: `06:00` do `22:00`. Ustaw suwak Priorytet (Priority) na wartość `2`.
- Zakładka **Zabezpieczenia** (Uprawnienia): Ustaw:
  - Administratorzy: Zaznacz Drukuj, Zarządzanie tymi drukarkami, Zarządzanie dokumentami. (Pełna kontrola).
  - TWÓRCA WŁAŚCICIEL: Zaznacz "Zarządzanie dokumentami".
  - Wszyscy: Usuń / Zabierz uprawnienia drukowania z listy, ewentualnie całkowicie usuń ten obiekt.
  - Dodaj do listy: Użytkownika `Drukarz`. Zaznacz mu wyłącznie pole zezwolenia "Drukuj".

**4. Wydruk strony testowej:**

- Wejdź we Właściwości tejże drukarki i w pierwszej zakładce Ogólne, kliknij przycisk "Drukuj stronę testową". (Zgłoś gotowość Egzaminatorowi).

**5. Skrypt w systemie Windows (`student.cmd`):**
Otwórz notatnik i napisz poprawną pętlę tworzącą konta przy użyciu narzędzia `net user`.
Skrypt pod nazwą `student.cmd` (Zapisany na podanym dysku USB jako typ: Wszystkie Pliki, by wymusić rozszerzenie):

```bat
@echo off
FOR /L %%i IN (1, 1, 5) DO (
    net user student_%%i /add
)
echo Proces tworzenia kont zostal zakonczony.
pause
```

Wykonaj plik ze środowiska by przetestować jego polecenia w CMD (Uruchomiony jako Administrator).

### D2 - Serwer Linux Ubuntu (Wersja alternatywna Serwera)

**1. Interfejsy sieciowe (Netplan):**
Zidentyfikuj z `ip a` karty (np. `eth0` i `eth1`). W pliku yaml ustaw dwie adresacje:

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses: [192.168.1.4/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8]
    eth1:
      addresses: [192.168.0.101/24] # Dla X = 1
```

**2. Konto oraz usługa druku (CUPS):**

- Konto w Linuksie:

  ```bash
  sudo useradd -m -s /bin/bash Drukarz
  echo 'Drukarz:QWSAzx_12' | sudo chpasswd
  ```

- Instalacja środowiska wspierającego Drukarki w Linux i interfejs GUI CUPS:

  ```bash
  sudo apt-get install cups -y
  sudo usermod -aG lpadmin administrator
  sudo systemctl restart cups
  ```

**3. Instalacja i Ograniczenia Drukarki (Przez CUPS Web/CLI):**
W konsoli za pomocą `lpadmin`:

```bash
# Instalacja drukarki wskazując socket/RAW TCP 192.168.0.200 (na sterowniku np. Generic)
sudo lpadmin -p INF02 -E -v socket://192.168.0.200:9100 -m everywhere

# Ustawienie opcji dostępności od godzin jest w Linuksie specyficzne dla CUPS (brak trywialnej metody CLI), można włączyć dostęp do web ui i zmieniać JobPolicies, albo skorzystać z reguł iptables portowych na dany serwer, ale do zabezpieczeń uprawnień użytkowników należy wpisać:
sudo lpadmin -p INF02 -u allow:Drukarz
```

**4. Skrypt basha w systemie Linux (`student.sh` z polecenia na wypadek braku Windows):**
Napisz skrypt `student.sh`:

```bash
#!/bin/bash
for i in {1..5}; do
    sudo useradd -m "student_${i}"
done
echo "Zrobione"
```

Uruchom `chmod +x student.sh` i odpal go na test `sudo ./student.sh`.

## 8. Testy Komunikacji

Polecenie wymaga pingu z poziomu Serwera do 3 węzłów.
Z poziomu konsoli CMD (Windows Server D1) lub Terminala (Linux D2) wydaj komendy:

- Test do interfejsu rutera w LAN (192.168.1.1):

  ```cmd
  ping 192.168.1.1
  ```

- Test do Przełącznika (192.168.1.2):

  ```cmd
  ping 192.168.1.2
  ```

- Test do Stacji Roboczej, która przydzielony ma IP po DHCP w sieci bezprzewodowej (zakładamy adres IP np. `192.168.1.51` - w Windows Serwer z weryfikacją na Windows Stacji Roboczej za pomocą `ipconfig` by sprawdzić adres do pingu):

  ```cmd
  ping 192.168.1.51
  ```

*(Uwaga: Jeżeli podczas testu połączenie stacja robocza nie zwraca echa w CMD, należy na PC z Windows (Stacji) wejść w Zaporę systemu Windows (Windows Defender Firewall) i wyłączyć profil np. sieci publicznej lub ustawić zaawansowaną regułę dla reguł Przychodzących - zezwalającą na komunikację "Udostępnianie plików i drukarek (żądanie echa - ruch przychodzący ICMPv4)" na Zezwalaj).*

Na Stacji Roboczej wydaj komendę aby pokazać przydzielony IP:
`ipconfig /all` w Windows albo `ip a` pod Linuksem i zawołaj Egzaminatora.

## 9. Skanowanie Nmap / Zenmap na stacji roboczej

Zadanie wymaga przeprowadzenia tzw. "Quick scan" dla całego segmentu z użyciem aplikacji graficznej Zenmap na platformie Windows.

1. Zainstaluj Zenmap za pomocą instalatora na nośniku USB w folderze z Programami.
2. Uruchom jako Administrator (by zapobiec problemom z odczytem z winpcap/npcap).
3. W polecenie Target wpisz cały zakres swojej podsieci czyli: `192.168.1.0/24` (Ponieważ stacja robocza pracuje z IP 192.168.1.51 i maską /24).
4. Z rozwijanego paska "Profile" (Profil) wybierz z listy na samym początku lub pozycję nazwaną **Quick scan**.
5. Kliknij przycisk **Scan**. (Wygeneruje to komendę nmap -T4 -F 192.168.1.0/24 i wyświetli dostępne stacje).
6. Po przeanalizowaniu całości, wejdź w oknie Zenmapa na zakładkę graficzną nazwaną **"Topology"** (Topologia).
7. Widok w tej zakładce pokazuje graf sieciowy i węzły, które "znalazł" Nmap.
8. Użyj narzędzia Windows wycinanie (`Win+Shift+S`) i wykonaj zrzut.
9. Zapisz i nadaj mu nazwę `zenmap.jpg`, wrzucając go do pamięci Egzamin-x.
