# Cachet 3.x – KI-Dokumentation (README_AI.md)

> Diese Datei beschreibt die Architektur und den Code dieses Repositories für KI-Assistenten und Entwickler.
> Stand: Branch `3.x`, Repository [`cachethq/cachet`](https://github.com/cachethq/cachet).

## Kurzüberblick

**Cachet** ist ein selbst gehostetes Open-Source-Statusseiten-System. Version 3.x ist ein **Neuaufbau** auf Laravel 11. Die meiste Geschäftslogik steckt **nicht** in diesem Repository, sondern im Composer-Paket [`cachethq/core`](https://github.com/cachethq/core).

Dieses Repo ist die **Laravel-Hülle (Application Shell)**:

- Bootstrap, Konfiguration, Umgebungsvariablen
- Anpassbare `User`-Klasse
- Laravel-Standard-Migrationen (Users, Sessions, Cache, Jobs, Sanctum)
- Minimale eigene Routen (fast alles kommt aus `cachethq/core`)

**Status:** 3.x ist in Entwicklung und noch **nicht vollständig produktionsreif**.  
Offizielle Doku: https://docs.cachethq.io/v3.x/

**Installiertes Core-Paket:** `cachethq/core` `3.x-dev` (Commit `d30447a`, Pfad: `vendor/cachethq/core/`)

---

## Architektur

```
┌─────────────────────────────────────────────────────────┐
│  cachet (dieses Repo) – Laravel Application Shell       │
│  ├── app/Models/User.php      (erweitert Core-User)     │
│  ├── config/cachet.php        (Cachet-Konfiguration)    │
│  ├── bootstrap/providers.php  (Service Provider)          │
│  └── database/migrations/     (Laravel-Basis-Tabellen)  │
└───────────────────────────┬─────────────────────────────┘
                            │ composer require
                            ▼
┌─────────────────────────────────────────────────────────┐
│  cachethq/core – Kernfunktionalität                     │
│  ├── Statusseite (öffentlich)                           │
│  ├── Filament-Dashboard (/dashboard)                    │
│  ├── REST-API ({CACHET_PATH}/api/...)                   │
│  ├── Models, Events, Webhooks, Commands                 │
│  └── Eigene Migrationen, Views, Assets                  │
└─────────────────────────────────────────────────────────┘
```

### Service Provider (Bootstrap)

In `bootstrap/providers.php` werden drei Provider registriert:

| Provider | Rolle |
|----------|-------|
| `App\Providers\AppServiceProvider` | App-spezifisch (Rate Limiting) |
| `Cachet\CachetCoreServiceProvider` | API, Statusseite, Commands, Webhooks |
| `Cachet\CachetDashboardServiceProvider` | Filament-Admin-Panel |

Das Core-Paket registriert u.a.:

- Routen unter `{CACHET_PATH}` (Statusseite) und `{CACHET_PATH}/api` (JSON-API)
- Middleware-Gruppen `cachet` und `cachet:api`
- Migrationen, Views, Übersetzungen, Artisan-Commands
- Webhook-Listener für Incidents, Components, Subscribers, Metrics

---

## Tech-Stack

| Schicht | Technologie |
|---------|-------------|
| Backend | PHP 8.2+, Laravel 11 |
| Admin-UI | Filament 4.11 (aus `cachethq/core`) |
| Frontend Statusseite | Tailwind CSS, Alpine.js, Twig |
| API | JSON:API (`timacdonald/json-api`), Laravel Sanctum |
| Daten | MariaDB, MySQL, PostgreSQL oder SQLite |
| Queue / Cache / Session | Standard Laravel (Default: Database) |
| Monitoring | Laravel Nightwatch (optional) |
| Storage | Local oder AWS S3 (`league/flysystem-aws-s3-v3`) |

---

## Verzeichnisstruktur (dieses Repo)

```
cachet/
├── app/
│   ├── Http/
│   │   ├── Controllers/Controller.php   # Laravel-Basiscontroller
│   │   └── Middleware/TrustProxies.php  # Proxy-Konfiguration via CACHET_TRUSTED_PROXIES
│   ├── Models/User.php                  # Erweitert Cachet\Models\User + FilamentUser
│   └── Providers/AppServiceProvider.php # HOME=/home, API-Rate-Limit
├── bootstrap/
│   ├── app.php                          # Laravel 11 Bootstrap (Health: /up)
│   └── providers.php                    # Service-Provider-Liste
├── config/
│   ├── cachet.php                       # Zentrale Cachet-Konfiguration (wichtig!)
│   └── …                                # Standard-Laravel-Configs
├── database/
│   ├── migrations/                      # Laravel-Basis-Tabellen
│   ├── seeders/DatabaseSeeder.php       # Demo-User test@test.com / test123
│   └── factories/UserFactory.php
├── public/index.php                     # Front Controller
├── routes/
│   ├── web.php                          # Leer – Routen kommen aus Core
│   └── console.php
├── composer.json                        # Abhängigkeit: cachethq/core (dev-main)
└── .env.example                         # Umgebungsvariablen-Vorlage
```

**Wichtig:** Nach `composer install` liegt die eigentliche Anwendung unter `vendor/cachethq/core/`.

---

## Installiertes Paket `vendor/cachethq/core`

### Verzeichnisstruktur (Core)

```
vendor/cachethq/core/
├── config/cachet.php              # Default-Config (wird nach config/ publiziert)
├── database/
│   ├── migrations/              # ~70 Migrationen (Domain + Legacy-Tabellen)
│   ├── seeders/DatabaseSeeder.php  # Demo-Daten (Components, Incidents, …)
│   └── factories/
├── public/build/                  # Kompilierte Vite-Assets (Dashboard + Statusseite)
├── resources/
│   ├── views/status-page/         # Twig/Blade-Templates der Statusseite
│   ├── views/filament/            # Dashboard-Views
│   ├── lang/                      # Übersetzungen (de, en, fr, …)
│   ├── css/dashboard/             # Filament-Theme (Vite)
│   └── svg/                       # Cachet-Icons (BladeUI)
├── routes/api.php                 # JSON-API-Routen
└── src/
    ├── Actions/                   # Command-Pattern (CreateIncident, …)
    ├── Commands/                  # Artisan-Commands
    ├── Data/                      # Spatie Laravel Data DTOs
    ├── Enums/                     # Status-Enums (Component, Incident, …)
    ├── Events/                    # Domain-Events (Incidents, Components, …)
    ├── Filament/                  # Dashboard (Resources, Pages, Widgets)
    ├── Http/Controllers/          # API, Statusseite, Setup, Auth
    ├── Jobs/                      # SendBeaconJob
    ├── Listeners/                 # Webhook-Listener
    ├── Models/                    # Eloquent-Modelle
    ├── Renderers/                 # Statusseiten-Renderer
    ├── Settings/                  # Spatie Settings (App, Theme, Customization)
    ├── Cachet.php                 # Facade-ähnliche Hilfsklasse
    ├── CachetCoreServiceProvider.php
    └── CachetDashboardServiceProvider.php
```

### Architekturmuster im Core

Cachet 3.x folgt einem **Action-basierten** Muster: HTTP-Controller und Filament-Ressourcen delegieren an Klassen unter `src/Actions/` (z. B. `CreateIncident`, `UpdateSchedule`). Domain-Events unter `src/Events/` triggern Webhooks über `SendWebhookListener`.

Einstellungen werden über **Spatie Laravel Settings** verwaltet (persistiert in der `settings`-Tabelle), nicht über `.env`:

| Settings-Klasse | Gruppe | Wichtige Felder |
|-----------------|--------|-----------------|
| `AppSettings` | `app` | `name`, `about`, `locale`, `timezone`, `incident_days`, `major_outage_threshold`, `display_graphs`, `enable_external_dependencies` |
| `ThemeSettings` | `theme` | `app_banner`, `accent`, `accent_content`, `accent_pairing` |
| `CustomizationSettings` | `customization` | `header`, `footer`, `stylesheet` (Custom HTML/CSS) |

Verwaltung im Dashboard unter **Settings** → Manage Cachet / Theme / Customization / Localization.

---

## Anwendungsspezifischer Code

### User-Modell (`app/Models/User.php`)

- Erbt von `Cachet\Models\User` (Core)
- Implementiert `FilamentUser` für Dashboard-Zugang
- Nutzt `HasApiTokens` (Sanctum) und `Notifiable`
- `canAccessPanel()` gibt aktuell immer `true` zurück (keine rollenbasierte Einschränkung)
- `isAdmin()` delegiert an `$this->is_admin` (Feld aus Core-Migration)

Anpassung des User-Modells über `.env`:

```env
CACHET_USER_MODEL=App\Models\User
AUTH_MODEL=App\Models\User
```

### TrustProxies (`app/Http/Middleware/TrustProxies.php`)

Liest vertrauenswürdige Proxies aus `config('cachet.trusted_proxies')` (Env: `CACHET_TRUSTED_PROXIES`, kommagetrennt). Wichtig hinter Load Balancern / Reverse Proxies.

### AppServiceProvider

- `HOME = '/home'` – Redirect nach Login
- API-Rate-Limit: 60 Requests/Minute pro User oder IP (Laravel-Standard-`api`-Limiter; Cachet-API hat eigenen Limiter `cachet-api` im Core)

---

## Konfiguration (`config/cachet.php`)

Zentrale Cachet-Einstellungen. Wichtige Optionen:

| Key / Env | Default | Bedeutung |
|-----------|---------|-----------|
| `CACHET_ENABLED` | `true` | Statusseite ein/aus |
| `CACHET_PATH` | `/` | URI-Prefix für Statusseite & API |
| `CACHET_DOMAIN` | – | Optional eigene Domain |
| `CACHET_TITLE` | `{APP_NAME} - Status` | Seitentitel |
| `CACHET_GUARD` | – | Optional eigener Auth-Guard |
| `CACHET_API_RATE_LIMIT` | `300` | API-Requests/Minute |
| `CACHET_BEACON` | `true` | Anonyme Telemetrie an cachethq.io |
| `CACHET_DEMO_MODE` | `false` | Demo-Modus (DB-Reset alle 30 Min.) |
| `CACHET_TRUSTED_PROXIES` | `""` | Proxy-IPs |
| `CACHET_USER_MIGRATIONS` | `true` | User-Migrationen aus Core |

Unterstützte Locales u.a.: `de`, `de_AT`, `de_CH`, `en`, `en_GB`, `es_ES`, `ko`, `nl`, `ph`, `pt_BR`, `zh_CN`, `zh_TW`.

---

## Datenbank

### Migrationen in diesem Repo (Laravel-Basis)

| Migration | Tabelle |
|-----------|---------|
| `create_users_table` | `users` |
| `create_password_reset_tokens_table` | `password_reset_tokens` |
| `create_personal_access_tokens_table` | `personal_access_tokens` (Sanctum) |
| `create_failed_jobs_table` | `failed_jobs` |
| `create_cache_table` | `cache` |
| `create_sessions_table` | `sessions` |

**Cachet-Domain-Tabellen** kommen aus `vendor/cachethq/core/database/migrations/`.

### Migrationen im Core (Domain-Tabellen)

| Tabelle | Zweck |
|---------|-------|
| `component_groups` | Gruppierung von Components |
| `components` | Überwachte Dienste |
| `incidents` | Vorfälle (mit `guid` für URLs) |
| `updates` | Updates zu Incidents/Schedules (polymorph) |
| `incident_templates` | Vorlagen (Twig-Engine) |
| `incident_components` | Verknüpfung Incident ↔ Component |
| `schedules` | Geplante Wartungen |
| `schedule_components` | Verknüpfung Schedule ↔ Component |
| `metrics` / `metric_points` | Metriken und Messpunkte |
| `subscribers` / `subscriptions` | E-Mail-Abonnenten (Legacy-Schema, UI teilweise) |
| `webhook_subscriptions` / `webhook_attempts` | Ausgehende Webhooks |
| `api_keys` | API-Schlüssel (Sanctum) |
| `settings` | Spatie Settings (App/Theme/Customization) |

**Legacy-Tabellen** (aus Cachet 2.x migriert, **ohne PHP-Modelle/Controller in 3.x**):

| Tabelle | Status in 3.x |
|---------|---------------|
| `tags`, `component_tag`, `taggables` | DB-Schema vorhanden, **keine Anwendungslogik** |
| `meta` | DB-Schema vorhanden, **keine Anwendungslogik** |
| `actions`, `invites` | Legacy, ungenutzt in 3.x |

### Seeder

`DatabaseSeeder` legt einen Test-User an:

- E-Mail: `test@test.com`
- Passwort: `test123`

Der **Core-Seeder** (`Cachet\Database\Seeders\DatabaseSeeder`) wird im Demo-Modus alle 30 Minuten ausgeführt und legt zusätzlich Demo-Components, Incidents, Schedules, Metrics und Settings an.

---

## Routen & Endpunkte

Eigene `routes/web.php` ist leer. Das Core-Paket registriert alle Routen automatisch über `PendingRouteRegistration` und `routes/api.php`.

### Statusseite (öffentlich, Middleware `cachet`)

Prefix: `{CACHET_PATH}` (Default: `/`)

| Methode | Pfad | Controller | Name |
|---------|------|------------|------|
| GET | `/` | `StatusPageController@index` | `cachet.status-page` |
| GET | `/incidents/{guid}` | `StatusPageController@show` | `cachet.status-page.incident` |
| GET | `/setup` | `SetupController@index` | `cachet.setup.index` |
| POST | `/setup` | `SetupController@store` | `cachet.setup.store` |
| GET | `/health` | `HealthController` | `cachet.health` |
| GET | `/rss` | `RssController` | `cachet.rss` |

**Fehlende Routen (TODO im Code):** Subscribe, Manage Subscriptions, Unsubscribe – siehe `PendingRouteRegistration.php` Zeile 39.

### Dashboard (Filament)

| Pfad | Beschreibung |
|------|--------------|
| `{CACHET_PATH}/dashboard` | Filament-Admin (Login, Passwort-Reset, Profil) |

Berechnet via `Cachet::dashboardPath()` → bei `CACHET_PATH=/` ist das `/dashboard`.

### JSON-API

Prefix: `{CACHET_PATH}/api` (Default: `/api`), Middleware: `cachet:api` + `throttle:cachet-api`

**Öffentlich (nur Lesen):**

| Endpunkt | Beschreibung |
|----------|--------------|
| `GET /api/ping` | Health-Ping |
| `GET /api/version` | Cachet-Version |
| `GET /api/status` | Gesamtstatus |
| `GET /api/components` | Components auflisten |
| `GET /api/component-groups` | Component Groups |
| `GET /api/incidents` | Incidents |
| `GET /api/incidents/{id}/updates` | Incident-Updates |
| `GET /api/incident-templates` | Vorlagen |
| `GET /api/metrics` | Metriken |
| `GET /api/metrics/{id}/points` | Metric Points |
| `GET /api/schedules` | Wartungen |
| `GET /api/schedules/{id}/updates` | Schedule-Updates |

**Authentifiziert (Sanctum Bearer, Schreiben):** Gleiche Ressourcen mit `POST`, `PUT/PATCH`, `DELETE` – siehe `routes/api.php`.

API-Abilities (für API-Keys): definiert in `Cachet::getResourceApiAbilities()`.

### Laravel Health

| Pfad | Beschreibung |
|------|--------------|
| `/up` | Laravel-eigener Health Check (nicht Cachet-spezifisch) |

---

## Domain-Modelle (in `cachethq/core`)

| Model | Zweck |
|-------|-------|
| `Component` / `ComponentGroup` | Überwachte Dienste/Komponenten |
| `ComponentCheck` | Health-Checks |
| `Incident` / `Update` | Vorfälle und Updates |
| `IncidentTemplate` | Vorlagen für Incidents |
| `Schedule` | Geplante Wartungen |
| `Metric` / `MetricPoint` | Metriken/Zeitreihen |
| `Subscriber` / `Subscription` | E-Mail-Abonnenten |
| `WebhookSubscription` / `WebhookAttempt` | Ausgehende Webhooks |
| `User` | Basis-User (wird in App erweitert) |

Felder im Core-User (`Cachet\Models\User`): `is_admin`, `preferred_locale`, E-Mail-Verifizierung (`MustVerifyEmail`).

---

## Enums (Core)

| Enum | Werte / Zweck |
|------|---------------|
| `ComponentStatusEnum` | operational, performance_issues, partial_outage, major_outage |
| `ComponentGroupVisibilityEnum` | expanded, collapsed |
| `IncidentStatusEnum` | investigating, identified, watching, fixed |
| `ScheduleStatusEnum` | scheduled, in_progress, completed |
| `MetricTypeEnum` | average, sum |
| `MetricViewEnum` | today, last_24_hours, last_30_days |
| `ResourceVisibilityEnum` | guest, user (Sichtbarkeit) |
| `SystemStatusEnum` | Gesamtstatus der Statusseite |
| `WebhookEventEnum` | component_*, incident_*, metric_*, subscriber_* |
| `UserLevelEnum` | admin, user |

---

## Filament-Dashboard (Core)

Panel-ID: `cachet`, Pfad: `{CACHET_PATH}/dashboard`

### Ressourcen (CRUD)

| Ressource | Model | Navigation |
|-----------|-------|------------|
| `Components` | `Component` | Resources |
| `ComponentGroups` | `ComponentGroup` | Resources |
| `Incidents` | `Incident` | Resources |
| `Updates` | `Update` | Resources |
| `Schedules` | `Schedule` | Resources |
| `Metrics` | `Metric` | Resources |
| `IncidentTemplates` | `IncidentTemplate` | Resources |
| `Subscribers` | `Subscriber` | Resources (UI vorhanden, Funktion unvollständig) |
| `WebhookSubscriptions` | `WebhookSubscription` | Resources |
| `ApiKeys` | Sanctum Tokens | Settings |
| `Users` | `User` | Settings |

### Einstellungsseiten

| Seite | Zweck |
|-------|-------|
| `ManageCachet` | AppSettings (Name, Locale, Incident-Tage, …) |
| `ManageTheme` | ThemeSettings (Banner, Akzentfarben) |
| `ManageCustomization` | CustomizationSettings (Header/Footer/CSS) |
| `ManageLocalization` | Spracheinstellungen |
| `OhDear` (Integrations) | Oh Dear Feed-Import |

### Widgets

`Overview`, `Components`, `Feed` (Blog-RSS), `Support`

---

## Events & Webhooks (Core)

Domain-Events werden an `SendWebhookListener` gebunden:

```
Cachet\Events\Incidents\*
Cachet\Events\Components\*
Cachet\Events\Subscribers\*
Cachet\Events\Metrics\*
```

Webhook-Events (`WebhookEventEnum`): u.a. `incident_created`, `component_status_changed`, `subscriber_verified`.

Webhook-Versand über `spatie/laravel-webhook-server`, Queue: konfigurierbar via `CACHET_WEBHOOK_QUEUE_*`.

---

## Artisan-Commands (Core)

| Command | Zweck |
|---------|-------|
| `cachet:make-user` | Benutzer anlegen |
| `cachet:beacon` | Telemetrie senden (täglich geplant) |
| `cachet:version` | Version anzeigen (`3.x-dev`) |

**Hinweis:** `cachet:check-components` existiert in der installierten Version **nicht mehr** (wurde aus dem Core entfernt).

Geplante Tasks (Scheduler):

- `cachet:beacon` – täglich
- `db:seed --class=Cachet\Database\Seeders\DatabaseSeeder` – alle 30 Min. im Demo-Modus

---

## Entwicklung

### Voraussetzungen

- PHP 8.2+
- Composer
- Node.js/npm (für Vite-Assets im Core)
- SQLite (Default) oder andere DB

### Setup

```bash
composer install
cp .env.example .env   # falls nicht vorhanden
php artisan key:generate
touch database/database.sqlite   # bei SQLite
php artisan migrate
php artisan vendor:publish --tag=cachet        # Config, Views, Assets
php artisan vendor:publish --tag=cachet-assets --force  # Public Assets nach public/vendor/cachethq/cachet
php artisan db:seed
```

Veröffentlichte Assets liegen unter `public/vendor/cachethq/cachet/` (Favicon, Vite-Build für Dashboard).

### Dev-Server (alle Prozesse parallel)

```bash
composer dev
```

Startet: `artisan serve`, Queue-Listener, Log-Tailing (Pail), Vite.

### Login (Dashboard)

- URL: `{APP_URL}/dashboard`
- E-Mail: `test@test.com`
- Passwort: `test123`

---

## Composer-Abhängigkeiten (Auszug)

```json
"require": {
    "php": "^8.2",
    "cachethq/core": "dev-main",
    "laravel/framework": "^11.2",
    "laravel/sanctum": "^4.0",
    "laravel/nightwatch": "^1.0",
    "league/flysystem-aws-s3-v3": "^3.32"
}
```

Das Core-Paket wird über VCS aus `git@github.com:cachethq/core.git` bezogen (`composer.json` → `repositories.cachet-core`).

---

## Wo Änderungen hingehören

| Aufgabe | Ort |
|---------|-----|
| Statusseite, API, Dashboard, Incidents | `cachethq/core` (oder Fork) |
| Eigenes User-Modell, Auth-Anpassungen | `app/Models/User.php` |
| Cachet-Konfiguration | `config/cachet.php` / `.env` |
| Proxy-/Infra-Anpassungen | `app/Http/Middleware/TrustProxies.php` |
| App-spezifische Laravel-Logik | `app/Providers/AppServiceProvider.php` |
| Eigene Web-Routen (selten nötig) | `routes/web.php` |

**Regel für KI-Assistenten:** Vor Änderungen an Features prüfen, ob der Code in `vendor/cachethq/core/` liegt. Direkte Edits in `vendor/` sind tabu – stattdessen Core forken oder App-Layer erweitern.

### Anpassungspunkte ohne Core-Fork

| Anpassung | Vorgehen |
|-----------|----------|
| Statusseiten-Template | `php artisan vendor:publish --tag=cachet-views` → `resources/views/vendor/cachet/` |
| Cachet-Config | `config/cachet.php` (bereits publiziert) |
| User-Modell / Auth | `app/Models/User.php` |
| Custom CSS/HTML | Dashboard → Settings → Customization (persistiert in DB) |
| Eigene Web-Routen | `routes/web.php` (selten nötig) |

---

## Feature-Status

### Funktionsfähig (laut `cachethq/core` README)

| Feature | Status |
|---------|--------|
| Incident Management | ✅ |
| Incident Updates | ✅ |
| Scheduled Maintenance | ✅ |
| Scheduled Maintenance Updates | ✅ |
| Components | ✅ |
| Metrics | ✅ |
| API | ✅ (Feintuning möglich) |
| Webhooks | ✅ |
| Dashboard (Filament) | ✅ |
| Localization | ✅ |
| Status Page Customization | ✅ |
| User Management | ✅ |

---

## Noch nicht funktionsfähig / in Entwicklung

> Quellen: `vendor/cachethq/core/README.md`, Code-Analyse in `vendor/cachethq/core/src/`.
> Die folgenden Bereiche sind **explizit als unvollständig** markiert oder im Code mit `@todo` / fehlenden Routen dokumentiert.

### Gesamtprojekt

| Bereich | Status | Hinweis |
|---------|--------|---------|
| **Cachet 3.x insgesamt** | ⚠️ Nicht produktionsreif | Offiziell: „currently in development and is not yet completely ready for production use“ |
| **Upgrade von Cachet 2.x** | ⚠️ Unvollständig | 3.x ist ein Neuaufbau; Legacy-Tabellen (`tags`, `meta`, …) existieren in der DB, aber ohne 3.x-Logik |
| **Automatisierte Tests** | ❌ Entfernt | Dev-Dependencies und Tests wurden aus dem App-Repo entfernt; Tests existieren nur im Core-Paket |

### Core-Features (explizit offen laut Roadmap)

| Feature | Status | Was fehlt (Code-Nachweis) |
|---------|--------|---------------------------|
| **Subscribers (E-Mail-Abonnenten)** | ❌ Nicht fertig | Filament-Ressource `SubscriberResource` und Actions (`CreateSubscriber`, …) existieren. **Fehlend:** Öffentliche Routen für Subscribe/Unsubscribe (`PendingRouteRegistration.php`: `@todo subscription routes`). **Fehlend:** Benachrichtigungsversand bei Incidents/Wartungen (`CreateIncident.php`, `CreateSchedule.php`, `CreateUpdate.php`: `@todo Dispatch notification`). SMS/Slack-Felder im Dashboard-Formular auskommentiert. |
| **Meta (SEO-/Meta-Daten)** | ❌ Nicht fertig | Migration `CreateMetaTable.php` legt Tabelle an, aber **kein** Model, Controller oder Filament-Ressource in `src/`. |
| **Tags** | ❌ Nicht fertig | Migrationen `CreateTagsTable`, `CreateTaggablesTable` vorhanden, aber **keine** PHP-Implementierung in `src/`. |

### Offene TODOs im Core-Quellcode

| Datei | TODO |
|-------|------|
| `PendingRouteRegistration.php` | Subscription-Routen (subscribe, manage, unsubscribe) |
| `Actions/Incident/CreateIncident.php` | Notification bei neuem Incident |
| `Actions/Update/CreateUpdate.php` | Notification bei Incident-Update |
| `Actions/Schedule/CreateSchedule.php` | Notification bei geplanter Wartung |
| `Actions/Schedule/UpdateSchedule.php` | Notification bei Wartungs-Update |
| `Http/Controllers/Api/ComponentController.php` | Verhalten bei gelöschten Components mit verknüpften Incidents; Neuberechnung der Component-Reihenfolge |

### Teilweise implementiert / Feinschliff nötig

| Feature | Status | Hinweis |
|---------|--------|---------|
| **REST-API** | ⚠️ Größtenteils fertig | Als „complete but may need some tweaks“ beschrieben; Component-Delete-Edge-Cases offen |
| **Dashboard-Zugangskontrolle** | ⚠️ Unsicher | `App\Models\User::canAccessPanel()` gibt immer `true` zurück – jeder eingeloggte User hat Dashboard-Zugang |
| **Admin-Rollen** | ⚠️ Basis vorhanden | `is_admin`-Flag und `UserLevelEnum` (admin/user) existieren; feingranulare Berechtigungen fehlen |
| **Oh Dear Integration** | ⚠️ Vorhanden | Filament-Seite `Integrations/OhDear.php` + Action `ImportOhDearFeed` – externe Abhängigkeit, kein vollständiger Ersatz für 2.x-Integrationen |

### Aus Cachet 2.x bekannte Features – in 3.x noch nicht (wieder) verfügbar

Diese Variablen stehen in `.env.example`, sind aber **nicht** in `config/cachet.php` angebunden:

| Env-Variable | Feature aus 2.x | Status in 3.x |
|--------------|-----------------|---------------|
| `CACHET_EMOJI` | Emoji-Unterstützung in Statusmeldungen | ❌ Nicht implementiert (nur Env-Rest, keine Config) |
| `CACHET_AUTO_TWITTER` | Automatisches Posten bei Incidents auf Twitter/X | ❌ Nicht implementiert (nur Env-Rest, keine Config) |

Legacy-DB-Spalten ohne UI: `subscribers.phone_number`, `subscribers.slack_webhook_url` (Felder in Filament auskommentiert).

### Weitere geplante, aber noch offene Bereiche

| Bereich | Hinweis |
|---------|---------|
| **E-Mail-/Push-Benachrichtigungen** | Event-System und Webhooks vorhanden; direkter Subscriber-Mailversand fehlt (siehe `@todo Dispatch notification`) |
| **Zwei-Faktor-Authentifizierung** | In Cachet 2.x vorhanden; in 3.x noch nicht nachgebaut |
| **Slack-/Discord-Integrationen** | In 2.x als Integrationen verfügbar; in 3.x nur Webhooks als Alternative |
| **Component Health Checks** | Command `cachet:check-components` entfernt; automatische Component-Checks nicht aktiv |
| **Themes / CSS-Anpassung** | `ThemeSettings` + `CustomizationSettings` im Dashboard vorhanden; umfangreiche Theme-Verwaltung wie in 2.x fehlt |

---

## Lizenz & Sicherheit

- Lizenz: Proprietär (siehe `LICENSE.md`)
- Sicherheitsmeldungen: support@cachethq.io (siehe `SECURITY.md`)
- Markenrichtlinien: `TRADEMARKS.md`

---

## Nützliche Links

- Repository: https://github.com/cachethq/cachet
- Core-Paket: https://github.com/cachethq/core
- Doku: https://docs.cachethq.io/v3.x/
- Demo: https://v3.cachethq.io/dashboard
- 3.x-Ankündigung: https://github.com/CachetHQ/Cachet/discussions/4342
