# Rozwiązanie egzaminu INF.02 (Styczeń 2025, wersja 01)

Poniżej znajduje się kompletna dokumentacja wykonania zadań egzaminacyjnych zgodnie z instrukcją inf_02_2025_01_01_sg.pdf oraz dodatkowymi wytycznymi z pliku README.

## 1. Montaż okablowania sieciowego

*Zgodnie z wytyczną A:* Pomijamy zadanie dotyczące fizycznego montażu okablowania sieciowego oraz jego testowania w obecności egzaminatora.

## 2. Konfiguracja rutera

*Zgodnie z wytyczną B:* Przeprowadzono konfigurację dla urządzeń sieciowych MikroTik. Do konfiguracji można użyć graficznego narzędzia Winbox lub linii komend (CLI). Poniżej zestawiono polecenia CLI dla RouterOS.

**Polecenia konfigurujące ruter MikroTik:**

```routeros
# Opcjonalnie: reset do ustawień fabrycznych bez domyślnej konfiguracji
/system reset-configuration no-defaults=yes skip-backup=yes

# Przypisanie adresów IP do odpowiednich interfejsów (przyjmując ether1 jako WAN i ether2 jako LAN)
/ip address add address=90.0.0.3/28 interface=ether1
/ip address add address=10.10.0.1/24 interface=ether2

# Ustawienie bramy domyślnej dla rutera (wyjście w świat)
/ip route add distance=1 gateway=90.0.0.2

# Konfiguracja serwerów DNS
/ip dns set servers=4.4.4.4,4.4.5.5 allow-remote-requests=yes

# Konfiguracja serwera DHCP dla sieci LAN
/ip pool add name=dhcp_pool_lan ranges=10.10.0.2-10.10.0.20
/ip dhcp-server add address-pool=dhcp_pool_lan interface=ether2 name=dhcp_lan disabled=no
/ip dhcp-server network add address=10.10.0.0/24 dns-server=10.10.0.20 gateway=10.10.0.1

# Rezerwacja adresu IP dla serwera w puli DHCP. Zamiast XX:XX:XX:XX:XX:XX należy podać adres MAC interfejsu WAN serwera:
/ip dhcp-server lease add address=10.10.0.20 mac-address=XX:XX:XX:XX:XX:XX
```

**Jak odczytać dane z konfiguracji w MikroTik (Weryfikacja):**
Aby egzaminator mógł w krótkim czasie zweryfikować konfigurację z użyciem terminala, należy wydać polecenia:

- Adresy IP interfejsów: `/ip address print`
- Brama domyślna rutera: `/ip route print`
- Adresy DNS: `/ip dns print`
- Zakres i ustawienia sieci DHCP: `/ip pool print` oraz `/ip dhcp-server network print`
- Zarezerwowany adres dla serwera: `/ip dhcp-server lease print`

## 3. Konfiguracja przełącznika

*Zgodnie z wytyczną B:* Konfiguracja zarządzalnego przełącznika MikroTik z poziomu CLI (RouterOS dla przełączników lub warstwa Switch/Bridge dla routerów).

**Polecenia konfigurujące przełącznik MikroTik:**

```routeros
# Tworzenie interfejsu Bridge, który łączy wszystkie porty przełącznika (jeśli brakuje)
/interface bridge add name=bridge_lan
/interface bridge port add bridge=bridge_lan interface=all

# Przypisanie adresu IP do zarządzania przełącznikiem
/ip address add address=192.168.1.2/25 interface=bridge_lan

# Ustawienie bramy domyślnej na przełączniku, by umożliwić jego administrację z innych sieci
/ip route add distance=1 gateway=192.168.1.1
```

**Jak odczytać dane z konfiguracji w MikroTik (Weryfikacja):**

- Przypisany adres zarządzania: `/ip address print`
- Sprawdzenie bramy domyślnej: `/ip route print`

## 4. Połączenie urządzeń

Należy fizycznie połączyć urządzenia przy pomocy kabli krosowych dostępnych na stanowisku:

