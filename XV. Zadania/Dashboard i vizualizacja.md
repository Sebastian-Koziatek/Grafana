# Dashboard i Wizualizacja

Zadania wykonujesz w labie: **http://10.123.2.206:3000**, konto `admin` / `admin` (jeśli prowadzący założył Ci własne konto – użyj go).

Źródła danych są już skonfigurowane (**Connections → Data sources**): `Zabbix`, `MySQL`, `PostgreSQL`, `Prometheus`, `InfluxDB`, `Elasticsearch`, `TestData`. Monitorowany host w Zabbixie to **Zabbix server** (grupa **Zabbix servers**) – czyli sama maszyna labowa `vmzabbixsrv16`.

Przed startem utwórz folder na swoje prace: **Dashboards → New → New folder**, nazwa np. `Zadania – <Twoje imię>`. Wszystkie dashboardy z tego pliku zapisuj w tym folderze.

> Historia danych w Zabbixie może być krótka (lab postawiono niedawno). Jeśli wykres jest prawie pusty, ustaw zakres czasu **Last 1 hour** lub **Last 15 minutes**.

---

## Stwórz poniższą konfigurację

### 1. Dashboard z wizualizacją CPU, RAM, dysku i karty sieciowej

Utwórz dashboard **Dashboards → New → New dashboard → Add visualization**, wybierz źródło **Zabbix** i zapisz go jako `Serwer – zasoby (Zabbix)`.

W każdym panelu zapytanie Zabbix ustawiasz tak samo:

| Pole | Wartość |
|------|---------|
| **Query type** | `Metrics` |
| **Group** | `Zabbix servers` |
| **Host** | `Zabbix server` |
| **Item tag** | według tabeli w panelu (pole opcjonalne – zawęża listę itemów) |
| **Item** | według tabeli w panelu |

#### Panel 1: Użycie CPU (Time series)

- **Typ wizualizacji:** Time series
- **Title:** `Użycie CPU`
- **Description:** `Wykorzystanie procesora w procentach (Zabbix, item CPU utilization).`
- **Zapytanie:**
  - **Item tag:** `component: cpu`
  - **Item:** `CPU utilization`
- **Opcje:** **Standard options → Unit** `Percent (0-100)`, **Min** `0`, **Max** `100`; **Thresholds**: 80 (żółty), 90 (czerwony), **Show thresholds** `As lines`.

#### Panel 2: Użycie RAM (Bar chart)

- **Typ wizualizacji:** Bar chart
- **Title:** `Użycie RAM`
- **Description:** `Wykorzystanie pamięci RAM w postaci wykresu słupkowego.`
- **Zapytanie:**
  - **Item tag:** `component: memory`
  - **Item:** `Memory utilization`
- **Opcje:**
  - Bar chart potrzebuje kategorii na osi X. Ustaw krótszy zakres czasu (np. **Last 30 minutes**), a w **Query options** pole **Interval** na `5m`, żeby słupków nie było za dużo.
  - **X Axis** – pole czasu (Time), **Unit** `Percent (0-100)`.

#### Panel 3: Porównanie zasobów (Bar gauge)

- **Typ wizualizacji:** Bar gauge
- **Title:** `CPU / RAM / Dysk`
- **Description:** `Bieżące wykorzystanie CPU, pamięci RAM i dysku / w procentach.`
- **Zapytania** (trzy zapytania A, B, C – przycisk **Add query**):

| Zapytanie | Item tag | Item |
|-----------|----------|------|
| A | `component: cpu` | `CPU utilization` |
| B | `component: memory` | `Memory utilization` |
| C | `component: storage` | `FS [/]: Space: Used, in %` |

- **Opcje:** **Value options → Calculation** `Last *`, **Bar gauge → Orientation** `Horizontal`, **Display mode** `Gradient`, **Unit** `Percent (0-100)`, **Min** `0`, **Max** `100`, **Thresholds** 70 / 90.

#### Panel 4: Karta sieciowa (Time series)

Ruch sieciowy ma inną jednostkę (bity na sekundę) niż procenty z panelu 3, dlatego pokaż go w osobnym panelu.

- **Typ wizualizacji:** Time series
- **Title:** `Sieć – ens18`
- **Zapytania:**

| Zapytanie | Item tag | Item |
|-----------|----------|------|
| A | `component: network` | `Interface ens18: Bits received` |
| B | `component: network` | `Interface ens18: Bits sent` |

- **Opcje:** **Unit** `bits/sec(SI)`; dodaj **Override → Fields with name** `Interface ens18: Bits sent` → **Transform** `Negative Y`, aby ruch wychodzący był rysowany pod osią.

Zapisz dashboard (**Save**).

**Wariant dodatkowy (Prometheus):** zduplikuj dashboard i zamień źródło na **Prometheus** (node_exporter tej maszyny):

