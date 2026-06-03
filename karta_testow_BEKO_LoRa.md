# Karta testów bezpieczeństwa radiowego
## BEKO 2025L

**Zespół w składzie:** Wiktor Sosnowski, Michał Lejwoda, Krzysztof Kulesza  
**Warszawa, dnia:** [DATA]

---

## 1. Lokalizacja stanowiska pomiarowego

![Rys. 1.1 Lokalizacja stanowiska pomiarowego w laboratorium](placeholder_rys_1_1_lokalizacja.png)

*Rys. 1.1 Lokalizacja stanowiska pomiarowego w laboratorium*

---

## 2. Opis przedmiotu badań

**Nazwa i krótki opis systemu stanowiącego przedmiot badań:**

System BEKO Pager — sieć urządzeń komunikacyjnych opartych na module radiowym LoRa SX1276 pracującym w paśmie 868 MHz. System składa się z bramy (gateway) opartej na Raspberry Pi Zero z modułem LoRa oraz węzłów końcowych (pagerów) komunikujących się z bramą za pomocą protokołu radiowego LoRa. Brama udostępnia interfejs REST API umożliwiający zarządzanie siecią, wysyłanie wiadomości i monitorowanie stanu węzłów.

---

*Rys. 2.1 Schemat blokowy systemu stanowiącego przedmiot badań*

![Rys. 2.1 Schemat blokowy systemu](placeholder_rys_2_1_schemat_blokowy.png)

---

**Elementy składowe systemu (z odniesieniami do rys. 2.1):**

- Gateway BEKO — Raspberry Pi Zero + moduł LoRa SX1276 (Rx/Tx)
- Węzeł końcowy (pager) — urządzenie z modułem LoRa (Tx/Rx)
- Interfejs webowy — panel administracyjny dostępny przez przeglądarkę

**Specyfikacja poszczególnych elementów systemu:**

**Gateway:**  
Raspberry Pi Zero, Linux (Debian Bookworm aarch64), moduł LoRa SX1276, SPI bus 0.1, reset GPIO 25, DIO0 GPIO 22, częstotliwość 868,5 MHz, sync word 0x34.

**Węzeł końcowy:**  
[UZUPEŁNIĆ — model pagera, zasilanie, parametry RF]

**Słowny opis funkcjonalności:**

System BEKO służy do bezprzewodowej komunikacji pagerowej w sieci lokalnej. Gateway odbiera i wysyła ramki LoRa do węzłów końcowych. Użytkownik za pośrednictwem panelu webowego może wysyłać wiadomości do wybranych węzłów, parować nowe urządzenia, monitorować stan sieci oraz przeglądać logi systemowe. Komunikacja między węzłami a bramą odbywa się w paśmie 868 MHz przy użyciu modulacji LoRa (Chirp Spread Spectrum).

**Czy system jest autonomiczny:**

System jest częściowo autonomiczny — gateway działa samodzielnie po uruchomieniu, nasłuchując ramek LoRa. Do zarządzania systemem wymagany jest dostęp do panelu webowego przez sieć lokalną. Węzły końcowe działają autonomicznie po sparowaniu z bramą.

**Informacje o dostępnej dokumentacji i certyfikatach:**

Dokumentacja protokołu komunikacyjnego dostępna wewnętrznie w repozytorium projektu:
- `FRAME_CRYPTO_SPEC.md` — specyfikacja kryptografii ramek
- `FRONTEND_API.md` — dokumentacja API
- `RPI_SX1276_IMPLEMENTATION.md` — implementacja sterownika LoRa
- `README-BACKEND.md` — dokumentacja backendu

Moduł SX1276: certyfikaty CE, dokumentacja producenta Semtech dostępna online.

**Czy system może być legalnie używany na terenie Polski:**

Tak — pasmo 868 MHz (ISM) jest legalne do użytku w Polsce i całej Unii Europejskiej zgodnie z dyrektywą ETSI EN 300 220. Moc nadawania oraz współczynnik wypełnienia (duty cycle) muszą spełniać wymagania regulacyjne ETSI.