1. Kabel od gniazda dostawcy (lub imitatora) do portu WAN w ruterze (ether1).
2. Kabel z portu LAN w ruterze (ether2) do wyznaczonego portu na Serwerze (WAN/Interfejs uzyskujący IP z DHCP).
3. Serwer łączy się również oddzielnym kablem (z interfejsu LAN1) z odpowiednim portem przełącznika.
4. Stacja robocza (interfejs LAN2) łączona jest kablem sieciowym z wyznaczonym portem przełącznika.
Po wykonaniu fizycznych podłączeń sieciowych, wszystkie urządzenia należy podpiąć do zasilania i uruchomić.

## 5. Identyfikacja podzespołów komputera

Należy zidentyfikować: Nazwę procesora, Liczbę rdzeni, Taktowanie procesora, Producenta i model karty graficznej. Wyniki te mają być uzupełnione w Tabeli 1 na karcie egzaminacyjnej, a potwierdzenie należy zapisać w postaci zrzutów ekranu w folderze `Identyfikacja` na dysku USB `Egzamin-x`.

**C1 - W systemie Windows:**
Najszybszą i najpewniejszą drogą do weryfikacji i zebrania zrzutów z wszystkich żądanych informacji jest otwarcie **Menedżera zadań** (`taskmgr`):

1. Przejdź na zakładkę **Wydajność** (Performance).
2. Wybierz zakładkę **CPU**: W prawym górnym rogu znajduje się model procesora. W dolnej części okna znajdziesz liczbę rdzeni ("Rdzenie") oraz domyślne taktowanie ("Szybkość bazowa").
3. Wybierz zakładkę **GPU**: W prawym górnym rogu znajduje się pełna nazwa i producent karty graficznej.
4. Wykonaj odpowiednie zrzuty ekranu (Win+Shift+S) i zapisz na pendrive.

Alternatywnie przy użyciu PowerShell (do szybkiego odczytu textowego):

```powershell
Get-WmiObject Win32_Processor | Select-Object Name, NumberOfCores, MaxClockSpeed
Get-WmiObject Win32_VideoController | Select-Object Name
```

**C2 - W systemie Linux Ubuntu:**
Aby wykonać identyfikację sprzętową, w terminalu wpisz poniższe polecenia i wykonaj zrzut ekranu po ich pomyślnym wykonaniu.

1. Parametry Procesora:

   ```bash
   lscpu | grep -E "Model name|CPU\(s\)|CPU max MHz|Thread"
   # Można też użyć pełnego wydruku, by znaleźć wymagane parametry.
   ```

2. Parametry Karty graficznej (wymaga użycia `lspci` do wyświetlenia sterowników VGA):

   ```bash
   lspci -nn | grep -i vga
   # Można również wywołać 'lshw -C display' jako root dla obszerniejszego opisu:
   sudo lshw -C display
   ```

## 6. Konfiguracja stacji roboczej

Konfiguracja obejmuje fizyczny interfejs sieciowy "LAN 2" (łączący z przełącznikiem) oraz wirtualny system Linux.

### C1 - Konfiguracja na systemie Windows (jako fizyczny Host)

1. Otwórz `ncpa.cpl` (Połączenia sieciowe).
2. Znajdź interfejs sieciowy, zmień jego nazwę (Zmień nazwę tego połączenia) na `LAN 2`.
3. Prawy przycisk myszy na `LAN 2` -> Właściwości -> Protokół internetowy w wersji 4 (TCP/IPv4) -> Właściwości.
4. Ustaw ręczne adresy IP:
   - Adres IP: `192.168.1.3`
   - Maska podsieci: `255.255.255.128` *(ważne, bo sieć jest w notacji /25)*
   - Brama domyślna: `192.168.1.1`
   - Preferowany serwer DNS: `192.168.1.1`

**Weryfikacja w systemie Windows:**
Otwórz wiersz polecenia (`cmd`) i wydaj komendę:

```cmd
ipconfig /all
```

### C2 - Konfiguracja na systemie Linux Ubuntu (jako fizyczny Host)