| Panel | Zapytanie PromQL |
|-------|------------------|
| CPU % | `100 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100` |
| RAM % | `(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100` |
| Dysk / % | `(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100` |
| Sieć – odbiór | `rate(node_network_receive_bytes_total{device="ens18"}[5m]) * 8` |
| Sieć – wysyłka | `rate(node_network_transmit_bytes_total{device="ens18"}[5m]) * 8` |

Porównaj wyniki z Zabbixa i Prometheusa – czy wartości są zbliżone?

---

### 2. Dashboard z dokumentacją w formie strony Markdown

Ten dashboard zawiera panel tekstowy z dokumentacją monitorowanych zasobów.

#### Panel 1: Dokumentacja

- **Typ wizualizacji:** Text
- **Title:** `Dokumentacja systemu`
- **Konfiguracja:** **Text → Mode** `Markdown`, w polu **Content** wklej:

```markdown
# Dokumentacja systemu vmzabbixsrv16

| Zasób | Źródło | Item / metryka | Próg ostrzeżenia |
|-------|--------|----------------|------------------|
| CPU | Zabbix | `CPU utilization` | 80 % |
| RAM | Zabbix | `Memory utilization` | 80 % |
| Dysk `/` | Zabbix | `FS [/]: Space: Used, in %` | 90 % |
| Sieć | Zabbix | `Interface ens18: Bits received / sent` | – |

## CPU
Średnie wykorzystanie procesora w procentach (item `system.cpu.util`).

## RAM
Procent zajętej pamięci operacyjnej (item `vm.memory.util`).

## Dysk
Procent zajętego miejsca na systemie plików `/`.

## Karta sieciowa
Ruch przychodzący i wychodzący na interfejsie `ens18` w bitach na sekundę.

---
Dashboard z wykresami: [Serwer – zasoby (Zabbix)](/dashboards)
```

- Dodaj obok drugi panel **Text** w trybie `HTML` z krótką informacją o dyżurach (np. tabela z nazwiskami i telefonami – dane fikcyjne).
- Zapisz dashboard jako `Dokumentacja serwera`.

**Rozszerzenie:** w panelu Markdown użyj zmiennej dashboardu. Dodaj zmienną typu **Constant** `serwer` = `vmzabbixsrv16` i w treści wpisz `Serwer: ${serwer}` – sprawdź, że Grafana podstawia wartość.

---

### 3. Dashboard z geolokalizacją (MySQL)

Baza MySQL/MariaDB `lab` zawiera tabelę `locations` z 10 serwerowniami w Polsce (kolumny `city`, `latitude`, `longitude`, `servers`, `status`, `load_pct`). Źródło danych w Grafanie: **MySQL**.

1. Nowy dashboard → **Add visualization** → źródło **MySQL**.
2. W edytorze zapytania przełącz **Builder** na **Code**, ustaw **Format** `Table` i wpisz:
   ```sql
   SELECT city, latitude, longitude, servers, status, load_pct
   FROM locations
   ```
3. Kliknij **Run query** i włącz **Table view** – sprawdź, że widzisz 10 miast.
4. Zmień wizualizację na **Geomap**.
5. **Map view → Initial view:** **View** `Coordinates`, **Latitude** `52`, **Longitude** `19.4`, **Zoom** `6` – mapa wyśrodkuje się na Polsce.
6. Warstwa danych (**Markers**):
   - **Location mode** `Coords`, **Latitude field** `latitude`, **Longitude field** `longitude`,
   - **Size** – pole `servers` (większa serwerownia = większy znacznik),
   - **Color** – pole `load_pct`, w **Thresholds** ustaw 65 (pomarańczowy) i 85 (czerwony),
   - **Symbol** – wybierz ikonę **plane** (samolot),
   - **Text label** – pole `city`.
7. Tytuł panelu: `Serwerownie – obciążenie`. Zapisz dashboard jako `Geolokalizacja (MySQL)`.

**Pytania:** która serwerownia ma najwyższe obciążenie? Dlaczego kolor znacznika zależy od `load_pct`, a nie od kolumny `status`?

---

## Wskazówki

- Gotowy przykład geomapy (na danych z PostgreSQL) znajdziesz w dashboardzie **Szkolenie Grafana → Wizualizacje → Wizualizacje – wszystkie typy**, a przykład zapytań Zabbix i MySQL – w **Szkolenie Grafana → Źródła danych → Źródła danych – przegląd**. Możesz je podglądać, ale nie zapisuj w nich zmian.
- Jeśli lista **Item** w Zabbixie jest pusta – wyczyść pole **Item tag** albo sprawdź **Group** i **Host**.
- Panel edytujesz przez menu panelu → **Edit**; do dashboardu wracasz przyciskiem **Back**.