**Dodatkowe informacje o systemie:**

System BEKO używa protokołu LoRa (Long Range) firmy Semtech, pracującego w paśmie 868 MHz. Poniżej zebrano informacje o znanych słabościach protokołu i implementacji:

1. **Podatność na atak typu Replay** — ramki LoRa w podstawowej konfiguracji nie zawierają mechanizmu ochrony przed powtórzeniem (brak licznika sekwencji lub znacznika czasowego weryfikowanego przez odbiorcę). Przechwyconą ramkę można odtworzyć.

2. **Brak szyfrowania warstwy radiowej** — transmisja odbywa się bez szyfrowania na poziomie RF; dane mogą być odczytane przez dowolny odbiornik LoRa w zasięgu.

3. **Podatność na zakłócanie (Jamming)** — pasmo 868 MHz nie dysponuje mechanizmami frequency hopping; zagłuszenie kanału uniemożliwia komunikację.

4. **Słabe uwierzytelnienie na warstwie aplikacji** — backend auth używa domyślnego hasła (`admin123`) jawnie opisanego w `seed.py`; brak wymuszenia zmiany hasła po pierwszym logowaniu.

5. **Brak szyfrowania dysku** — karta SD Raspberry Pi nie jest szyfrowana, co umożliwia fizyczne przejęcie systemu przez wymontowanie karty i modyfikację plików systemowych.

---

## 3. Analiza systemu metodą inżynierii odwrotnej

![Rys. 3.1 Ogólne zdjęcie systemu](placeholder_rys_3_1_ogolne_zdjecie.png)

*Rys. 3.1 Ogólne zdjęcie systemu*

---

![Rys. 3.2 Zdjęcie płytki Raspberry Pi Zero z modułem LoRa](placeholder_rys_3_2_rpi_lora.png)

*Rys. 3.2 Zdjęcie Raspberry Pi Zero z zamontowanym modułem LoRa SX1276*

---

![Rys. 3.3 Zdjęcie węzła końcowego ze zdjętą obudową](placeholder_rys_3_3_wezel.png)

*Rys. 3.3 Zdjęcie węzła końcowego (pagera) ze zdjętą obudową*

**Opis:**

Rys. 3.1 przedstawia ogólne zdjęcie badanego systemu:
- (G) Gateway — Raspberry Pi Zero z modułem LoRa SX1276
- (N) Węzeł końcowy — pager z modułem LoRa

Rys. 3.2 przedstawia zdjęcie Raspberry Pi Zero z modułem LoRa. Na zdjęciu można zidentyfikować:
- moduł LoRa SX1276 podłączony przez SPI
- antenę 868 MHz
- złącza GPIO

Rys. 3.3 przedstawia węzeł końcowy ze zdjętą obudową. Na zdjęciu można zidentyfikować:
- [UZUPEŁNIĆ — opis układu scalonego, anteny, zasilania]
- antenę nadawczo-odbiorczą
- źródło zasilania: [UZUPEŁNIĆ]

**Wnioski:**

Rozkręcenie urządzeń dostarczyło informacji o parametrach zasilania, długości i pozycji anteny oraz zastosowanych układach scalonych. Konstrukcja gateway oparta jest na powszechnie dostępnych komponentach (RPi Zero + SX1276), których dokumentacja techniczna jest publicznie dostępna — ułatwia to analizę i potencjalne ataki.

---

## 4. Analiza warstwy radiowej

**Parametry radiowe systemu — Gateway (SX1276):**

- częstotliwość fali nośnej: **868,500 MHz**
- modulacja: **LoRa (CSS — Chirp Spread Spectrum)**
- szerokość pasma: [UZUPEŁNIĆ po pomiarze — np. 125 kHz / 250 kHz / 500 kHz]
- Spreading Factor (SF): [UZUPEŁNIĆ — SF7–SF12]
- Coding Rate: [UZUPEŁNIĆ]
- sync word: **0x34**
- moc nadawania: [UZUPEŁNIĆ — dBm]
- zasięg deklarowany: [UZUPEŁNIĆ]
- zasięg rzeczywisty: [UZUPEŁNIĆ po pomiarze]

