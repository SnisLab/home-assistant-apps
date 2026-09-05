# SnisLab Home Assistant Apps

Dieses Repository ist der zentrale, öffentliche Katalog für die Home Assistant
Apps von SnisLab. Die App-Verzeichnisse werden automatisch erzeugt und dürfen
nicht direkt bearbeitet werden.

## Repository In Home Assistant Hinzufügen

1. **Einstellungen > Apps > App Store** öffnen.
2. Die Verwaltung der Repositories öffnen.
3. `https://github.com/SnisLab/home-assistant-apps` hinzufügen.

## Architektur

Die Verantwortlichkeiten bleiben getrennt:

```text
SnisLab/<projekt>
  -> Release und ghcr.io/snislab/<projekt>:<version>
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
image: "ghcr.io/snislab/example"
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
    "image": "ghcr.io/snislab/example:1.0.0"
  }
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
