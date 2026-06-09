# Rozwiązanie egzaminu INF.02 (Styczeń 2025, wersja 04)

Poniżej znajduje się kompletna dokumentacja wykonania zadań egzaminacyjnych zgodnie z instrukcją inf_02_2025_01_04_sg.pdf oraz dodatkowymi wytycznymi z pliku README.

## 1. Okablowanie sieciowe

*Zgodnie z wytyczną A:* Pomijamy zadanie dotyczące fizycznego montażu okablowania sieciowego (wykonanie kabla połączeniowego prostego w standardzie T568B i jego testowanie w obecności Egzaminatora).

## 2. Identyfikacja podzespołów (Linux) oraz Zasilacza

Zgodnie z poleceniem, stacja robocza operuje na systemie Linux do wykonania tego zadania.

**1. Identyfikacja procesora (`cpu.jpg`)**
Należy otworzyć terminal na koncie administratora i wywołać polecenie identyfikujące procesor.

```bash
lscpu | grep "Model name"
# LUB
cat /proc/cpuinfo | grep "model name" | head -n 1
```

Należy wywołać narzędzie do zrzutów ekranu (np. `gnome-screenshot` lub PrintScreen), zapisać obraz jako `cpu.jpg` i umieścić go na nośniku USB w folderze `Egzamin-x`. Odczytany producent i model należy zapisać do Tabeli 1 w arkuszu.

**2. Identyfikacja dysku i wolnego miejsca (`dysk.jpg`)**
Aby zidentyfikować ilość wolnego miejsca na dysku (partycji systemowej), można użyć polecenia:

```bash
df -h
```

W kolumnie "Avail" (Dostępne) dla partycji zmontowanej w katalogu głównym `/` odczytujemy ilość wolnego miejsca. Dodatkowo dla pełnego obrazu można użyć dyskowego narzędzia np. `lsblk` lub `fdisk -l`.
Ponownie, robimy zrzut, nazywamy go `dysk.jpg` i wgrywamy na pendrive, a wynik przepisujemy do Tabeli 1.

**3. Odczyt z zasilacza (Tabliczka znamionowa)**
Należy otworzyć obudowę stacji roboczej, obejrzeć tabliczkę znamionową wklejoną na zasilaczu ATX/ITX. Należy odczytać:

- **Moc maksymalną w trybie ciągłym** (zazwyczaj podana jako "Max. Output Power" np. 500W, wpisać 500 w kolumnie Wartość i "W" w kolumnie Jednostka).
- **Napięcie wejściowe** (Input Voltage, np. 230 V).
- **Napięcia wyjściowe** (Output Voltages, zazwyczaj wymienione jako np. +3.3V, +5V, +12V, -12V, +5VSB. Wpisać je do rubryki, a w jednostkach "V").

## 3. Konfiguracja rutera MikroTik

*Zgodnie z wytyczną B:* Skonfigurowano ruter MikroTik (jako połączone urządzenie) z poziomu CLI (RouterOS).

**Polecenia konfigurujące ruter MikroTik:**

```routeros
# Reset ustawień fabrycznych (opcjonalnie)
/system reset-configuration no-defaults=yes skip-backup=yes

# 1. Konfiguracja interfejsu WAN (100.100.100.8/28) i bramy (100.100.100.1)
/ip address add address=100.100.100.8/28 interface=ether1
/ip route add distance=1 gateway=100.100.100.1

# 2. Konfiguracja serwerów DNS (4.4.4.4 i 8.8.8.8)
/ip dns set servers=4.4.4.4,8.8.8.8 allow-remote-requests=yes

# 3. Konfiguracja interfejsu LAN (192.168.0.1/24)
/ip address add address=192.168.0.1/24 interface=ether2

# 4. Serwer DHCP jest domyślnie wyłączony, ale dla upewnienia się można wymusić usunięcie z LAN
/ip dhcp-server disable [find interface=ether2]
```

**Jak odczytać dane z konfiguracji w MikroTik (Weryfikacja):**
Aby szybko pokazać wyniki egzaminatorowi w terminalu:

- Zobaczenie adresów IP: `/ip address print`
- Zobaczenie ustawień bramy: `/ip route print`
- Zobaczenie ustawień DNS: `/ip dns print`
- Zobaczenie ustawień DHCP: `/ip dhcp-server print`