**Parametry radiowe systemu — Węzeł końcowy:**

- częstotliwość fali nośnej: **868,500 MHz**
- modulacja: **LoRa (CSS)**
- szerokość pasma: [UZUPEŁNIĆ]
- Spreading Factor (SF): [UZUPEŁNIĆ]
- zasięg rzeczywisty: [UZUPEŁNIĆ po pomiarze]

---

![Rys. 4.1 Widmo przesyłanego sygnału z zaznaczonymi krańcami pasma](placeholder_rys_4_1_widmo.png)

*Rys. 4.1 Widmo przesyłanego sygnału z zaznaczonymi krańcami pasma*

---

**Wpływ dłoni użytkownika na częstotliwość nadajnika:**

[UZUPEŁNIĆ po pomiarze]

**Opis procedury pomiaru zasięgu:**

1. Umieszczono gateway na stałej pozycji w [UZUPEŁNIĆ — opis lokalizacji]
2. Operator z węzłem końcowym udał się na maksymalną odległość (~[X] m)
3. Operator zaczął przemieszczać się w stronę gatewayu w odstępach [X] m
4. Operator zanotował pierwszą odległość, przy której udało się nadać i odebrać sygnał [X]/10 prób

---

**Parametry czasowe depeszy:**

- całkowity czas trwania depeszy: jedna ramka na transmisję (brak retransmisji ciągłych — gateway wysyła żądaną liczbę ramek)
- liczba ramek: konfigurowalna w protokole
- czas trwania ramki LoRa (szacowany dla SF7, BW 125 kHz, CR 4/5, payload [X] bajtów):

| Parametr | Wartość |
|---|---|
| Preamble | [UZUPEŁNIĆ] ms |
| Header | [UZUPEŁNIĆ] ms |
| Payload | [UZUPEŁNIĆ] ms |
| **Całkowity czas ramki** | **[UZUPEŁNIĆ] ms** |

- odstęp pomiędzy ramkami: [UZUPEŁNIĆ] ms
- liczba bajtów w ramce: 45 lub 61 (na podstawie kodu gateway: `Błędna długość ramki: X to nie [45, 61]`)
- czas trwania symbolu LoRa: 2^SF / BW = [UZUPEŁNIĆ] ms

---

![Rys. 4.2 Zapis czasowy ramki LoRa w SigDigger](placeholder_rys_4_2_ramka_czasowa.png)

*Rys. 4.2 Zapis czasowy ramki LoRa (widok w SigDigger)*

---

![Rys. 4.3 Widmo ramki LoRa — waterfall](placeholder_rys_4_3_waterfall.png)

*Rys. 4.3 Widmo ramki LoRa — waterfall (SigDigger / GNU Radio)*

---

**Prędkość transmisji:**

Dla modulacji LoRa prędkość bitowa wyraża się wzorem:

```
Rb = SF * (BW / 2^SF) * (4 / (4 + CR))
```

Dla SF=[X], BW=[X] kHz, CR=[X]:

- Prędkość symbolowa: [UZUPEŁNIĆ] Bd
- Prędkość bitowa brutto: [UZUPEŁNIĆ] bit/s
- Prędkość bitowa netto (z uwzględnieniem nagłówka i preambuły): [UZUPEŁNIĆ] bit/s

---

## 5. Analiza depeszy

**Nietypowe elementy depeszy:**

Na podstawie kodu źródłowego gateway (`FRAME_CRYPTO_SPEC.md`) ramka zawiera:
- pole nagłówka z identyfikatorem nadawcy i odbiorcy
- pole danych (payload)
- pole HMAC (podpis kryptograficzny ramki)
- pole numeru sekwencji (counter) — [UZUPEŁNIĆ: czy weryfikowany?]

**Czy wszystkie ramki danej depeszy są identyczne:**

[UZUPEŁNIĆ po analizie przechwyconych ramek] — w protokołach z licznikiem sekwencji kolejne ramki tej samej wiadomości powinny różnić się polem counter. Weryfikacja wymaga przechwycenia kilku kolejnych transmisji.

