# Rozwiązanie egzaminu INF.02 (Styczeń 2025, wersja 02)

Poniżej znajduje się kompletna dokumentacja wykonania zadań egzaminacyjnych zgodnie z instrukcją inf_02_2025_01_02_sg.pdf oraz dodatkowymi wytycznymi z pliku README.

## 1. Montaż okablowania sieciowego

*Zgodnie z wytyczną A:* Pomijamy zadanie dotyczące fizycznego montażu okablowania sieciowego oraz jego testowania za pomocą testera w obecności egzaminatora.

## 2. Konfiguracja rutera

*Zgodnie z wytyczną B:* Skonfigurowano ruter MikroTik przy użyciu interfejsu linii poleceń (CLI). Zakładamy, że port WAN to `ether1`, a port LAN to `ether2`.

**Polecenia konfigurujące ruter MikroTik:**

```routeros
# Ewentualny reset bez ustawień fabrycznych:
/system reset-configuration no-defaults=yes skip-backup=yes

# 1. Konfiguracja interfejsu WAN: IP 55.55.55.55/28
/ip address add address=55.55.55.55/28 interface=ether1

# 2. Konfiguracja bramy domyślnej dla interfejsu WAN: 55.55.55.50
/ip route add distance=1 gateway=55.55.55.50

# 3. Konfiguracja serwerów DNS (4.4.4.4 oraz 7.7.7.7)
/ip dns set servers=4.4.4.4,7.7.7.7 allow-remote-requests=yes

# 4. Konfiguracja interfejsu LAN: 192.168.0.1/24
/ip address add address=192.168.0.1/24 interface=ether2

# 5. Serwer DHCP wyłączony - w MikroTiku bez domyślnej konfiguracji nie ma go dopóki się go nie stworzy, jednak by mieć pewność można usunąć lub wyłączyć ewentualny domyślny wpis.
/ip dhcp-server disable [find]
```

**Jak odczytać dane z konfiguracji w MikroTik (Weryfikacja):**
Aby szybko zweryfikować konfigurację w terminalu:

- Zobaczenie adresów IP: `/ip address print`
- Zobaczenie ustawień bramy (routingu): `/ip route print`
- Zobaczenie ustawień DNS: `/ip dns print`
- Upewnienie się o wyłączonym DHCP: `/ip dhcp-server print` (wynik powinien być pusty lub flagi wyłączone).

## 3. Konfiguracja przełącznika

*Zgodnie z wytyczną B:* Konfiguracja zarządzalnego przełącznika (lub warstwy Bridge w RouterOS).

**Polecenia konfigurujące przełącznik MikroTik:**

```routeros
# Upewniamy się, że istnieje bridge zbierający porty (np. bridge1)
/interface bridge add name=bridge1
/interface bridge port add bridge=bridge1 interface=all

# 1. Przypisanie adresu IP do zarządzania (192.168.0.2/24)
/ip address add address=192.168.0.2/24 interface=bridge1

# 2. Ustawienie bramy domyślnej przełącznika (192.168.0.1)
/ip route add distance=1 gateway=192.168.0.1
```

**Jak odczytać dane z konfiguracji w MikroTik (Weryfikacja):**

- Przypisany adres zarządzania: `/ip address print`
- Sprawdzenie bramy domyślnej: `/ip route print`

## 4. Połączenie urządzeń

- Serwer należy podłączyć kablem z portem nr 3 przełącznika (LAN).
- Stację roboczą należy podłączyć kablem z portem nr 4 przełącznika (LAN).
- Ruter łączy się portem LAN z przełącznikiem, a swoim portem WAN w świat/do modemu. Urządzenia podłączyć do prądu i włączyć.

## 5. Identyfikacja Pamięci RAM (Stacja Robocza)

Należy sprawdzić pojemność, standard (typ), częstotliwość oraz numer seryjny/produktu pamięci RAM. Odczyty mają zostać udokumentowane zrzutami z ekranu do folderu `Identyfikacja_stacji_roboczej` i zapisane do Tabeli 1.

**C1 - W systemie Windows:**
Zgodnie z poleceniem z arkusza użyto by CPU-Z.

1. Zainstaluj CPU-Z i uruchom aplikację.
2. Przejdź na zakładkę **Memory** by sprawdzić pojemność całkowitą oraz standard (np. DDR4). Oraz bieżące taktowanie.
3. Przejdź na zakładkę **SPD** by odczytać parametry per dany moduł RAM, tutaj znajduje się Maksymalna Częstotliwość oraz **Part Number** (Nr produktu) i **Serial Number**.
4. Wykonaj zrzuty do folderu.

Alternatywnie szybka weryfikacja w wierszu poleceń (cmd):

```cmd
wmic memorychip get capacity, memorytype, speed, partnumber, serialnumber
```

**C2 - W systemie Linux Ubuntu:**
Aby uzyskać dokładne informacje o sprzętowej identyfikacji kości RAM (wymaga podniesionych uprawnień `sudo`):