## 4. Konfiguracja przełącznika MikroTik

*Zgodnie z wytyczną B:* Skonfigurowano przełącznik z poziomu CLI (RouterOS / warstwa Bridge).

**Polecenia konfigurujące przełącznik MikroTik:**

```routeros
# Tworzenie interfejsu Bridge, który łączy porty przełącznika
/interface bridge add name=bridge_lan
/interface bridge port add bridge=bridge_lan interface=all

# 1. Przypisanie adresu IP dla przełącznika (192.168.0.10/24)
/ip address add address=192.168.0.10/24 interface=bridge_lan

# 2. Ustawienie bramy domyślnej na IP rutera (192.168.0.1)
/ip route add distance=1 gateway=192.168.0.1
```

*(Weryfikacja: `ip address print` i `ip route print`)*

## 5. Połączenie urządzeń

- Należy podłączyć Ruter (interfejsem LAN - ether2) do Przełącznika.
- Należy podłączyć Serwer oraz Stację Roboczą kablem Ethernet do portów Przełącznika.
- Należy podłączyć drukarkę sieciową również do portu Przełącznika (w sali z gniazda E-X).

## 6. Konfiguracja Serwera

Zgodnie z poleceniem i standardem edukacyjnym, Serwer (Windows Server D1 i alternatywnie Linux D2) pełni rolę kontrolera domeny Active Directory oraz serwera wydruku z narzuconymi politykami grupy (GPO). Z racji że Active Directory i GPO to natywne technologie Windows Server - to w nim przebiega to w 100% z interfejsu graficznego. Linuksowym odpowiednikiem (D2) byłoby zestawienie Samba4 AD DC, co również zwięźle opiszemy pod koniec punktu.

### Część 6a: Interfejs i Usługi Active Directory (Windows Server - D1)

**1. Sieć serwera:**
Otwórz właściwości karty sieciowej połączonej z przełącznikiem (`ncpa.cpl`).
Skonfiguruj protokół IPv4:

- Adres IP: `192.168.0.101` *(dla stanowiska X=1; 100+1)*.
- Maska: `255.255.255.0`
- Brama domyślna: `192.168.0.1`
- Serwer DNS: `127.0.0.1` lub `::1` *(localhost, ponieważ instalujemy tu kontroler AD)*.

**2. Instalacja Active Directory i Promowanie:**

- Menedżer Serwera -> Zarządzanie -> Dodaj role i funkcje. Zaznacz Usługi domenowe Active Directory (AD DS). Zainstaluj.
- Po instalacji kliknij ikonę powiadomienia (flagę z żółtym wykrzyknikiem) w Menedżerze Serwera -> "Podnieś poziom tego serwera do poziomu kontrolera domeny" (Promote).
- Wybierz "Dodaj nowy las" (Add a new forest). Nazwa domeny głównej: `inf02.local`.
- Wpisz dwukrotnie hasło trybu przywracania usług katalogowych (DSRM): `ZAQ!2wsx`.
- Przeklikaj do końca i zainstaluj. Serwer zrestartuje się.

**3. Tworzenie jednostek organizacyjnych, użytkowników i grup:**
Zaloguj się na serwer już do domeny jako `INF02\Administrator`.

- Otwórz Użytkownicy i komputery usługi Active Directory (`dsa.msc`).
- Prawy przycisk na domenę `inf02.local` -> Nowe -> Jednostka organizacyjna (Organizational Unit).
  - Utwórz jednostkę: `Egzaminatorzy`.
  - Powtórz i utwórz jednostkę: `Uczniowie`.
- Prawy przycisk na `Egzaminatorzy` -> Nowy -> Użytkownik.
  - Imię: Jan, Nazwisko: Abacki, Nazwa logowania (User logon name): `jabacki`.
  - Hasło: `ZAQ1@wsx1`. (Odznacz "Użytkownik musi zmienić hasło" i opcjonalnie zaznacz "Hasło nigdy nie wygasa").
- Prawy przycisk na `Uczniowie` -> Nowy -> Użytkownik.
  - Imię: Zenon, Nazwisko: Babacki, Nazwa logowania: `zbabacki`.
  - Hasło: `ZAQ1@wsx2`. (Odznacz "Użytkownik musi zmienić hasło").