Używając mechanizmu Netplan (domyślnego w nowych Ubuntu) lub NetworkManager. Zwykle systemy Desktop korzystają z Network Managera.
**Z użyciem narzędzia `nmcli` w terminalu:**

```bash
# Odnalezienie nazwy domyślnego połączenia
nmcli connection show
# (Załóżmy, że nazywa się "Wired connection 1" dla interfejsu eth0)

# Zmiana nazwy profilu
nmcli connection modify "Wired connection 1" connection.id "LAN 2"

# Modyfikacja ustawień na tryb ręczny z uwzględnieniem CIDR /25:
nmcli connection modify "LAN 2" ipv4.addresses 192.168.1.3/25 ipv4.gateway 192.168.1.1 ipv4.dns 192.168.1.1 ipv4.method manual

# Zastosowanie zmian
nmcli connection up "LAN 2"
```

**Weryfikacja w systemie Linux:**
Wydaj w terminalu polecenia:

```bash
ip a
ip route show
resolvectl status # aby zweryfikować adresy serwerów DNS
```

### Konfiguracja Maszyny Wirtualnej Linux (w VirtualBox)

1. W aplikacji Oracle VirtualBox zainstaluj/stwórz nową maszynę wirtualną typu Linux, podpinając pobrany obraz ISO.
2. W Opcjach maszyny wirtualnej, przejdź do zakładki **Sieć** -> Karta 1.
3. Wybierz opcję "Podłączona do: **Bridged Adapter**" (Karta mostkowana). Jako nazwę interfejsu, do którego się "mostkuje", wybierz fizyczną kartę hosta odpowiedzialną za `LAN 2`.
4. Uruchom maszynę wirtualną, otwórz terminal i ręcznie przypisz konfigurację. Na przykład przez edycję profilu dla interfejsu maszyny nazwanego na potrzeby polecenia `LAN 3`:
   - Nazwa interfejsu (w systemie hosta to po prostu zmiana ustawień logicznych dla połączenia lub profilu na nazwę `LAN 3`).
   - Adres IP: `192.168.1.4/25`
   - Brama: `192.168.1.1`
   - DNS: `192.168.1.1`

## 7. Konfiguracja serwera

Serwer ma pełnić m.in. rolę DNS oraz hostować lokalną stronę Web dla testów stacji roboczej.

### D1 - System Windows Server 2022

**1. Interfejsy sieciowe:**

1. W "Połączeniach sieciowych" (ncpa.cpl) zmień nazwę interfejsu połączonego z ruterem na `WAN`, a podłączonego z przełącznikiem na `LAN 1`.
2. Interfejs `WAN`: We właściwościach IPv4 zaznacz **"Uzyskaj adres IP automatycznie"** oraz **"Uzyskaj adres serwera DNS automatycznie"**.
   - Odświeżenie adresu: w CMD wpisz `ipconfig /release` i zaraz po tym `ipconfig /renew`. Serwer powinien otrzymać adres rezerwowany `10.10.0.20`.
3. Interfejs `LAN 1`: We właściwościach IPv4 ustaw ręcznie:
   - Adres IP: `192.168.1.1`
   - Maska: `255.255.255.128` *( /25 )*
   - Brama domyślna: (puste)
   - Serwer DNS: `192.168.1.1`

**2. Instalacja i konfiguracja Usług (DNS i IIS):**

1. Otwórz "Menedżer serwera" -> "Zarządzaj" -> "Dodaj role i funkcje". Zainstaluj **Serwer DNS** oraz **Serwer sieci Web (IIS)**.
2. Po instalacji otwórz Menedżer DNS (dnsmgmt.msc).
   - Wybierz "Strefy wyszukiwania do przodu", kliknij prawym -> **"Nowa strefa..."**.
   - Utwórz Strefę podstawową o nazwie: `egzamin.local`.
   - Wewnątrz nowej strefy dodaj nowy **rekord hosta (A)** o nazwie `www`, któremu przypisz adres IP interfejsu `LAN 1`, czyli `192.168.1.1`.