**Ramka (zapis binarny/hex przechwyconych danych):**

```
Ramka 1: [UZUPEŁNIĆ — hex dump]
Ramka 2: [UZUPEŁNIĆ — hex dump]
Ramka 3: [UZUPEŁNIĆ — hex dump]
```

**Liczba przechwyconych ramek vs reakcja odbiornika:**

| Liczba ramek | Reakcja odbiornika |
|---|---|
| 1 | [UZUPEŁNIĆ] |
| 2 | [UZUPEŁNIĆ] |
| ≥ 3 | [UZUPEŁNIĆ] |

**Elementy składowe ramki:**

Na podstawie `FRAME_CRYPTO_SPEC.md` oraz analizy binarnej przechwyconych danych:

| Pole | Długość [B] | Opis |
|---|---|---|
| dst_id | [X] | identyfikator odbiorcy |
| src_id | [X] | identyfikator nadawcy |
| counter | [X] | licznik sekwencji |
| payload | [X] | dane użytkowe |
| HMAC | [X] | podpis kryptograficzny |

**Stały ciąg bitów / wzorzec ramki:**

[UZUPEŁNIĆ po analizie w Inspectrum / SigDigger]

---

## 6. Analiza podatności na wybrane typy ataków

### Atak typu REPLAY

Do przeprowadzenia ataku typu Replay nagrywano ramki LoRa emitowane przez węzeł końcowy. Nagranie wykonano przy użyciu PlutoSDR / RTL-SDR i programu SigDigger / GNU Radio, próbkując sygnał z częstotliwością [UZUPEŁNIĆ] ksps na częstotliwości 868,5 MHz.

Nagrano [X] ramek. Następnie przygotowano pliki `.raw` zawierające [3/4/5] ramek i odtworzono je przez PlutoSDR w GNU Radio Companion z interpolacją do 2M próbek/s.

**Konfiguracja GNU Radio Companion — atak Replay:**

![Rys. 6.1 Konfiguracja GNU Radio Companion — atak Replay](placeholder_rys_6_1_gnuradio_replay.png)

*Rys. 6.1 Konfiguracja GNU Radio Companion — odtwarzanie przechwyconych ramek*

**Parametry konfiguracji nadajnika:**
- LO Frequency: 868,5 MHz
- Sample Rate: 2M
- Attenuation TX: [UZUPEŁNIĆ] dB

**Parametry konfiguracji odbiornika:**
- LO Frequency: 868,5 MHz
- Sample Rate: [UZUPEŁNIĆ]

**Komentarz po przeprowadzonym ataku:**

[UZUPEŁNIĆ — opis wyniku]

Wyniki wstępne (do uzupełnienia po eksperymencie):
- Czy atak się powiódł: [TAK / NIE / CZĘŚCIOWO]
- Minimalna liczba ramek potrzebna do wywołania reakcji: [UZUPEŁNIĆ]
- Graniczne przesunięcie częstotliwości: [UZUPEŁNIĆ] kHz
- Graniczne spowolnienie próbkowania: [UZUPEŁNIĆ]×
- Graniczne przyspieszenie próbkowania: [UZUPEŁNIĆ]×

Jeśli system używa HMAC i licznika sekwencji, atak replay powinien zostać odrzucony przez gateway. Weryfikacja tego mechanizmu jest głównym celem eksperymentu.

---

### Odporność systemu na zakłócanie (Jamming)

Korzystając z GNU Radio Companion przeprowadzono atak typu Jamming. Zastosowano następujące rodzaje zagłuszania:

**Układ badawczy:**

![Rys. 6.2 Układ badawczy — atak Jamming](placeholder_rys_6_2_uklad_jamming.png)

*Rys. 6.2 Układ badawczy — atak Jamming*

**Aplikacja GNU Radio Companion — atak Jamming:**

![Rys. 6.3 Konfiguracja GNU Radio Companion — Jamming](placeholder_rys_6_3_gnuradio_jamming.png)