- Prawy przycisk np. na strukturze domeny lub w wybranej OU -> Nowa -> Grupa.
  - Nazwa grupy: `Egzamin`. Typ: Zabezpieczeń, Zakres: Globalna.
- Podwójny klik na grupę `Egzamin` -> Członkowie -> Dodaj... wpisz `jabacki; zbabacki` i zastosuj.

### Część 6b: Usługi druku, GPO, i RDP (Windows Server - D1)

**1. Udostępnienie drukarki:**

- W Menedżerze serwera dodaj rolę: **Usługi drukowania i zarządzania dokumentami**.
- Otwórz Menedżer wydruku (Print Management -> `printmanagement.msc`).
- Serwery wydruku -> Nazwa_serwera -> Drukarki -> Prawy klik -> Dodaj Drukarkę.
- "Dodaj nową drukarkę przy użyciu portu TCP/IP": wpisz adres `192.168.0.200`. Upewnij się w ustawieniach portu, że wybrano standard RAW. Użyj ogólnego sterownika (np. Generic Text Only).
- Nazwij drukarkę: `Egzamin_druk` i zaznacz opcję jej udostępnienia, nadając taką samą nazwę udziału. Zakończ kreatora.
- Kliknij Prawym na nową drukarkę `Egzamin_druk` we właściwości.
- Zakładka **Zaawansowane**: Zaznacz "Dostępne od" i wpisz `08:00` do `22:00`. Ustaw Priorytet najwyżej (zwykle suwak na wartość 99).
- Zakładka **Zabezpieczenia**: Usuń lub odznacz zezwolenie "Drukuj" dla `Wszyscy`. Dodaj grupę `Egzamin` i nadaj jej Zezwolenie na Drukuj. Dodaj `TWÓRCA WŁAŚCICIEL` z zaznaczonym "Zarządzaj dokumentami". `Administrator` powinien mieć domyślnie zaznaczone wszystko (Pełna kontrola - Manage printers, Manage documents, Print).
- Aby rozmieścić przez GPO, nadal w Menedżerze wydruku, prawy klik na drukarkę -> **Wdróż za pomocą zasad grupy... (Deploy with Group Policy)**. Utwórz nowy obiekt GPO i podepnij go pod OU `Uczniowie`, lub podepnij pod istniejący, zaznacz "Dla użytkowników, do których stosuje się ten obiekt (User)". (Zaleca się to po utworzeniu GPO z kolejnego punktu, jednak kreator z tego miejsca sam jest w stanie to utworzyć).

**2. Zasady grupy (GPO) i zablokowanie panelu sterowania:**

- Otwórz Zarządzanie zasadami grupy (`gpmc.msc`).
- Rozwiń domenę `inf02.local`, prawy przycisk na OU `Uczniowie` -> **Utwórz obiekt zasad grupy w tej domenie i umieść do niego łącze w tym miejscu...**
- Nazwa obiektu GPO: `Panel`. (Jeżeli wygenerowano GPO we wcześniejszym punkcie drukarki, można po prostu wyedytować tamto GPO jeśli dostało taką nazwę, albo mieć dwie osobne).
- Prawy przycisk na GPO `Panel` -> Edytuj.
- Konfiguracja użytkownika -> Zasady -> Szablony administracyjne -> Panel sterowania.
- Zmień stan na "Włączone" dla zasady: **Zabroń dostępu do Panelu sterowania i ekranu Ustawienia komputera** (Prohibit access to Control Panel and PC settings).

**3. Pulpit zdalny i uwierzytelnienie poziomu sieci (NLA):**

- Otwórz Menedżer Serwera -> Serwer lokalny. Znajdź sekcję Pulpit zdalny, kliknij na "Wyłączony".
- W oknie Właściwości systemu zaznacz **"Zezwalaj na połączenia zdalne z tym komputerem"**.
- Zaznacz pod spodem opcję **"Zezwalaj na połączenia tylko z komputerów, na których uruchomiono Pulpit zdalny z uwierzytelnieniem na poziomie sieci (zalecane)"** *(ang. NLA)*. Zatwierdź OK. (Może wyskoczyć dymek o otwarciu wyjątku w Zaporze, potwierdź).

### Odpowiednik Linux Server (D2)