```bash
sudo dmidecode -t memory | grep -E "Size|Type:|Speed:|Serial Number:|Part Number:"
```

(Wynik pokaże parametry dla wszystkich slotów; puste banki zwrócą "No Module Installed"). Z tego ekranu należy zrobić zrzut.

## 6. Identyfikacja Pamięci RAM (Serwer - Przekierowanie do plików)

**D2 / D1 - Identyfikacja RAM do pliku (Zgodnie z instrukcją pod serwer Linux):**
Utworzenie folderu i logowanie danych w systemie **Linux Server (D2)**:

```bash
mkdir ~/Identyfikacja_serwera
cd ~/Identyfikacja_serwera
sudo dmidecode -t memory > ram_info.txt
# Lub bardziej selektywnie, by zebrać tylko pożądane:
sudo dmidecode -t memory | grep -E "Size|Type:|Speed:|Serial Number:|Part Number:" > parametry_ram.txt
```

**D1 - Identyfikacja RAM do pliku (Windows Server):**
Gdyby polecenie było pod Windows Server, należy użyć PowerShell:

```powershell
New-Item -ItemType Directory -Path "$env:USERPROFILE\Identyfikacja_serwera"
Get-CimInstance Win32_PhysicalMemory | Select-Object Capacity, MemoryType, Speed, SerialNumber, PartNumber | Out-File -FilePath "$env:USERPROFILE\Identyfikacja_serwera\ram_info.txt"
```

## 7. Konfiguracja Serwera (Interfejs i SSH)

**D2 - W systemie Linux Ubuntu Server:**

1. Konfiguracja adresacji (Netplan dla portu np. `eth0` / `enp3s0`):
   Otwórz z `sudo nano /etc/netplan/00-installer-config.yaml`

   ```yaml
   network:
     version: 2
     ethernets:
       eth0:
         addresses: [192.168.0.4/24]
         routes:
           - to: default
             via: 192.168.0.1
         nameservers:
           addresses: [4.4.4.4]
   ```

   Zapisz i zastosuj z użyciem `sudo netplan apply`.
   *Weryfikacja adresacji: `ip a` i `ip route`*
2. Automatyczne uruchomienie SSH przy starcie i jego uruchomienie:

   ```bash
   sudo systemctl enable ssh
   sudo systemctl start ssh
   # weryfikacja statusu:
   sudo systemctl status ssh
   ```

**D1 - W systemie Windows Server 2022 (Odpowiednik):**

1. Adresacja: W `ncpa.cpl` (Połączenia sieciowe), wejdź we właściwości połączenia, IPv4 i wpisz:
   - IP: `192.168.0.4`
   - Maska: `255.255.255.0`
   - Brama: `192.168.0.1`
   - DNS: `4.4.4.4`
2. SSH OpenSSH serwer: Przez panel PowerShell zainstaluj usługę, a potem w `services.msc` znajdź "OpenSSH SSH Server", wejdź we właściwości, zmień Typ Uruchomienia na "Automatycznie" (Automatic) i kliknij Uruchom.

## 8. Konfiguracja Stacji Roboczej

Skonfigurowanie sieci oraz dodanie zabezpieczeń (lokalne zarządzanie użytkownikami) i nawiązanie sesji przez Putty.

### C1 - W systemie Windows (Zgodnie ze standardowym profilem)

1. **Adresacja Sieci (ncpa.cpl):**
   - Zmień nazwę na "LAN2". Właściwości IPv4:
   - IP: `192.168.0.3`
   - Maska: `255.255.255.0`
   - Brama: `192.168.0.1`
   - DNS: `4.4.4.4`
   *Weryfikacja: `ipconfig /all`*
2. **Konta Użytkowników i Grupy (Zarządzanie komputerem `compmgmt.msc`):**
   - Rozwiń Narzędzia systemowe -> Użytkownicy i grupy lokalne.
   - Prawy klik na "Użytkownicy" -> Nowy użytkownik: `serwisant`. Odznacz "Użytkownik musi zmienić hasło", zaznacz "Użytkownik nie może zmienić hasła".
   - Nowy użytkownik: `kierownik`. Wpisz hasło `XSW@#EDA`. Odznacz zmianę hasła.
   - Prawy klik na "Grupy" -> Nowa grupa: nazwij ją `serwis`. Dodaj do niej `serwisant` i `kierownik`.
3. **Zasady (Polityki) blokady konta (`secpol.msc`):**
   - Wejdź w Ustawienia zabezpieczeń -> Zasady konta -> Zasady blokady konta.
   - Ustaw "Próg blokady konta" (Account lockout threshold) na **4 nieudane próby**. Zastosuj domyślne limity czasowe.
