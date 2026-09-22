# 🚀 tinob-languages

> **Teil des `.tinob` Package Ecosystems**  
> Central Localization Engine für das `.tinob`-Ökosystem. Lädt Sprachdateien dynamisch, verwaltet Session-Locales und injiziert globale Projekt-Konstanten (`project-data`) direkt in die Übersetzungsschlüssel.

---

## 1. Repository & Beschreibung

* **Repository-Name:** `tinob-languages`
* **Beschreibung:** Deklaratives Lokalisierungspaket für das `.tinob`-Ecosystem. Inkorporiert globale Metadaten aus dem `project-data`-Block der zentralen `.tinob`-Konfiguration als PHP-Konstanten für redundanzfreie Mehrsprachigkeit.

---

## 2. Integration in die `.tinob` Konfiguration

In der zentralen `.tinob`-Steuerungsdatei wird das Paket unter `config -> active` registriert. Die globalen Projektdaten stehen im `project-data`-Block im Root:

```tinob
config
    active
        tinob-languages
        tinob-weak-password-detector
        tinob-login-signin
        tinob-rbac
        tinob-syslog

project-data
    name: PixelBoard
    version: 1.4.2
    environment: development

packages
    tinob-languages
        path: vendor/tinob-languages/locales/
        defaultLocale: de_DE
        fallbackLocale: en_US
        locales
            - de_DE
            - en_US
            - fr_FR

```

---

## 3. Bootstrapping & PHP-Konstanten

Beim Parsen des `project-data`-Blocks definiert das `.tinob`-Core-System globale PHP-Konstanten:

```php
// Vom System automatisch erzeugt aus 'project-data':
define('TINOB_PROJECT_NAME', 'PixelBoard');
define('TINOB_PROJECT_VERSION', '1.4.2');
define('TINOB_PROJECT_ENVIRONMENT', 'development');

```

---

## 4. Projektstruktur

```text
tinob-languages/
│
├── locales/
│   ├── de_DE.php
│   ├── en_US.php
│   └── fr_FR.php
│
├── src/
│   ├── LanguageEngine.php
│   └── helpers.php
│
├── controllers/
│   └── MainController.php
│
├── views/
│   └── main.php
│
└── index.php

```

---

## 5. Sprachdateien (`locales/`)

Da die Konfigurationswerte aus `project-data` als globale Konstanten vorliegen, können Sprachdateien direkt darauf zugreifen. Keine Dopplung von App-Namen oder Versionen!

### `locales/de_DE.php`

```php
<?php

return [
    'tinob-languages' => [
        'title' => 'Willkommen bei ' . TINOB_PROJECT_NAME,
        'description' => 'Deine Community für kreative Projekte.',
        'version' => 'Version ' . TINOB_PROJECT_VERSION,
    ],
    'tinob-login-signin' => [
        'login' => 'Anmelden',
        'register' => 'Registrieren',
        'logout' => 'Abmelden',
        'settings' => 'Einstellungen',
    ],
];

```

### `locales/en_US.php`

```php
<?php

return [
    'tinob-languages' => [
        'title' => 'Welcome to ' . TINOB_PROJECT_NAME,
        'description' => 'Your community for creative projects.',
        'version' => 'Version ' . TINOB_PROJECT_VERSION,
    ],
    'tinob-login-signin' => [
        'login' => 'Login',
        'register' => 'Register',
        'logout' => 'Logout',
        'settings' => 'Settings',
    ],
];

```

---

## 6. Helper & Engine (`src/helpers.php`)

```php
<?php

/**
 * Lädt die Sprachdatei für das aktuelle Locale basierend auf der .tinob Konfiguration.
 */
function loadLang(string $locale, string $basePath = 'vendor/tinob-languages/locales/'): array
{
    $langFile = rtrim($basePath, '/') . '/' . $locale . '.php';

    if (file_exists($langFile)) {
        return require $langFile;
    }

    // Fallback auf en_US
    return require rtrim($basePath, '/') . '/en_US.php';
}

/**
 * Entschärft Strings für die sichere HTML-Ausgabe.
 */
function e(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES,
        'UTF-8'
    );
}

```

---

## 7. MainController & View

### `controllers/MainController.php`

```php
<?php

class MainController
{
    public function index(string $locale): void
    {
        // Lädt die Wörterbücher inklusive injizierter TINOB_PROJECT_* Konstanten
        $translations = loadLang($locale);

        require __DIR__ . '/../views/main.php';
    }
}

```

### `views/main.php`

```php
<!DOCTYPE html>
<html lang="<?= e($locale) ?>">

<head>
    <meta charset="UTF-8">
    <title><?= e($translations['tinob-languages']['title']) ?></title>
</head>

<body>

    <h1><?= e($translations['tinob-languages']['title']) ?></h1>
    <p><?= e($translations['tinob-languages']['description']) ?></p>

    <button><?= e($translations['tinob-login-signin']['login']) ?></button>
    <button><?= e($translations['tinob-login-signin']['register']) ?></button>

    <footer>
        <?= e($translations['tinob-languages']['version']) ?>
    </footer>

</body>

</html>

```

---

## 8. Ablauf im Ecosystem

```text
               .tinob Konfigurationsdatei
                           │
                           ▼
                 [.tinob DSL Parser]
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
    Define PHP-Constants          Sequential Bootstrapping
    (TINOB_PROJECT_NAME)          active -> tinob-languages
             │                           │
             └─────────────┬─────────────┘
                           │
                           ▼
                    LanguageEngine
                           │
                           ▼
                 locales/de_DE.php
          (greift auf TINOB_* Konstanten zu)
                           │
                           ▼
                 $translations Array
                           │
                           ▼
                  View / HTML-Ausgabe

```

---

## 9. Kernvorteile dieser Architektur

1. **Keine Redundanz:** Projektname, Versionsnummern und Environments werden einmalig unter `project-data` definiert und als systemweite Konstanten bereitgestellt.
2. **Entkoppelte Sprachdateien:** Sprachdateien konsumieren direkt die globalen PHP-Konstanten (`TINOB_PROJECT_*`), wodurch manuelle Übergaben entfallen.
3. **Maximale Bootstrapping-Performance:** Da Konstanten global verfügbar sind, muss kein schwerfälliges State-Objekt durch das gesamte System gereicht werden.

```
