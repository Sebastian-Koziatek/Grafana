# Zadania z Grafany

Zadania wykonujesz w labie: **http://10.123.2.206:3000** (`admin` / `admin`). Pomocnicze adresy:

| Usługa | Adres | Dostęp |
|--------|-------|--------|
| Grafana | http://10.123.2.206:3000 | `admin` / `admin` |
| Zabbix (frontend) | http://10.123.2.206/zabbix | `Admin` / `zabbix` |
| Prometheus | http://10.123.2.206:9090 | – |
| Mailpit (skrzynka testowa na maile z alertów) | http://10.123.2.206:8025 | – |
| SSH | `ssh szkolenie@10.123.2.206` | `szkolenie` / `szkolenie` |

Zasady:

- Wszystko, co tworzysz, nazywaj z dopiskiem swojego imienia (np. `Zabbix – Anna`, `Zadania – Anna`) – lab może być wspólny dla kilku osób.
- Nie usuwaj ani nie zmieniaj obiektów przygotowanych do szkolenia (foldery **Szkolenie Grafana**, **Produkcja**, **Development**, źródła danych, reguły w folderze **Alerty**, użytkownicy labu).

---

## 1. Źródło danych Zabbix – dodanie od nowa

Istniejące źródło **Zabbix** jest używane przez dashboardy labu, więc go nie kasuj – utworzysz drugie, własne.