- Zamiast Active Directory korzysta się z pakietu `samba`.
  - Promowanie: `samba-tool domain provision --use-rfc2307 --interactive`
  - Wpisywanie odpowiedniej nazwy domeny i hasła administratora `ZAQ!2wsx`.
- Zarządzanie użytkownikami: `samba-tool user create zbabacki ZAQ1@wsx2`, dodawanie do OU, tworzenie grup itd. za pomocą składni z użyciem przełączników katalogowych, lub wykorzystanie z klienta RSAT z Windowsa.
- Zarządzanie Drukiem: W linuksie za drukarkę odpowiada CUPS i edycja plików (`/etc/cups/cupsd.conf`) i dodanie polityk z portem i uprawnieniami za pomocą demona lpadmin oraz samby udostępniającej drukarkę.
- Zamiast Pulpitu Zdalnego (RDP) natywnego dla Windows w środowisku z GUI uruchamia się `xrdp`, po zainstalowaniu np. lekkiego xfce. Uwierzytelnienie domyślnie wykorzysta mechanizm autentykacji Linuxa przed udostępnieniem pulpitu, zachowując zachowanie zbliżone do wymogów NLA dla Windows.

## 7. Konfiguracja stacji roboczej

### Część 7a: Konfiguracja i Testy Zasad - system Windows (C1)

**1. Adresacja sieciowa:**
Otwórz właściwości interfejsu sieciowego (`ncpa.cpl`) -> Właściwości IPv4:

- Adres IP: `192.168.0.51` *(dla stanowiska X=1; 50+1)*.
- Maska podsieci: `255.255.255.0`
- Brama domyślna: `192.168.0.1` (IP rutera wg polecenia).
- Preferowany serwer DNS: `192.168.0.101` (IP serwera wg polecenia).

**2. Dodanie stacji do domeny:**

- Należy upewnić się, że można zpingować serwer oraz zapytanie DNS na nazwę `inf02.local` odpowiada (np. za pomocą nslookup).
- Otwórz w Windows 10/11 "Zaawansowane Ustawienia Systemu" (sysdm.cpl).
- Nazwa komputera -> Zmień... -> Zaznacz pole "Domena" i wpisz `inf02.local`.
- Wprowadź poświadczenia administratora domeny `Administrator` z hasłem `ZAQ!2wsx`. Zrestartuj komputer po pomyślnym dodaniu.

**3. Pulpit zdalny i Weryfikacja (Egzaminator):**

- Włącz komputer, lecz zamiast logować się stacjonarnie na własnym systemie stacji roboczej, uruchom systemową aplikację "Podłączanie pulpitu zdalnego" (`mstsc`).
- Wpisz IP serwera (np. 192.168.0.101) i wciśnij Połącz. Zaakceptuj ewentualny certyfikat.
- Podaj poświadczenia użytkownika dziedziczącego GPO: `INF02\zbabacki` i hasło `ZAQ1@wsx2`.
- Po załadowaniu sesji zdalnej na serwerze, kliknij w Start i spróbuj otworzyć "Panel sterowania". Zgodnie z prawidłowo wykonanym zadaniem powinien pojawić się dymek z czerwoną ikoną i ostrzeżeniem "Ta operacja została anulowana ze względu na ograniczenia...".
- Nadal połączony na serwer (lub logując się potem stacjonarnie, drukarka powinna być przemapowana przez GPO), w pasku start wyszukaj Drukarki i Skanery, lub Panel Sterowania -> Urządzenia i drukarki, znajdź `Egzamin_druk`, kliknij prawym (lub we Właściwości) i zlokalizuj opcję "Drukuj stronę testową". Jeśli ustawienia sieci są prawidłowe, zostanie wysłana ramka. (Nie zapomnij zgłosić tego etapu zgodnie z uwagą).

### Część 7b: Odpowiedniki dla systemu Linux (Stacja Robocza - C2)

Jeśli korzystano z Linuxa jako systemu na stanowisku:

