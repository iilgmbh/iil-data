# iil-data — IIL Platform Schema-SSoT

> **Single-Source-of-Truth** für Cross-Repo-Daten-Schemata. Versioniert,
> Konsumenten-neutral, pure-JSON-Schema.

## Inhalt

| File | Beschreibung | Konsumenten |
|---|---|---|
| `apps.schema.json` | Schema für `apps.json` (27 IIL-Apps mit Stage/Dev-URLs) | `iil-relaunch` (produziert), `iil-pet-portal` (konsumiert) |
| `klickdummies.schema.json` | Schema für `klickdummies.json` (Cross-Repo-Klickdummy-Manifest) | `iil-pet-portal` (konsumiert), auto-generiert aus `iilgmbh/*` |

## Konsum-Pattern

### A) Build-time-fetch (empfohlen — kein Drift)

In Workflow:
```yaml
- name: Validate against schema
  run: |
    curl -sf -o /tmp/apps.schema.json \
      https://raw.githubusercontent.com/iilgmbh/iil-data/main/apps.schema.json
    pip install jsonschema
    python3 -c "
    import json
    from jsonschema import Draft202012Validator, FormatChecker
    schema = json.load(open('/tmp/apps.schema.json'))
    data = json.load(open('apps.json'))
    v = Draft202012Validator(schema, format_checker=FormatChecker())
    errors = sorted(v.iter_errors(data), key=lambda e: e.path)
    for e in errors[:10]:
        print(f'  [{\" › \".join(str(p) for p in e.path)}] {e.message}')
    exit(1 if errors else 0)
    "
```

### B) git-Submodule (für offline)

```bash
git submodule add https://github.com/iilgmbh/iil-data.git schemas/iil-data
# Konsument referenziert: ./schemas/iil-data/apps.schema.json
```

## Versionierung

Schemas folgen **additive-only** Konvention:
- Neue **optionale** Felder: PATCH-Bump (z. B. `1.0.1`)
- Neue **required** Felder oder breaking changes: MAJOR-Bump (z. B. `2.0.0`)
- Git-Tags für Major-Versionen: `v1.0.0`, `v2.0.0`

Aktuelle Versionen: in jedem Schema-File `$id` + `version` (TBD).

## Verwandte Repos

- [`iilgmbh/iil-relaunch`](https://github.com/iilgmbh/iil-relaunch) — produziert `apps.json` (validiert via `apps.schema.json`)
- [`iilgmbh/iil-pet-portal`](https://github.com/iilgmbh/iil-pet-portal) — konsumiert beide Schemas
- [`iilgmbh/iil-klickdummy`](https://github.com/iilgmbh/iil-klickdummy) — Tool-Library, kann gegen `klickdummies.schema.json` validieren

## Lizenz

MIT (siehe `LICENSE`).