*Rys. 6.3 Aplikacja GNU Radio Companion do ataku typu Jamming*

**Zagłuszanie szerokopasmowe:**

![Rys. 6.4 Zagłuszanie szerokopasmowe — widmo](placeholder_rys_6_4_jamming_szerokie.png)

*Rys. 6.4 Zagłuszanie szerokopasmowe — widmo w SigDigger*

[UZUPEŁNIĆ — opis wyniku. Oczekiwany wynik: system przestaje reagować na polecenia.]

**Zagłuszanie pasmowe (na 868,5 MHz):**

[UZUPEŁNIĆ — opis wyniku i skuteczności]

**Nadawanie nośnej:**

[UZUPEŁNIĆ — opis wyniku]

**Komentarz:**

LoRa charakteryzuje się wysoką odpornością na zakłócenia dzięki technice CSS (Chirp Spread Spectrum) i możliwości pracy przy ujemnym SNR (nawet do -20 dB). Weryfikacja rzeczywistej odporności systemu jest celem eksperymentu.

**Wpływ modyfikacji parametrów sygnału na skuteczność zagłuszania:**

[UZUPEŁNIĆ — badanie wpływu attenuation i szerokości pasma zakłócacza]

---

### Atak typu Brute Force (BF)

**Wyznaczenie liczby prób pełnego ataku BF:**

Na podstawie analizy ramki (sekcja 5) ramka zawiera [X] bitów zmiennych (identyfikator + payload). Pełna przestrzeń kluczy:

```
Liczba kombinacji = 2^[X]

Czas transmisji pojedynczej ramki LoRa = [UZUPEŁNIĆ] ms
Minimalna liczba ramek do reakcji odbiornika = [UZUPEŁNIĆ]
Czas na próbę = [UZUPEŁNIĆ] ms × [liczba ramek] = [UZUPEŁNIĆ] ms

Czas pełnego ataku BF = 2^[X] × [czas_próby] ms = [UZUPEŁNIĆ] dni
```

**Założenie — znana część klucza:**

Zakładamy, że znany jest identyfikator źródłowy i docelowy (src_id, dst_id) — widoczne w nagłówku ramki. Zgadujemy [X] bitów pola payload / counter = [2^X] kombinacji.

Czas ataku przy znanych [X] bitach:
```
2^[X] × [czas_próby] ms = [UZUPEŁNIĆ]
```

**Konfiguracja GNU Radio Companion — atak BF:**

![Rys. 6.5 Synteza sygnału — atak Brute Force — GNU Radio Companion](placeholder_rys_6_5_gnuradio_bf.png)

*Rys. 6.5 Synteza sygnału — atak Brute Force — GNU Radio Companion z modułem Python generującym zestawy ramek*

**Procedura ataku:**

1. Na podstawie analizy przechwyconych ramek wyznaczono stałą część ramki (nagłówek, identyfikatory)
2. Przygotowano skrypt Python generujący wszystkie kombinacje zmiennej części ramki
3. Zbudowano flowgraph w GNU Radio Companion: Vector Source → Repeat → PlutoSDR Sink (868,5 MHz, 2M sample)
4. Uruchomiono transmisję kolejnych kombinacji i obserwowano reakcję odbiornika

**Komentarz:**

[UZUPEŁNIĆ — opis wyniku]

Jeśli system weryfikuje HMAC, atak BF na poziomie radiowym jest praktycznie niemożliwy bez znajomości klucza. Weryfikacja tego mechanizmu jest celem eksperymentu.

---

## 7. Wnioski i rekomendacje

**Podsumowanie prac:**

W ramach badań przeprowadzono analizę systemu BEKO Pager obejmującą:
- analizę inżynierii odwrotnej sprzętu (gateway + węzły końcowe)
- pomiary parametrów warstwy radiowej (częstotliwość, modulacja, zasięg, parametry czasowe)
- analizę struktury ramek protokołu
- testy podatności na ataki: Replay, Jamming, Brute Force

**Wnioski:**