1. **Adresacja:** Ustawienie poprzez Netplan lub `nmcli`, wskazując IP z zakresu np. 192.168.0.51, maskę /24, bramę i DNS (np. 192.168.0.101).
2. **Dodanie do domeny AD:** Wymaga zainstalowania realmd, sssd i adcli. Należy upewnić się że wpis `/etc/resolv.conf` zawiera resolver serwera, a nastepnie wydanie polecenia `realm join inf02.local -U Administrator` i poprawne zalogowanie systemowe.
3. **Pulpit zdalny:** W środowisku GUI dla Linuksa można połączyć się z usługami Microsoft RDP używając wbudowanego klienta np. Remmina (`sudo apt-get install remmina`), wpisując tam parametry uwierzytelnienia z domeny (Użytkownik: `zbabacki`, Domena: `inf02.local`) a protokół ustawiając na standardowy RDP, i tak również zalogować się z weryfikacją polityk.

## 8. Testy komunikacji stacji roboczej

Z polecenia wynika test pingu od strony stacji roboczej. Uruchom wiersz poleceń `cmd` (Windows) lub terminal (Linux):

- Do rutera: `ping 192.168.0.1`
- Do serwera: `ping 192.168.0.101`
- Do drukarki: `ping 192.168.0.200`

Jeśli z jakiegoś powodu serwer uważa naszą komunikację za nienależycie zaufaną, lub z innego stanowiska, należy pamiętać, że pod adresem Serwera działa firewall. Aby dopuścić ICMP Echo Request - w Serwerze przejdź do: Zaawansowane zabezpieczenia zapory systemu Windows -> Reguły przychodzące -> włącz "Udostępnianie plików i drukarek (żądanie echa - ruch przychodzący ICMPv4)" (File and Printer Sharing Echo Request). O ile przy domenie często polityki przepuszczają ruch dla sieci profilu domenowego, o tyle sprawdzenie Zapory na Serwerze jest wylistowane w podpunkcie zadania.

## 9. Kalkulacja Mocy

Zadanie wymaga użycia arkusza kalkulacyjnego (`kalkulacja_mocy.xlsx`) zgodnie ze wzorem z Tabeli 3.

**Odtworzenie arkusza i poprawne formuły w komórkach (od wiersza nr 9):**
Zakładając, że na samej górze masz uzupełnione poszczególne zidentyfikowane czy narzucone komponenty oraz ich szacowany pobór mocy (w Watach). Otrzymujemy podsumowanie.

- **Komórka B9:** Oczekiwana automatyczna suma dotychczas wymienionych elementów. Należy wprowadzić formułę używając zakresu mocy od B2 do B8 (zależnie od wierszy):
  `=SUMA(B2:B8)` lub `=SUM(B2:B8)` w zależności od języka.
- **Komórka B11:** Należy wpisać moc stałą, którą odczytano z tabliczki znamionowej na samym początku (np. wpisać sztywne `500`).
- **Komórka B12:** Obliczenie automatyczne łącznej mocy z 20% buforem/marginesem bezpieczeństwa. Zgodnie z instrukcją: "wartość równa B9 powiększona o 20%". Formuła:
  `=B9*1,20` (lub `B9*1.20` z kropką przy konfiguracji EN-US). Można też wpisać `=B9+(B9*20%)`.
- **Komórka B13:** Komórka ma decydować o zapotrzebowaniu, pisząc czy zasilacz pociągnie modernizację ("NIE" (nie pociągnie) - lub "TAK" jeśli w przeciwnym razie). Należy napisać poprawną instrukcję warunkową. Instrukcja z egzaminu jest opisana tak: "*ustalany napis „NIE”, jeżeli moc z komórki B11 jest większa lub równa mocy z komórki B12 lub napis „TAK” w przeciwnym wypadku*".
  Zwróć uwagę na logikę egzaminacyjną - zwykle jeśli B11 (nasz zasilacz) jest większy lub równy niż wymogi modernizacji B12, to logicznie "Pociągnie", ale treść wymusza tutaj napis "NIE" jeśli ten warunek jest spełniony (często te arkusze są specyficznie sformułowane w zadaniach lub pytają "Czy trzeba wymienić zasilacz?"). Postępuj dokładnie według instrukcji z arkusza dla formuły:

  ```excel
  =JEŻELI(B11>=B12; "NIE"; "TAK")
  ```

  *(Jeżeli angielski pakiet Office/LibreOffice to `IF(B11>=B12, "NIE", "TAK")`).*

Zapisz skoroszyt na nośniku USB w podanym pliku i nie wyłączaj sprzętu do sprawdzania.