4. **Logowanie z użyciem PuTTY (Dokumentacja):**
   - Włącz pobrany program `putty.exe`.
   - Host Name (IP): wpisz IP serwera Linux: `192.168.0.4`. Port `22`. Zaznacz Connection type `SSH`.
   - ZRZUT 1 (`putty1.jpg`).
   - Kliknij "Open". Zaakceptuj klucz serwera. Zaloguj się podając użytkownika `administrator` i jego hasło `ZAQ!2wsx` z instrukcji.
   - Po powitaniu w powłoce linuksa serwera zrób ZRZUT 2 (`putty2.jpg`).

### C2 - W systemie Linux Ubuntu Desktop (Wersja alternatywna stacji roboczej)

1. **Adresacja Sieci (`nmcli`):**

   ```bash
   nmcli connection modify "Wired connection 1" connection.id "LAN2"
   nmcli connection modify "LAN2" ipv4.addresses 192.168.0.3/24 ipv4.gateway 192.168.0.1 ipv4.dns 4.4.4.4 ipv4.method manual
   nmcli connection up "LAN2"
   ```

2. **Użytkownicy i polityki (`useradd` i `pam_tally2/faillock`):**

   ```bash
   # Utworzenie kierownika z hasłem i brakiem expiracji
   sudo useradd -m -s /bin/bash kierownik
   echo 'kierownik:XSW@#EDA' | sudo chpasswd

   # Utworzenie serwisanta (bez hasła) i blokada zmiany hasła
   sudo useradd -m -s /bin/bash serwisant
   sudo passwd -d serwisant # Usuwa hasło pozwalając na logowanie "bez"
   sudo chage -m 99999 serwisant # Uniemożliwia zmianę hasła

   # Utworzenie grupy i dodanie kont
   sudo groupadd serwis
   sudo usermod -aG serwis kierownik
   sudo usermod -aG serwis serwisant

   # Ustawienie blokady logowania po 4 próbach (konfiguracja pam_faillock - pliki autoryzacyjne w Ubuntu 20.04+ common-auth / faillock.conf)
   # W pliku /etc/security/faillock.conf (lub przez pam-auth-update) zmiana dyrektywy deny = 4
   ```

3. **Połączenie PuTTY:** W systemie Linux instalacja PuTTY z pakietów (lub po prostu przez komendę `ssh administrator@192.168.0.4`) i w ten sam sposób utworzenie okna do zrzutów.

## 9. Testy komunikacji na serwerze

Egzaminator wymaga wykonania pingów z wiersza poleceń Serwera do 3 wyznaczonych adresów.
Oto sekwencja, którą należy wykonać (Zgłosić Egzaminatorowi i po jego wezwaniu ponowić testy na ekranie).

**D2/D1 Testy na serwerze (Ubuntu / CMD w Windows Server):**

- Test do Stacji Roboczej: `ping -c 4 192.168.0.3` (w Windows usunąć `-c 4`)
- Test do LAN Rutera: `ping -c 4 192.168.0.1`
- Test do Przełącznika: `ping -c 4 192.168.0.2`

*(Uwaga pod system Windows Stacja Robocza: Upewnij się że na PC docelowym `192.168.0.3` wyłączona jest Zaporą Windows profil prywatny, ewentualnie dodana jest reguła zezwalająca na Udostępnianie Plików / Echo ICMP. Bez tego Serwer może nie połączyć się podczas testu).*

## 10. Kosztorys

Stworzenie odpowiedniego dokumentu kosztorysowego w Excelu (Kosztorys.xlsx).
Stawka roboczogodziny to 125,00 zł netto. Podatek to 23%. Należy wypisać 5 prac pochodzących z samego egzaminu.

**Przykładowe uzupełnienie Excela na bazie wykonanego egzaminu:**

| Nazwa czynności | Szacowany czas [h] | Stawka za 1 roboczogodzinę | Wartość robocizny netto |
|---|---|---|---|
| Montaż panelu krosowego i zaciśnięcie modułów | 1,00 | 125,00 zł | `=B2*C2` |
| Konfiguracja routera MikroTik | 0,50 | 125,00 zł | `=B3*C3` |
| Konfiguracja przełącznika z nadaniem adresacji | 0,50 | 125,00 zł | `=B4*C4` |
| Konfiguracja usług sieciowych i sieci Serwera | 1,00 | 125,00 zł | `=B5*C5` |
| Zarządzanie kontami użytkowników na stacji roboczej | 1,00 | 125,00 zł | `=B6*C6` |
| **Razem netto** | | | `=SUM(D2:D6)` |
| **Razem brutto** | | | `=D7*1,23` |

Obliczenia wzorcowe z poprawnym matematycznie wynikiem:

- 1 + 0,5 + 0,5 + 1 + 1 = 4h
- Razem Netto: 4h * 125 zł = **500,00 zł**
- Razem Brutto: 500 zł * 1,23 = **615,00 zł**

Na komórkach D2:D8 oraz w sekcji Stawek należy ustawić Formatowanie: **Walutowe (PLN/zł)**.
Zapisz plik pod nazwą `Kosztorys` i zgraj na nośnik.
