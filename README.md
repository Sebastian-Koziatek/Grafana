# 📊 Grafana - Kompleksowe Materiały Szkoleniowe

Repozytorium zawiera pełne materiały szkoleniowe do nauki Grafany - platformy do wizualizacji danych i monitoringu infrastruktury IT.

## 📚 Struktura Materiałów

### I. Wprowadzenie
- **Grafana Cloud** - rozwiązanie chmurowe
- **Grafana Enterprise** - wersja dla przedsiębiorstw
- **Grafana OSS** - wersja open source
- **Instalacja i konfiguracja wersji OSS** - pierwsze kroki

### II. Data Source – Integracja ze Źródłem Danych
- **PostgreSQL** - integracja z bazą danych relacyjną
- **InfluxDB** - bazy danych szeregów czasowych
- **Prometheus** - system monitoringu i alertowania
- **Elasticsearch** - wyszukiwarka i analiza logów
- **Zabbix** - monitoring infrastruktury IT
- **MySQL** - relacyjna baza MySQL / MariaDB (m.in. dane do geolokalizacji)

### III. Dashboard – Zestaw Wykresów i Wskaźników
- **Foldery** - organizacja dashboardów
- **Playlisty** - automatyczne przewijanie dashboardów
- **Import - Export** - udostępnianie i backup
- **Ustawienia** - konfiguracja dashboardów
- **Wiersz** - organizacja paneli
- **Panel** - podstawowy element dashboardu

### IV. Visualization – Formy Prezentacji Danych
- **Time series** - wykresy szeregów czasowych
- **Bar chart** - wykresy słupkowe
- **Stat** - wyświetlanie pojedynczych wartości
- **Gauge** - wskaźniki miernikowe
- **Bar gauge** - paski poziome
- **Table** - prezentacja tabelaryczna
- **Pie chart** - wykresy kołowe
- **State timeline** - linia czasu stanów
- **Heatmap** - mapy cieplne
- **Status history** - historia statusów
- **Histogram** - rozkład wartości
- **Text** - teksty i notatki
- **Geomap** - wizualizacja geograficzna

### V. Variables – Tworzenie Dynamicznych Dashboardów
- **Konfiguracja zmiennych** - parametryzacja dashboardów
- **Typy zmiennych** - Query, Custom, Interval, Constant, Data source, Ad hoc filters, Text box
- **Zmienne zależne i składnia** - łańcuchy zmiennych, formaty `${var:format}`

### VI. Alerting – Obsługa Notyfikacji
- **Alert rules** - reguły alertowania
- **Expresje** - wyrażenia w alertach
- **Alert messages** - wiadomości powiadomień
- **Contact points** - punkty kontaktowe
- **Notification policies** - polityki powiadomień
- **Silences** - wyciszanie alertów

### VII. Administration – Zarządzanie Platformą
- **Metody logowania** - uwierzytelnianie użytkowników
- **Users and Roles** - zarządzanie użytkownikami i rolami
- **Teams** - praca zespołowa
- **Orgs** - organizacje w Grafanie

### VIII. Plugins – Rozszerzenie Funkcjonalności
- **Typy pluginów** - Data source, Panel, App; katalog i podpisy
- **Instalacja i zarządzanie** - UI, `grafana cli`, aktywacja App, air-gapped
- **Popularne pluginy** - Zabbix, Infinity, Clock; pluginy wycofane i ich zamienniki

### XIV. Grafana-CLI
- Narzędzie linii poleceń `grafana cli` (dawniej `grafana-cli`) – pluginy i administracja

### XV. Zadania
- **Dashboard i wizualizacja** - praktyczne ćwiczenia
- **Podsumowanie** - zadania końcowe

## 🧪 Środowisko laboratoryjne

Materiały są dopasowane do labu z **Grafana 13.3 OSS** na Ubuntu 26.04 LTS (maszyna `vmzabbixsrv16`, adres **10.123.2.206**). Zrzuty ekranu `grafiki/lab-*.png` pochodzą z tej maszyny.

| Usługa | Wersja | Adres | Dostęp (domyślny szkoleniowy) |
|--------|--------|-------|-------------------------------|
| Grafana OSS | 13.3.0 | http://10.123.2.206:3000 | `admin` / `admin` |
| PostgreSQL | 18 | 10.123.2.206:5432 | baza `grafana_db` (tabele ćwiczeniowe `server_metrics`, `metrics_table`, `sales`, `locations`) |
| MySQL / MariaDB | 11.8 | 10.123.2.206:3306 | baza `lab` (tabela `locations`) |
| InfluxDB + Telegraf | 2.9 / 1.40 | http://10.123.2.206:8086 | org `szkolenie`, bucket `telegraf` (Flux) |
| Prometheus + node_exporter | 3.16 / 1.12 | http://10.123.2.206:9090 | – |
| Elasticsearch | 9.5 | http://10.123.2.206:9200 | indeksy `logstash-*` |
| Zabbix | 7.0 LTS | http://10.123.2.206/zabbix | `Admin` / `zabbix` |
| Mailpit (testowa skrzynka na maile z alertów) | 1.33 | http://10.123.2.206:8025 | – |
| SSH | – | `ssh szkolenie@10.123.2.206` | `szkolenie` / `szkolenie` (sudo) |

W Grafanie przygotowano:

- **źródła danych:** PostgreSQL, InfluxDB (Flux), Prometheus, Elasticsearch, Zabbix (plugin `alexanderzobnin-zabbix-app`), MySQL, TestData,
- **dashboardy:** folder **Szkolenie Grafana** (Źródła danych, Wizualizacje – wszystkie typy, Zmienne), folder **Produkcja** (Serwer Linux – przegląd), folder **Development**, playlistę **Ekran NOC**,
- **alerting:** reguły w folderze **Szkolenie Grafana/Alerty**, contact pointy e-mail (Mailpit), polityki powiadomień i szablon wiadomości,
- **administracja:** użytkowników `jan.kowalski`, `anna.nowak`, `piotr.zielinski`, `ewa.wisniewska` (hasło `Szkolenie123!`), zespoły **Zespół Ops** i **Zespół Dev**, organizację **Klient B**,
- **pluginy dodatkowe:** Zabbix, Clock, Infinity oraz wycofany JSON API (do porównania).

Ćwiczenia praktyczne są na końcu lekcji (sekcje **Ćwiczenie w labie**) oraz w rozdziale **XV. Zadania**.

## 📁 Dodatkowe Zasoby

- **grafiki** - katalog techniczny zawierający grafiki (proszę zignorować)

## 🔧 Zalecane Narzędzia

Do przeglądania materiałów zalecamy użycie [Obsidian](https://obsidian.md/download) - umożliwia wygodną nawigację i podgląd powiązań między tematami.

## 📝 Licencja

Materiały szkoleniowe - wszelkie prawa zastrzeżone.

---

**Powodzenia w nauce! 🚀**
