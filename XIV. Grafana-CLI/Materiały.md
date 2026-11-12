![[Pasted image 20260608081708.png]]

Grafana CLI (`grafana cli`) to narzędzie wiersza poleceń instalowane razem z Grafaną. Służy do zarządzania pluginami i do operacji administracyjnych (np. reset hasła administratora) – bez interfejsu webowego. Wszystkie wyniki poniżej pochodzą z maszyny labowej (Grafana 13.3.0 na Ubuntu 26.04).

## `grafana cli` zamiast `grafana-cli`

Od Grafany 12 wszystkie narzędzia są podkomendami jednego programu `grafana`:

| Stara forma (przestarzała) | Nowa forma |
|----------------------------|------------|
| `grafana-cli ...` | `grafana cli ...` |
| `grafana-server ...` | `grafana server ...` |

Stare polecenia nadal działają, ale wypisują ostrzeżenie:

```text
$ sudo grafana-cli -v
Deprecation warning: 'grafana-cli' is deprecated and will be removed in a future release. Use the 'grafana cli' subcommand instead.
grafana version 13.3.0

$ grafana-server -v
Deprecation warning: 'grafana-server' is deprecated and will be removed in a future release. Use the 'grafana server' subcommand instead.
Version 13.3.0 (commit: 6f8ce3c693b493043ebcd4787566d2b6b118b0f0, branch: release-13.3.0#patched)
```

Nowa forma:

```text
$ grafana cli -v
grafana version 13.3.0
```

> Usługa systemd nadal nazywa się **`grafana-server`** (`sudo systemctl restart grafana-server`) – zmieniła się tylko nazwa polecenia.

W pakiecie `.deb` / `.rpm` polecenie `/usr/sbin/grafana` jest skryptem, który przy `grafana cli` i `grafana server` sam dokłada ścieżki pakietu: `--homepath=/usr/share/grafana`, `--config=/etc/grafana/grafana.ini` i `--pluginsDir=/var/lib/grafana/plugins`. Dzięki temu nie trzeba ich podawać ręcznie. Polecenia zmieniające pliki (pluginy, baza) uruchamiaj przez `sudo`.

---

## Struktura polecenia

```bash
grafana cli [opcje globalne] <grupa> <komenda> [argumenty]
```

Pomoc (`grafana cli --help`):

```text
NAME:
   grafana cli - run the grafana cli

USAGE:
   grafana cli [command options]

COMMANDS:
   write-openapi  write-openapi <manifest.json> [--api-version <version>] -o <path>
   plugins        Manage plugins for grafana
   admin          Grafana admin commands
   help, h        Shows a list of commands or help for one command
```

| Grupa | Opis |
|-------|------|
| `plugins` | zarządzanie pluginami |
| `admin` | operacje administracyjne (hasło admina, migracje bazy, sekrety) |

Pomoc dla każdej grupy i komendy: `grafana cli plugins --help`, `grafana cli admin reset-admin-password --help`.

---

## Zarządzanie pluginami

Komendy grupy `plugins` (`grafana cli plugins --help`):

```text
COMMANDS:
   install                  install <plugin id> <plugin version (optional)>
   list-remote              list remote available plugins
   list-versions            list-versions <plugin id>
   update, upgrade          update <plugin id>
   update-all, upgrade-all  update all your installed plugins
   ls                       list installed plugins (excludes core plugins)
   uninstall, remove        uninstall <plugin id>
   help, h                  Shows a list of commands or help for one command
```

> W starszych materiałach pojawia się `plugins list-available` – w Grafanie 13 taka komenda nie istnieje, właściwa nazwa to **`list-remote`**.

### Zainstalowane pluginy

```bash
sudo grafana cli plugins ls
```

```text
installed plugins:

grafana-advisor-app @ 1.1.3
grafana-exploretraces-app @ 3.2.0
zipkin @ 12.4.12
cloudwatch @ 12.11.2
grafana-pyroscope-datasource @ 13.0.10
yesoreyeram-infinity-datasource @ 4.1.6
grafana-metricsdrilldown-app @ 3.1.0
mssql @ 13.0.10
jaeger @ 13.0.8
prometheus @ 13.2.5
elasticsearch @ 12.9.6
loki @ 13.2.5
tempo @ 13.4.2
grafana-clock-panel @ 3.2.5
opentsdb @ 13.0.8
grafana-postgresql-datasource @ 13.0.7
alexanderzobnin-zabbix-app @ 6.10.3
grafana-lokiexplore-app @ 3.1.0
grafana-pyroscope-app @ 3.1.0
influxdb @ 13.1.10
marcusolsson-json-datasource @ 1.4.4
```

Lista nie obejmuje pluginów wkompilowanych w Grafanę (np. panele Time series, Stat, źródło MySQL). Widoczne są natomiast źródła danych, które Grafana 13 dostarcza jako osobne pluginy i doinstalowuje przy pierwszym starcie (`prometheus`, `influxdb`, `loki`…).

### Pluginy dostępne w katalogu

```bash
sudo grafana cli plugins list-remote | grep -i -E "clock|zabbix|infinity"
```