3. Otwórz katalog dysku `C:\` i utwórz tam folder `www`.
4. Zainstaluj program 7-Zip z dostępnego instalatora z nośnika (Folder DOKUMENTACJA/PROGRAMY).
5. Za pomocą 7-Zip wypakuj pliki ze `strona_testowa.7z` i przenieś do katalogu `C:\www`. Do tego samego katalogu skopiuj plik `zegarek.jpg`.
6. Otwórz "Menedżer internetowych usług informacyjnych (IIS)" (`inetmgr`).
   - W drzewku po lewej, rozwiń do pozycji "Witryny". Zaznacz "Default Web Site" i ją **zatrzymaj** lub usuń.
   - Prawy przycisk myszy na "Witryny" -> **"Dodaj witrynę sieci Web..."**.
   - Nazwa witryny: `strona_testowa`
   - Ścieżka fizyczna: `C:\www`
   - Typ: HTTP
   - Adres IP: `192.168.1.1` (lub Wszystkie nieprzypisane)
   - Port: `80`
   - Nazwa hosta: `www.egzamin.local`
   - Kliknij "OK". Po jej dodaniu w głównym oknie Witryny `strona_testowa` kliknij "Dokument domyślny". Jeśli `strona_testowa.html` nie widnieje na liście, dodaj go na samej górze.

**Weryfikacja szybkiego podglądu powiązań Windows:**

- `ipconfig /all` - wyświetla szczegółową konfigurację obu kart sieciowych.

### D2 - System Linux Ubuntu Server

**1. Interfejsy sieciowe (używamy Netplan):**
Zidentyfikuj odpowiednie interfejsy (np. eth0 = WAN, eth1 = LAN 1) za pomocą polecenia `ip a`. Następnie otwórz do edycji plik konfiguracyjny (np. `/etc/netplan/00-installer-config.yaml`):

```yaml
network:
  version: 2
  ethernets:
    eth0: # Interfejs WAN podłączony do routera
      dhcp4: true
    eth1: # Interfejs LAN 1 podłączony do switcha
      addresses:
        - 192.168.1.1/25
      nameservers:
        addresses: [192.168.1.1]