1. W Zabbixie (http://10.123.2.206/zabbix, `Admin` / `zabbix`) utwórz token API: **User settings → API tokens → Create API token**, nazwa `grafana-<imię>`. Skopiuj token (wyświetla się tylko raz).
2. W Grafanie: **Connections → Add new connection** → wyszukaj **Zabbix** → **Add new data source**.
3. Ustaw:
   - **Name:** `Zabbix – <imię>`,
   - **URL:** `http://localhost/zabbix/api_jsonrpc.php`,
   - **Auth type:** `API token`, wklej token.
4. **Save & test** – wynik powinien być zielony.
5. Odpowiedz na pytania:
   - Jak działa pobieranie danych z maszyny wirtualnej? (agent Zabbix → serwer Zabbix)
   - Gdzie te dane są przechowywane? (baza Zabbixa – w labie MariaDB, baza `zabbix`)
   - Jak dane są przekazywane do Grafany? (Grafana odpytuje API Zabbixa – JSON-RPC – przy każdym odświeżeniu panelu)
   - Jaki plugin należy zainstalować, aby te dane dodać do źródeł danych w Grafanie i co trzeba zrobić po instalacji? (`alexanderzobnin-zabbix-app` + **Enable** na stronie pluginu)
6. Na koniec usuń swoje źródło `Zabbix – <imię>` (**Connections → Data sources** → źródło → **Delete**) i token w Zabbixie.

---

## 2. Folder na zadania i dashboard z dwiema tabelami

1. Utwórz folder **Dashboards → New → New folder** `Zadania – <imię>`.
2. Utwórz w nim dashboard `Zasoby – tabele` z dwoma panelami **Table** (źródło **Zabbix**, **Group** `Zabbix servers`, **Host** `Zabbix server`):
   1. **Pierwsza tabela – „Zasoby”:** zapytania A/B/C z itemami
      - `CPU utilization`,
      - `Memory utilization`,
      - `FS [/]: Space: Used, in %`.
   2. **Druga tabela – „Sieć”:** zapytania A/B z itemami
      - `Interface ens18: Bits received` (ruch przychodzący),
      - `Interface ens18: Bits sent` (ruch wychodzący).
3. Aby każda metryka była jednym wierszem z ostatnią wartością, dodaj w zakładce **Transformations** transformację **Reduce** (**Mode** `Series to rows`, **Calculations** `Last *`, opcjonalnie `Max` i `Mean`).
4. Ustaw jednostki: w pierwszej tabeli `Percent (0-100)`, w drugiej `bits/sec(SI)`.

---

## 3. Dashboard z danymi z MySQL (geolokalizacja)

- Stwórz dashboard, który pokaże dane z MySQL (baza `lab`, tabela `locations`) – lokalizacje 10 serwerowni w Polsce.
- Zastosuj znacznik punktowy z ikoną samolotu (**Symbol** `plane`) i wyśrodkuj mapę na Polsce.
- Szczegółowa instrukcja: plik **Dashboard i vizualizacja**, zadanie 3.

---

## 4. Testowy dashboard Markdown

- Stwórz dashboard z panelem **Text** w trybie **Markdown**, zawierający: nagłówki, listę, tabelę, blok kodu i link do innego dashboardu.
- Szczegółowa instrukcja: plik **Dashboard i vizualizacja**, zadanie 2.

---

## 5. Utworzenie playlisty

1. **Dashboards → Playlists → New playlist**, nazwa `Playlista – <imię>`.
2. **Interval:** `5m`.
3. Dodaj swoje dashboardy z zadań 2–4 (**Add by title**) i opcjonalnie dashboardy z tagiem (**Add by tag**).
4. Zapisz i kliknij **Start playlist** – wypróbuj tryb **Normal** i **TV (Kiosk)**. Tryb kiosk opuszczasz klawiszem **Esc**.

---

## 6. Snapshot dashboardu

1. Otwórz dashboard `Zasoby – tabele`.
2. Rozwiń menu przy przycisku udostępniania (ikona *share* obok **Edit**) i wybierz opcję snapshotu (**Share snapshot**).
3. Nadaj nazwę, ustaw czas wygaśnięcia i opublikuj snapshot lokalnie.
4. Otwórz link snapshotu w oknie incognito. Sprawdź listę w **Dashboards → Snapshots**.
5. Pytanie: czym snapshot różni się od zwykłego linku do dashboardu? (Snapshot zawiera zapisane dane z chwili wykonania, bez dostępu do źródła danych.)

---

## 7. Przegląd metryk w Explore

1. Otwórz **Explore**, wybierz źródło **Zabbix**.
2. Wybierz **Group** `Zabbix servers`, **Host** `Zabbix server` i przejrzyj dostępne itemy (filtr **Item tag**, np. `component: cpu`, `component: memory`, `component: network`, `component: storage`).
3. Przełącz źródło na **Prometheus** i wpisz `node_load1` – porównaj z itemem Zabbixa `Load average (1m avg)`.

---

## 8. Metryki Prometheusa w Drilldown

1. Otwórz menu **Drilldown** i wybierz przeglądanie metryk (**Metrics**), źródło **Prometheus**.
2. Wyszukaj metryki zaczynające się od `node_` (node_exporter tej maszyny) i otwórz np. `node_cpu_seconds_total`.
3. Sprawdź podział według etykiet (`mode`, `cpu`) i dodaj wybrany wykres do nowego dashboardu.
4. Dla porównania: w Prometheusie (http://10.123.2.206:9090) sprawdź **Status → Target health** – jakie joby są monitorowane?

---

## 9. Contact point – e-mail (Mailpit), opcjonalnie Discord

Grafana w labie wysyła pocztę przez lokalny serwer testowy **Mailpit** – maile nie wychodzą do internetu, a odczytasz je na http://10.123.2.206:8025.

1. **Alerting → Notification configuration** → zakładka **Contact points** → **New contact point**.
2. **Name:** `E-mail – <imię>`, **Integration:** `Email`, **Addresses:** `<imię>@szkolenie.lab`.
3. Kliknij **Test** – w Mailpit powinna pojawić się wiadomość testowa.
4. Zapisz: **Save contact point**.

**Opcja – Discord:** jeśli masz własny serwer Discord, utwórz webhook (**Ustawienia kanału → Integracje → Webhooki → Nowy webhook → Kopiuj URL**) i dodaj contact point z integracją **Discord**, wklejając URL w pole **Webhook URL**. Nie używaj webhooków cudzych serwerów.

---

## 10. Reguła alertu na użycie CPU

1. **Alerting → Alert rules → New alert rule**.
2. **1. Enter alert rule name:** `CPU – <imię>`.
3. **2. Define query and alert condition:** źródło **Prometheus**, przełącz edytor z **Builder** na **Code** i wpisz:
   ```promql
   100 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100
   ```
   W sekcji **Alert condition** ustaw **WHEN QUERY IS ABOVE** `5` (niski próg, żeby alert odpalił się na szkoleniu; na produkcji np. 80). Kliknij **Preview alert rule condition**.
   Przełącznik **Advanced options** pokazuje ten sam warunek jako wyrażenia **Reduce** i **Threshold** (lekcja VI.2).
4. **3. Add folder and labels:** folder `Zadania – <imię>`.
5. **4. Set evaluation behavior:** **New evaluation group** (np. `zadania-<imię>`, interwał `1m`), **Pending period** `1m`.
6. **5. Configure notifications:** **Contact point** `E-mail – <imię>`.
7. **Save**. Obserwuj stany **Normal → Pending → Firing** na liście reguł i sprawdź maila w Mailpit.
8. Na koniec zmień próg na `80` i sprawdź, że alert wraca do **Normal** (przychodzi mail o rozwiązaniu – *Resolved*).

---

## 11. Etykiety alertu

- W regule `CPU – <imię>` (sekcja **3. Add folder and labels → Add labels**) dodaj etykiety `severity=warning` i `team=<imię>`.
- Sprawdź w **Alerting → Notification configuration → Notification policies**, do której polityki trafia alert (podpowiedź: polityka z `severity=critical` kieruje do zespołu Ops – co się stanie, gdy zmienisz etykietę na `critical`?).

---

## 12. Notification template

1. **Alerting → Notification configuration** → zakładka **Templates** → dodaj nowy szablon (grupę szablonów) o nazwie `szablon-<imię>`.
2. Zdefiniuj szablony w języku **Go template**, np.:
   ```go
   {{ define "imie.tytul" }}[{{ .Status | toUpper }}] {{ .CommonLabels.alertname }} ({{ len .Alerts }}){{ end }}

   {{ define "imie.tresc" }}
   {{ range .Alerts }}
   Alert: {{ .Labels.alertname }}
   Stan: {{ .Status }}
   Wartość: {{ .ValueString }}
   Etykiety: {{ range .Labels.SortedPairs }}{{ .Name }}={{ .Value }} {{ end }}
   Od: {{ .StartsAt.Format "2006-01-02 15:04:05" }}
   {{ end }}
   {{ end }}
   ```
3. Podejrzyj wynik w podglądzie edytora szablonu i zapisz.
4. Wzór: istniejący szablon **szkolenie** (definiuje `szkolenie.tytul` i `szkolenie.tresc`).

---

## 13. Dodanie szablonu do powiadomienia

1. Edytuj contact point `E-mail – <imię>` (**Edit**) → rozwiń **Optional Email settings**.
2. **Subject:** `{{ template "imie.tytul" . }}`, **Message:** `{{ template "imie.tresc" . }}`.
3. Zapisz (**Save contact point**), kliknij **Test** i porównaj wiadomość w Mailpit z wcześniejszą. Poczekaj też na prawdziwy alert z reguły `CPU – <imię>`.

---

## 14. Eksport i import dashboardu

1. Otwórz swój dashboard → **Export → Export as code** → format **JSON** → **Download file**.
2. Zaimportuj plik: **Dashboards → New → Import** → wgraj plik, zmień nazwę na `Kopia – <imię>` i nowy **UID** (albo pozostaw pusty) → **Import**.
3. Odpowiedz na pytania:
   - W jaki sposób Grafana przechowuje definicje dashboardów? (model JSON w bazie Grafany – w labie SQLite `/var/lib/grafana/grafana.db`)
   - Jak zaimportować dashboard do innej instancji Grafany? (eksport z opcją *Share dashboard with another instance*, import przez **Dashboards → New → Import**, wybór źródeł danych w nowej instancji; alternatywnie ID z grafana.com, np. `1860` – Node Exporter Full)

---

## 15. Użytkownicy, zespoły i organizacje

Wykonuj jako `admin`. Konta labu (`jan.kowalski`, `anna.nowak`, `piotr.zielinski`, `ewa.wisniewska`) mają hasło `Szkolenie123!` – możesz się nimi logować w oknie incognito, ale ich nie usuwaj.

- **Dodanie nowego użytkownika:**
  - **Administration → Users and access → Users → New user**: `imie.test`, hasło `Szkolenie123!`. Sprawdź, jaką rolę dostał w **Main Org.** (domyślnie Viewer).
  - Dodaj go do zespołu **Zespół Ops** (**Teams → Zespół Ops → Add member**, uprawnienie **Member**).
  - Przejrzyj stronę użytkownika: sekcje **User information**, **Permissions** (Grafana Admin), **Organizations**, **Sessions**.

- **Tworzenie zespołu:**
  - Utwórz zespół `Monitoring – <imię>` i dodaj do niego `imie.test` oraz `piotr.zielinski`.
  - Nadaj zespołowi uprawnienie **Edit** do swojego folderu `Zadania – <imię>` (**Folder actions → Manage permissions → Add a permission**).
  - W **Settings** zespołu ustaw **Home Dashboard** na swój dashboard z zadania 2.

- **Zmiana ról w organizacji:**
  - Zaloguj się jako `imie.test` – czy możesz edytować dashboard **Serwer Linux – przegląd** w folderze **Produkcja**? (Podpowiedź: Zespół Ops ma tam **Edit**.)
  - Usuń `imie.test` z Zespołu Ops i sprawdź ponownie. Potem zmień jego rolę z **Viewer** na **Editor** (**Change role**) – co się zmieniło w folderze **Produkcja**, a co w folderze **Development**? Wyjaśnij na podstawie tabeli uprawnień z lekcji VII.2.

- **Druga organizacja:**
  - Utwórz organizację `Org – <imię>` (**Administration → General → Organizations → New org**).
  - Dodaj do niej `imie.test` z rolą **Admin** (strona użytkownika → **Add user to organization**).
  - Przełącz się między organizacjami (przełącznik w lewym górnym rogu) i porównaj: dashboardy, źródła danych, reguły alertów.

- **Sprzątanie:** usuń swoją organizację, zespół `Monitoring – <imię>` i użytkownika `imie.test`.