1. **Warstwa radiowa** — System pracuje na 868,5 MHz z modulacją LoRa (CSS), która zapewnia dobrą odporność na zakłócenia i duży zasięg. Pasmo ISM 868 MHz jest legalne w Polsce i UE.

2. **Kryptografia ramek** — Kod źródłowy sugeruje obecność mechanizmu HMAC (`FRAME_CRYPTO_SPEC.md`, `gateway_master.key`). Skuteczność weryfikacji licznika sekwencji przeciwko atakom replay wymaga potwierdzenia eksperymentalnego.

3. **Atak Replay** — [UZUPEŁNIĆ po przeprowadzeniu eksperymentu — TAK/NIE skuteczny]

4. **Atak Jamming** — System jest podatny na zagłuszanie — LoRa nie implementuje frequency hopping. Skuteczność zagłuszania zależy od mocy zakłócacza i odległości.

5. **Atak Brute Force** — Przy użyciu HMAC pełny atak BF jest praktycznie niewykonalny w rozsądnym czasie. Bez HMAC czas ataku wynosi [UZUPEŁNIĆ] dni.

6. **Bezpieczeństwo fizyczne** — Brak szyfrowania karty SD umożliwia fizyczne przejęcie systemu. Domyślne hasło `admin123` w backendzie jest poważną słabością.

**Rekomendacje:**

| Nr | Rekomendacja | Priorytet |
|---|---|---|
| 1 | Zmienić domyślne hasło administratora i wymusić zmianę przy pierwszym logowaniu | Wysoki |
| 2 | Wdrożyć szyfrowanie karty SD (LUKS) | Wysoki |
| 3 | Weryfikować licznik sekwencji w ramkach w celu blokowania ataków replay | Wysoki |
| 4 | Wyłączyć logowanie SSH hasłem — używać wyłącznie kluczy SSH | Średni |
| 5 | Rozważyć frequency hopping lub dodatkowe szyfrowanie warstwy aplikacji | Średni |
| 6 | Wdrożyć monitoring anomalii RF (wykrywanie jammingu) | Niski |
| 7 | Ograniczyć CORS w środowisku produkcyjnym — nie używać `"*"` | Średni |

---

## Dodatek 1. Archiwum sygnałów radiowych

Poniżej opisano strukturę archiwum nagranych sygnałów radiowych:

```
archiwum/
├── replay/
│   ├── ramka_gateway_tx_[DATA].raw       # nagranie ramki wysyłanej przez gateway, [X] ramek
│   ├── ramka_wezel_tx_[DATA].raw         # nagranie ramki wysyłanej przez węzeł końcowy
│   └── metadata.txt                      # opis: częstotliwość próbkowania, cel nagrania
├── jamming/
│   ├── jamming_szerokopasmowe_[DATA].raw # nagranie podczas ataku szerokopasmowego
│   ├── jamming_pasmowe_[DATA].raw        # nagranie podczas ataku pasmowego
│   └── jamming_nosna_[DATA].raw          # nagranie podczas nadawania nośnej
├── brute_force/
│   ├── bf_sekwencja_[DATA].raw           # nagranie sekwencji ramek BF
│   └── generator_940aprc.py              # skrypt Python generujący kombinacje ramek
└── baseline/
    ├── widmo_868MHz_bez_transmisji.raw   # tło radiowe bez aktywnej transmisji
    └── widmo_868MHz_transmisja.raw       # widmo podczas normalnej pracy systemu
```

**Zawartość plików metadanych (metadata.txt):**

```
Data nagrania: [DATA]
Narzędzie: PlutoSDR / RTL-SDR + SigDigger / GNU Radio
Częstotliwość centralna: 868,5 MHz
Częstotliwość próbkowania: [X] ksps / Msps
Cel nagrania: [opis, np. "akwizycja ramki wysyłanej przez gateway przy komendzie send_message do węzła 0x01"]
Wynik: [opis zaobserwowanego zachowania systemu]
```

[UZUPEŁNIĆ — rzeczywista struktura katalogów po przeprowadzeniu badań]
