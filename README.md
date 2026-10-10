[README.md](https://github.com/user-attachments/files/33290313/README.md)
# Offline AI Code Refactoring Audit Cockpit Pro

**Lokales KI-Code-Audit- & Refactoring-Cockpit** für Python – Streamlit-UI, Ollama, Ruff, Pyright, Black und pytest.  
Kein Cloud-Zwang für die Analyse: Code und Prompts bleiben auf dem Rechner (bzw. dem konfigurierten Ollama-Host).

> **Philosophie:** Die KI darf vorschlagen und ändern – **deterministische Schichten entscheiden**, ob die Änderung akzeptiert wird.

```
KI = Vorschlag / Reparatur
AST / Diff / Accept-Gate = Struktur- & Sicherheitskontrolle
Ruff / Pyright / pytest = statische & Laufzeit-Kontrolle
User = Freigabe (Checkboxen standardmäßig aus)
```

---

## Features

| Bereich | Beschreibung |
|--------|--------------|
| **Import** | Einzelne `.py`-Datei oder ZIP-Codebase (Zip-Slip-geschützt) |
| **Syntax-First** | Hard-Stop (Einzeldatei) oder Soft-Gate pro Datei (ZIP) |
| **Hybrid-Analyse** | Ruff-Findings + AST (z. B. except-import) + Ollama-Vorschläge |
| **Filter & Suche** | Schlagwörter, Kategorien, Sortierung – Befundtext vollständig |
| **Selektives Apply** | Nur angehakte Tasks; Checkboxen starten **deaktiviert** |
| **Deterministische Fixes** | Ruff-Autofix, DTZ005, S110, except-import – oft ohne LLM |
| **Unified Diff** | Diff bevorzugt; Full-File als Fallback; bei großen/gechunkten Dateien Diff-only |
| **Accept-Gate** | Syntax, Größe, Whitespace, Struktur, Semantik, Lint-/Typ-Delta |
| **Repair-Loop** | Optional (Sidebar); Early-Stopping, Gate-Retry bei Semantics/Types |
| **Graph** | Import-Graph offline (lokales vis-network) oder Graphviz-Fallback |
| **Editor** | Ace-Editor, Sprung aus Befund/Graph zur Datei + Zeile |
| **Diff & Export** | Unified Diff gegen Import-Original, ZIP-Export |
| **pytest** | Opt-in (Upload-Code wird lokal ausgeführt – keine echte Sandbox) |

---

## Architektur

```
app.py                 → Streamlit-UI, Session-State, History
config.py              → Presets, Timeouts, Chunk-Budgets
core/
  ollama_client.py     → Ollama REST + Retries
  processor.py         → Analyse, Chunking, Diff/Patch, Apply, Graph
  accept_gate.py       → Accept / Reject von Kandidaten
  repair.py            → Optionaler Self-Healing-Loop
  validator.py         → Syntax, Ruff, Pyright, Black
  test_runner.py       → pytest (PASS / FAIL / SKIPPED / UNKNOWN)
  zip_handler.py       → ZIP extract/create, safe_relpath
  messages_de.py       → Deutsche UI-/Fehlerübersetzungen
static/
  vis-network.min.js   → Offline-Graph
```

### Apply-Pipeline (vereinfacht)

1. Import → Working-Tree + Original für Diff  
2. Analyse → Syntax → deterministische Findings → KI → Ruff-Merge → Filter  
3. Nutzer wählt Tasks  
4. Apply → deterministische Pre-Passes → Diff/Full-File → **Accept-Gate**  
5. Optional Repair-Loop  
6. Diff-Tab / ZIP-Export  

---

## Voraussetzungen

- **Python 3.10+**
- **[Ollama](https://ollama.com/)** lokal (oder vertrauenswürdiger Remote-Host) mit einem Code-Modell (z. B. `qwen3-coder`, `deepseek-coder-v2`, …)
- CLI-Tools im `PATH`: `ruff`, `black`, `pyright` (optional `pytest`, `graphviz`)

```bash
pip install -r requirements.txt
# Modelle z. B.:
ollama pull qwen3-coder
```

---

## Start

```bash
# Windows (mitgeliefertes Skript) oder:
streamlit run app.py
```

UI: `http://localhost:8501`

In der Sidebar: Ollama-Host, Modell, Preset (Allgemein / Security / Typing / Performance / Style),  
Test-Modus (schnell / gründlich), optional Repair-Loop und pytest.

---

## Sicherheit & Grenzen

- **Offline-Anspruch:** Graph und Tools lokal; Remote-Ollama wird in der UI gekennzeichnet.
- **Accept-Gate** und Diff-Scope sollen verhindern, dass die KI still andere Dateien oder Semantik zerstört – kein Ersatz für Code-Review.
- **pytest** führt hochgeladenen Code aus → nur bei vertrauenswürdigem Code aktivieren.
- Große Dateien: AST-Chunking + Diff-Pfad; Full-File bei Chunk ist absichtlich gesperrt.
- Geeignet für **manuelle, selektive** Audits – nicht als unbeaufsichtigter CI-Autopilot auf Fremdcode.

---

## Status (Kurz)

Alltagstauglich für Einzeldateien und mittelgroße Projekte mit manuellem Gatekeeping.  
Restrisiken: modellabhängige Diff-Qualität bei sehr großen Dateien, gelegentliches KI-Rauschen nach Re-Analyse.

Ausführlichere technische Doku: [`DOKUMENTATION.md`](DOKUMENTATION.md) · Revisionsnotizen: [`revision.txt`](revision.txt)

---

## Lizenz

Noch nicht veröffentlicht
