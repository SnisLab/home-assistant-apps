# SnisLab Home Assistant Apps

Offizieller Katalog der Home Assistant Apps von SnisLab.

Die Apps werden für Home Assistant bereitgestellt und automatisch aus den
jeweiligen App-Repositories synchronisiert. Dieses Repository ist der stabile
Kanal. Test- und Edge-Versionen werden getrennt davon bereitgestellt.

## Installation

1. **Einstellungen > Apps > App Store** öffnen.
2. **Repositories verwalten** öffnen.
3. Dieses Repository hinzufügen:

   `https://github.com/SnisLab/home-assistant-apps`

4. Zum App Store zurückkehren und die gewünschte SnisLab-App installieren.

## Updates

Veröffentlichte Versionen werden automatisch in den Katalog übernommen. Danach
erscheinen sie in Home Assistant als verfügbare App-Versionen.

Beta- und Edge-Versionen gehören nicht zum stabilen Katalog. Sie werden in einem
separaten Repository geführt, damit Testversionen keine stabile Installation
ersetzen.

## Dokumentation

Die ausführliche Dokumentation zu einer App befindet sich im jeweiligen
App-Repository in `DOCS.md`. Dort stehen unter anderem Konfiguration,
Voraussetzungen, Fehlerbehebung und bekannte Einschränkungen.

Dieses README enthält bewusst nur die Informationen, die beim Hinzufügen des
Repositories in Home Assistant benötigt werden.

## Architektur

Die Verantwortlichkeiten bleiben getrennt:

```text
SnisLab/<projekt>
  -> Release und docker.io/dersni/<projekt>:<version>
  -> SnisLab/app-<projekt>
  -> SnisLab/home-assistant-apps/<projekt>
```

Das Hauptprojekt enthält die Anwendung. `app-<projekt>` enthält ausschließlich
das Home-Assistant-spezifische Packaging und verwendet das veröffentlichte
Image oder Release des Hauptprojekts. Dieses Repository dient nur der
Distribution.

## Aufbau Eines App-Repositories

Die Home-Assistant-Dateien liegen direkt im Root des jeweiligen
`app-<projekt>`-Repositories:

```text
app-example/
|-- config.yaml
|-- Dockerfile
|-- run.sh
|-- README.md
|-- DOCS.md
|-- CHANGELOG.md
|-- icon.png
|-- logo.png
`-- translations/
```

Die Anwendung selbst wird dort nicht dupliziert. Bei einem bereits gebauten
Container verweist `config.yaml` auf GHCR:

```yaml
name: "Example App"
version: "1.0.0"
slug: "snislab_example"
description: "Short description of the app"
arch:
  - aarch64
  - amd64
image: "docker.io/dersni/example"
```

Der Katalog kopiert `app-example` automatisch nach `example/`. Home Assistant
findet die dort enthaltene `config.yaml` als eigenständige App.

## Automatische Synchronisierung

Der Workflow `.github/workflows/sync-apps.yml` unterstützt zwei Wege:

- `repository_dispatch` mit dem Typ `app-released` aktualisiert sofort eine
  bestimmte App anhand ihres unveränderlichen Commit-SHAs.
- Ein täglicher Vollabgleich sowie `workflow_dispatch` erkennen neue
  `app-*`-Repositories und übernehmen deren neuestes veröffentlichtes Release.
  Bereits veröffentlichte Apps werden ausschließlich per Dispatch aktualisiert.

Der Vollabgleich entfernt keine App automatisch. Dadurch führt ein abgelaufenes
oder zu eng eingeschränktes Token nicht versehentlich zu einer Depublikation.
Vor einer bewussten Entfernung muss ihr `app-*`-Quell-Repository archiviert
oder umbenannt werden; anschließend werden Katalogordner und Metadateneintrag
durch einen separaten Commit entfernt.

Der Dispatch muss diese Daten enthalten:

```json
{
  "event_type": "app-released",
  "client_payload": {
    "repository": "app-example",
    "version": "1.0.0",
    "tag": "v1.0.0",
    "sha": "0123456789abcdef0123456789abcdef01234567",
    "image": "docker.io/dersni/example:1.0.0"
  }
}
```

Für Edge-Builds kann derselbe Dispatch mit `tag: "edge"` und dem Edge-Image
verwendet werden. Das App-Repository muss dafür einen `edge`-Ref besitzen:

```json
{
  "repository": "app-example",
  "version": "1.0.0",
  "tag": "edge",
  "sha": "0123456789abcdef0123456789abcdef01234567",
  "image": "docker.io/dersni/example:edge"
}
```

Ein Release-Workflow im `app-*`-Repository kann den Katalog so auslösen:

```yaml
- name: Home Assistant Katalog aktualisieren
  env:
    GH_TOKEN: ${{ secrets.HOME_ASSISTANT_APPS_TOKEN }}
    VERSION: ${{ steps.release.outputs.version }}
    IMAGE: ${{ steps.release.outputs.image }}
  run: |
    gh api --method POST repos/SnisLab/home-assistant-apps/dispatches --input - <<JSON
    {
      "event_type": "app-released",
      "client_payload": {
        "repository": "${{ github.event.repository.name }}",
        "version": "${VERSION}",
        "tag": "${{ github.ref_name }}",
        "sha": "${{ github.sha }}",
        "image": "${IMAGE}"
      }
    }
    JSON
```

Für private Quell-Repositories wird im Katalog das Secret
`ORG_REPOSITORY_TOKEN` benötigt. Es soll als Fine-Grained PAT ausschließlich
Leserechte auf Metadaten und Inhalte der benötigten `app-*`-Repositories
erhalten. Das Dispatch-Token erhält nur Schreibzugriff auf dieses Katalog-Repo.

## Lokale Prüfung

```shell
npm ci
npm test
```