```text
id: alexanderzobnin-zabbix-app version: 6.10.3
id: dzaczek-alpineclock-panel version: 1.5.4
id: grafana-clock-panel version: 3.2.5
id: yesoreyeram-infinity-datasource version: 4.1.6
```

Komenda wypisuje też na początku linie logów (`logger=settings ...`) – to normalne; filtruj wynik `grep`em.

### Dostępne wersje pluginu

```bash
sudo grafana cli plugins list-versions alexanderzobnin-zabbix-app | head -5
```

```text
6.10.3
6.10.2
6.10.1
6.10.0
6.9.1
```

### Instalacja pluginu

```bash
sudo grafana cli plugins install <plugin-id>
sudo systemctl restart grafana-server
```

Przykłady (tak instalowano pluginy w labie):

```bash
# Zabbix (aplikacja: źródło danych + panel)
sudo grafana cli plugins install alexanderzobnin-zabbix-app

# panel zegara
sudo grafana cli plugins install grafana-clock-panel

# Infinity – JSON, CSV, XML, REST API
sudo grafana cli plugins install yesoreyeram-infinity-datasource

sudo systemctl restart grafana-server
```

> Po każdej instalacji, aktualizacji lub usunięciu pluginu przez CLI zrestartuj usługę: `sudo systemctl restart grafana-server`.

### Instalacja konkretnej wersji

```bash
sudo grafana cli plugins install alexanderzobnin-zabbix-app 6.10.2
```

### Instalacja z pliku lub własnego URL (air-gapped)

Opcja `--pluginUrl` wskazuje plik ZIP zamiast pobierania z grafana.com:

```bash
# z serwera WWW w sieci firmowej
sudo grafana cli \
  --pluginUrl https://repo.firma.local/grafana/moj-panel-1.0.0.zip \
  plugins install moja-firma-panel

# z pliku lokalnego
sudo grafana cli \
  --pluginUrl file:///tmp/alexanderzobnin-zabbix-app-6.10.3.linux_amd64.zip \
  plugins install alexanderzobnin-zabbix-app
```

Plik ZIP pobierasz ze strony pluginu w katalogu grafana.com na komputerze z dostępem do internetu.

### Aktualizacja pluginów

```bash
# jeden plugin
sudo grafana cli plugins update alexanderzobnin-zabbix-app

# wszystkie zainstalowane
sudo grafana cli plugins update-all

sudo systemctl restart grafana-server
```

### Usuwanie pluginu

```bash
sudo grafana cli plugins remove grafana-clock-panel     # alias: uninstall
sudo systemctl restart grafana-server
```

---

## Komendy administracyjne

Grupa `admin` (`grafana cli admin --help`):

```text
COMMANDS:
   reset-admin-password        reset-admin-password <new password>
   data-migration              Runs a script that migrates or cleanups data in your database
   secrets-migration           Runs a script that migrates secrets in your database
   secrets-consolidation       Runs an operation that re-encrypts all encrypted values in your database with new data keys
   flush-rbac-seed-assignment  Clears RBAC seeding to force re-seeding on next startup. Use after running an Enterprise build, then an OSS build, then an Enterprise build again.
   db-migrate                  Run schema migrations against the database configured in --config.
   resource-db-migrate         Run the unified storage resource schema migrations against the database configured in --config.
```

Na co dzień potrzebna jest głównie pierwsza z nich.

### Reset hasła administratora

```bash
sudo grafana cli admin reset-admin-password <nowe-haslo>
```

Opcje (`grafana cli admin reset-admin-password --help`):

```text
OPTIONS:
   --password-from-stdin  Read the password from stdin (default: false)
   --user-id value        The admin user's ID (default: 1)
   --help, -h             show help
```

- `--password-from-stdin` – hasło z wejścia standardowego, więc nie zostaje w historii powłoki:
  ```bash
  echo -n 'NoweHaslo123!' | sudo grafana cli admin reset-admin-password --password-from-stdin
  ```
- `--user-id` – reset hasła innego konta administratora (domyślnie ID 1 = `admin`).

Komenda zmienia hasło bezpośrednio w bazie Grafany (domyślnie SQLite `/var/lib/grafana/grafana.db`), więc działa także wtedy, gdy nikt nie pamięta hasła do UI.

> W labie **nie resetuj** hasła konta `admin` – korzystają z niego wszyscy uczestnicy i skrypty labu.

### Wersja Grafany

```bash
grafana -v              # grafana version 13.3.0
grafana cli -v          # grafana version 13.3.0
curl -s http://localhost:3000/api/health
```

```json
{
  "database": "ok",
  "version": "13.3.0",
  "commit": "6f8ce3c693b493043ebcd4787566d2b6b118b0f0"
}
```

---

## Opcje globalne

Z `grafana cli --help`:

| Opcja | Opis |
|-------|------|
| `--config <plik>` | ścieżka do pliku konfiguracyjnego (pakiet: `/etc/grafana/grafana.ini`) |
| `--homepath <ścieżka>` | katalog instalacji Grafany (pakiet: `/usr/share/grafana`) |
| `--configOverrides <wartości>` | nadpisanie ustawień, np. `cfg:default.paths.log=/dev/null` |
| `--pluginsDir <ścieżka>` | katalog pluginów (domyślnie `/var/lib/grafana/plugins`, zmienna `GF_PLUGIN_DIR`) |
| `--repo <url>` | repozytorium pluginów (domyślnie `https://grafana.com/api/plugins`, zmienna `GF_PLUGIN_REPO`) |
| `--pluginUrl <url>` | pełny adres pliku ZIP pluginu zamiast pobierania z grafana.com (zmienna `GF_PLUGIN_URL`) |
| `--insecure` | pominięcie weryfikacji certyfikatu TLS |
| `--debug` | szczegółowe logi |
| `--version`, `-v` | wersja |

Opcje globalne podaje się **przed** grupą komend:

```bash
sudo grafana cli --pluginsDir /opt/grafana-plugins plugins install grafana-clock-panel
```

---

## Typowe scenariusze

### Instalacja i aktywacja pluginu Zabbix

```bash
# 1. instalacja
sudo grafana cli plugins install alexanderzobnin-zabbix-app

# 2. restart
sudo systemctl restart grafana-server

# 3. kontrola usługi i pluginu
systemctl status grafana-server --no-pager | head -12
sudo grafana cli plugins ls | grep zabbix

# 4. w UI: Administration → Plugins and data → Plugins → Zabbix → Enable
# 5. w UI: Connections → Add new connection → Zabbix
```

W wyniku `systemctl status` widać, że plugin z backendem działa jako osobny proces:

```text
     CGroup: /system.slice/grafana-server.service
             ├─22879 /usr/share/grafana/bin/grafana server --config=/etc/grafana/grafana.ini ...
             ├─22888 /var/lib/grafana/plugins/alexanderzobnin-zabbix-app/datasource/gpx_zabbix-datasource_linux_amd64
```

### Aktualizacja wszystkich pluginów

```bash
sudo grafana cli plugins update-all
sudo systemctl restart grafana-server
sudo journalctl -u grafana-server -n 50 --no-pager
```

### Dokumentacja zainstalowanych pluginów

```bash
sudo grafana cli plugins ls > zainstalowane_pluginy.txt
cat zainstalowane_pluginy.txt
```

---

## Lokalizacja plików Grafany (pakiet deb/rpm)

| Plik / katalog | Opis |
|----------------|------|
| `/etc/grafana/grafana.ini` | główny plik konfiguracyjny |
| `/etc/grafana/ldap.toml` | konfiguracja LDAP |
| `/etc/grafana/provisioning/` | provisioning (źródła danych, dashboardy, alerty) |
| `/var/lib/grafana/grafana.db` | baza danych SQLite (domyślna) |
| `/var/lib/grafana/plugins/` | zainstalowane pluginy |
| `/var/log/grafana/grafana.log` | logi serwera |
| `/usr/share/grafana/` | program (`bin/grafana`), frontend, `conf/defaults.ini` |
| `/usr/sbin/grafana` | skrypt uruchamiający z dołożonymi ścieżkami pakietu |

---

## Dobre praktyki

- Używaj formy **`grafana cli`** – `grafana-cli` zniknie w przyszłych wersjach.
- Po zmianach pluginów przez CLI restartuj `grafana-server`.
- Sprawdzaj kompatybilność (`list-versions`, pole **Dependencies** na stronie pluginu) przed aktualizacją.
- Na produkcji testuj nowe wersje pluginów najpierw na środowisku testowym.
- W sieciach bez internetu dystrybuuj pluginy przez `--pluginUrl` (plik lub wewnętrzny serwer WWW).
- Przechowuj listę pluginów (`grafana cli plugins ls`) w repozytorium.

---

## Podsumowanie

`grafana cli` to narzędzie do zarządzania pluginami (`plugins install`, `ls`, `list-remote`, `list-versions`, `update-all`, `remove`) i do administracji (`admin reset-admin-password`). Stara forma `grafana-cli` działa z ostrzeżeniem o wycofaniu. Po każdej zmianie pluginów przez CLI wymagany jest restart usługi `grafana-server`.

---

## Ćwiczenie w labie

SSH: `ssh szkolenie@10.123.2.206` (hasło `szkolenie`).

1. Porównaj `sudo grafana-cli -v` i `sudo grafana cli -v`.
2. Wyświetl pomoc: `grafana cli --help`, `grafana cli plugins --help`, `grafana cli admin --help`.
3. Wylistuj pluginy zainstalowane i znajdź w katalogu pluginy z „clock” w nazwie (`list-remote | grep -i clock`).
4. Sprawdź dostępne wersje pluginu Infinity (`list-versions yesoreyeram-infinity-datasource`).
5. Zainstaluj `dzaczek-alpineclock-panel`, zrestartuj Grafanę i sprawdź go w UI (**Administration → Plugins and data → Plugins → State → Installed**). Następnie usuń go (`plugins remove`) i ponownie zrestartuj usługę.
6. Odczytaj (bez wykonywania!) pomoc `reset-admin-password` i odpowiedz: jak zresetować hasło tak, aby nie zostało w historii powłoki?