```

Zapisz zmiany i zaktualizuj system poleceniem `sudo netplan apply`.
Aby wymusić odświeżenie IP na interfejsie WAN użyj komend: `sudo dhclient -r eth0` (zwolnienie) oraz `sudo dhclient eth0` (pobranie).

**2. Instalacja i konfiguracja Usług (BIND9 jako DNS, Apache2 jako IIS):**

1. Instalacja oprogramowania oraz pakietu 7-zip:

   ```bash
   sudo apt-get update
   sudo apt-get install bind9 bind9utils bind9-doc apache2 p7zip-full -y
   ```

2. **Konfiguracja DNS (`egzamin.local`):**
   Wyedytuj plik `/etc/bind/named.conf.local` i dodaj wpis strefy:

   ```text
   zone "egzamin.local" {
       type master;
       file "/etc/bind/db.egzamin.local";
   };
   ```

   Utwórz plik strefy skopiowany z szablonu, w którym ustawisz rekord dla serwera www:

   ```bash
   sudo cp /etc/bind/db.local /etc/bind/db.egzamin.local
   sudo nano /etc/bind/db.egzamin.local
   ```

   Wewnątrz pliku zmień zawartość, aby wskazać rekord A:

   ```text
   ; (Pozostaw początkowe bloki SOA i zmodyfikuj resztę)
   @       IN      NS      localhost.
   @       IN      A       192.168.1.1
   www     IN      A       192.168.1.1
   ```

   Uruchom ponownie BIND9: `sudo systemctl restart bind9`.
3. **Konfiguracja usługi Web i wypakowanie plików:**
   Utwórz folder i wypakuj archiwum:

   ```bash
   sudo mkdir /www
   # skopiuj strona_testowa.7z oraz zegarek.jpg na serwer
   sudo 7z x strona_testowa.7z -o/www/
   sudo cp zegarek.jpg /www/
   ```

   Skonfiguruj Wirtualny Host w Apache. Stwórz plik `/etc/apache2/sites-available/strona_testowa.conf`:

   ```apache
   <VirtualHost 192.168.1.1:80>
       ServerName www.egzamin.local
       DocumentRoot /www
       DirectoryIndex strona_testowa.html
       <Directory /www>
           Require all granted
       </Directory>
   </VirtualHost>
   ```

   Wyłącz domyślną stronę i włącz swoją, na koniec restart serwera apache2:

   ```bash
   sudo a2dissite 000-default.conf
   sudo a2ensite strona_testowa.conf
   sudo systemctl reload apache2
   ```

**Weryfikacja ustawień na Linux:**

- Obejrzenie IP: `ip a` lub `ip addr show`
- Obejrzenie procesów słuchających na portach (DNS 53, HTTP 80): `sudo ss -tuln | grep -E ':53|:80'`

## 8. Testy komunikacji na serwerze

Na serwerze należy przeprowadzić testy komunikacyjne w obrębie podsieci oraz pokazać automatycznie oddelegowany adres IP na zewnątrz. Testy należy wykonać w terminalu. Zgłaszając gotowość do egzaminatora, wydaj polecenia ponownie.

**D1/C1 System Windows:**

- Wyświetlenie przyznanego adresu IP na interfejsie WAN (zarezerwowanego w ruterze jako 10.10.0.20): `ipconfig /all` - wyszukaj sekcję "WAN".
- Test do stacji roboczej: `ping 192.168.1.3`
- Test do rutera (strona LAN rutera to 10.10.0.1): `ping 10.10.0.1`
*(Zauważ: Jeżeli komunikacja pomiędzy podsiecią `192.168.1.0/25` a `10.10.0.0/24` nie wychodzi standardowym routingiem z poziomu Windows, serwer może odpowiadać "Uplynał limit czasu żądania". W przypadku blokady na stacji roboczej można chwilowo wyłączyć jej systemową Zaporę (Zapora Windows Defender -> Wyłącz) dla celów ping, lub dopuścić ICMP Echo Request we właściwościach zaawansowanych Zapory na urządzeniu poddawanemu pingowaniu).*

**D2/C2 System Linux Ubuntu:**

- Wyświetlenie IP z interfejsu WAN: `ip a show eth0`
- Test pingu do stacji roboczej: `ping -c 4 192.168.1.3`
- Test pingu do rutera na LAN: `ping -c 4 10.10.0.1`
(Linuksy domyślnie zazwyczaj przepuszczają przez Iptables/UFW pakiety ICMP pingu, jeżeli zapora nie jest dodatkowo rekonfigurowana przez polecenia egzaminacyjne).

## 9. Testy działania maszyny wirtualnej (Linux na stacji roboczej)

W gotowej powłoce (tryb graficzny/przeglądarka) na maszynie wirtualnej:

1. Otwórz terminal i użyj komendy:

   ```bash
   ip addr show
   ```

   Wynik wyświetli podłączone IP interfejsu do sieci (zgodnie z zadaniem `192.168.1.4/25`). Przygotuj to na ekranie do oceny dla Egzaminatora.
2. Otwórz przeglądarkę internetową, np. Firefox.
3. W pasku adresu wpisz pełny URL powiązany z serwerem i domeną, ustawioną po stronie serwera DNS i www IIS/Apache:

   ```text
   http://www.egzamin.local
   ```

4. Pod adresem powinna wyświetlić się strona internetowa zawierająca dodany plik graficzny `zegarek.jpg` i treść `strona_testowa.html`. Przygotuj gotową stronę w oknie i po podniesieniu ręki zgłoś gotowość do Egzaminatora.

## 10. Kosztorys

Zadanie wymagało utworzenia arkusza kalkulacyjnego (`kosztorys.xlsx` / `.ods`) na dysku USB z wypełnioną tabelą nr 2 ze stosownymi wyliczeniami, automatycznymi formułami i formatowaniem komórek jako Waluta PLN.

**Analiza i wyliczenia do kosztorysu na podstawie nakładów:**

- Zamontowanych w szafie zostało 100 kabli do panelu krosowego. Ilość montażów to `100 szt.` (poz. 7: "Montaż kabla U/UTP do panelu krosowego").
- Drugi koniec (100 kabli) ląduje w gniazdach. Każde gniazdo w zleceniu to element 2x RJ45. Aby podłączyć 100 kabli, potrzebujemy dokładnie 50 gniazd 2x RJ45. Zatem wiersz nr 4 "Montaż gniazda naściennego 2 x RJ45" posiada wskaźnik sztuk `= 50 szt.`.
- Na każdym drugim końcu kabla trzeba zarobić wkład do niego - Keystone. Będzie to więc 100 modułów na 100 kabli. Zatem wiersz nr 2 "Montaż modułu Keystone do kabla U/UTP" to `= 100 szt.`.
- Wkładamy moduły do obudowy (w jedno gniazdo 2 moduły). Wiersz nr 3 mówi o "Montaż modułu Keystone w obudowie gniazda" z normą per sztuka modułu, co daje znowu wartość `= 100 szt.`.
- Stawka za 1 roboczogodzinę: `50 zł`. Wstawiamy ją wszędzie w odpowiedniej kolumnie.
- Łączna liczba roboczogodzin i wartość liczymy jako proste iloczyny wartości we wcześniejszych komórkach.

**Symulowany i przewidywany widok dokumentu MS Excel/LibreOffice Calc po rozwiązaniu:**

| Lp. | Nazwa czynności | Liczba sztuk | Liczba roboczogodzin na 1 sztukę | Łączna liczba roboczogodzin | Stawka za 1 roboczogodzinę | Wartość robocizny |
|---|---|---|---|---|---|---|
| **1.** | Montaż wtyku 8P8C na kablu U/UTP | 0 | 0,10 | `=C2*D2` | 50,00 zł | `=E2*F2` |
| **2.** | Montaż modułu Keystone do kabla U/UTP | 100 | 0,10 | `=C3*D3` | 50,00 zł | `=E3*F3` |
| **3.** | Montaż modułu Keystone w obudowie gniazda | 100 | 0,05 | `=C4*D4` | 50,00 zł | `=E4*F4` |
| **4.** | Montaż gniazda naściennego 2 x RJ45 | 50 | 0,10 | `=C5*D5` | 50,00 zł | `=E5*F5` |
| **5.** | Montaż gniazda podłogowego 2 x RJ45 | 0 | 0,30 | `=C6*D6` | 50,00 zł | `=E6*F6` |
| **6.** | Montaż złącza światłowodowego typu SC | 0 | 0,20 | `=C7*D7` | 50,00 zł | `=E7*F7` |
| **7.** | Montaż kabla U/UTP do panelu krosowego | 100 | 0,10 | `=C8*D8` | 50,00 zł | `=E8*F8` |
| | **RAZEM** | | | | | `=SUM(G2:G8)` |

*(Ostatnia komórka "Razem" korzysta z wbudowanej funkcji podsumowującej wiersze od 1 do 7, tak jak to określa regulamin: "z zastosowaniem wbudowanej funkcji sumującej" `SUM() / SUMA()`).*

Zsumowane przewidywane prawidłowe wyniki (niewidoczne wzory dla sprawdzającego):

- Montaż modułu Keystone (poz. 2): 10 h × 50 zł = 500,00 zł
- Montaż w obudowie (poz. 3): 5 h × 50 zł = 250,00 zł
- Montaż gniazda 2xRJ45 (poz. 4): 5 h × 50 zł = 250,00 zł
- Montaż panela (poz. 7): 10 h × 50 zł = 500,00 zł
- **Suma RAZEM z kosztorysu wynosi: 1500,00 zł.**
Puste komórki i zera zwracają kwotę `0,00 zł`.

Na sam koniec w edytorze zaznacz wszystkie komórki w kolumnie *Stawka...* oraz *Wartość...* następnie zastosuj ich formatowanie jako *Walutowe*, by obok widniały symbole 'zł' lub 'PLN'. Zapisz jako plik pod wyznaczoną nazwą na pamięci USB.
