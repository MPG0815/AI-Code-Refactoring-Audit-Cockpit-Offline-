# AI-Code-Refactoring-Audit-Cockpit-Offline-

Offline Code Auditor Pro – Dokumentation
> Stand: 2026-10-03 – an den aktuellen Code angeglichen.
1. Projektübersicht
Lokales Werkzeug zur KI-gestützten Analyse, Refactoring und Validierung von Python-Codebases.
Streamlit-UI + Ollama (keine Cloud-Pflicht für die KI). Qualitätschecks: Ruff, Pyright, Black, pytest.
Hauptfunktionen
Bereich	Beschreibung
KI-Code-Audit	ZIP oder Einzeldatei; Hybrid: Ruff + AST + KI-Vorschläge
Syntax-First	Hard-Stop (Einzeldatei) / Soft-Gate (ZIP) bei Syntaxfehlern
Selektives Refactoring	Checkboxen standardmäßig aus; nur angehakte Tasks
Deterministische Fixes	except-import (AST), Ruff `--fix` – ohne LLM
Apply	Unified Diff bevorzugt, Full-File = sicherer Pfad; Accept-Gate
Accept-Gate	Syntax, Size, Whitespace-Bloat, Struktur/Defs/Dekoratoren, Semantik, Lint-/Typ-Delta
Repair-Loop	Opt-in (Sidebar); erhält `audit_mode`
Test-Modus	⚡ Schnell (Gate-Pyright ≤60s) / 🔬 Gründlich (volles Timeout)
Graph	vis-network lokal (offline) oder Graphviz-Fallback
History	Snapshots (changed files), Rollback
---
2. Architektur
```
app.py                 → UI, Session-State, Presets, History, Audit-Modus
config.py              → Konstanten, ANALYSIS_PRESETS, Timeouts, MAX_CHARS_PER_CHUNK
core/
  ollama_client.py     → Ollama REST, Retries
  processor.py         → Analyse, AST-Chunking, Diff/Patch, Apply, Graph
  accept_gate.py       → Accept/Reject von Änderungen (Stufe A+B)
  repair.py            → Optionaler Self-Healing-Loop
  validator.py         → Syntax, Ruff, Pyright, Black, except-import
  test_runner.py       → pytest (PASS/FAIL/SKIPPED/UNKNOWN)
  zip_handler.py       → ZIP extract/create, safe_relpath (Zip-Slip)
  __init__.py          → Public API
static/
  vis-network.min.js   → Offline-Graph
```
Datenfluss (Apply)
Import → `codebase` (+ `original_codebase` für Diff)
`analyze_codebase` → Syntax → except-import-AST → KI → Ruff-Merge → Dedupe
Checkbox-Auswahl → Tasks
`apply_selected_refactorings`
Pre-Pass: except-import nur bei passendem Task
Ruff-only Tasks → Autofix + Gate (kein LLM)
sonst LLM: Diff → Patch; Fallback Full-File; Gate
Repair-Loop nur wenn Sidebar „Self-Healing“ an und nicht nur deterministische Tasks
Diff / ZIP-Export
---
3. Wichtige Module (aktuell)
`processor.py`
`analyze_codebase` – Syntax-Mode hard/soft, Presets, Hybrid-Ruff, `ast_except_import`
`ast_smart_truncate` – AST-Chunking (Kopf + Top-Level-Defs, Dekoratoren, Fokus-Namen)
`parse_unified_diff` / `apply_unified_diff` – Patch; Misserfolg → `gate_rejections`
`apply_selected_refactorings` – scoped, `audit_mode`, deterministische Pfade
`is_deterministic_task` / `is_ruff_task` / `is_except_import_task`
Graph: `build_graph_data`, `build_graphviz_dot`, `_GRAPH_SKIP_ROOTS`
`accept_gate.py`
`accept_change` – inkl. Whitespace-Bloat, Dekorator-Verlust, Ruff-/Pyright-Delta
Pyright-Cache-Key: `(file, sha256, timeout)` – fast ≠ thorough
`validator.py`
`collect_syntax_suggestions`, `collect_ruff_suggestions`, `collect_except_import_suggestions`
`fix_except_scope_imports`, `is_whitespace_bloat` (geteilt mit Gate)
`apply_ruff_autofix` (safe; `--unsafe-fixes` Opt-in)
`repair.py`
`repair_codebase_until_clean(..., audit_mode=...)`
Gate-Rejections → nächste Iteration als Feedback-Tasks („Datei vollständig ausgeben“)
`app.py`
Repair-Loop und Audit-Modus in der Sidebar
Flash-Messages über `st.rerun`
Stabile Checkbox-Keys (ohne `hash()`)
---
4. Analyse-Presets
Key	Fokus
Allgemein	Bugs, Lesbarkeit, Best Practices
Security	Injection, Secrets, Validierung
Typing	Type Hints, Pyright-Themen
Performance	Loops, I/O, Caching
Style	Namen, Komplexität, PEP-8
---
5. Abhängigkeiten
`requirements.txt`: streamlit≥1.40, requests, ruff, black, pyright, pytest.  
System-PATH: `ruff`, `black`, `pyright`, `pytest`. Ollama lokal.
---
6. Bekannte Grenzen
Unified Diffs von lokalen Modellen oft fehlerhaft → Full-File ist der sichere Pfad.
Gate-Pyright im Schnell-Modus max. 60s (Timeout → SKIP, nicht „0 Fehler“).
Cross-File-Typbrüche: Gate per Datei; Gesamtbild in der End-Validierung.
Ruff-Autofix-Verwurf bei Whitespace-Bloat verwirft die ganze Datei-Fix-Runde (nicht pro Finding).
Graph: Kanten nur zu geladenen Projektdateien (eine Datei → 1 Knoten, 0 Kanten).
History: begrenzte Snapshots im Session-State (keine vollen ZIP-Bytes mehr in jedem Snapshot).
---
7. Start
```bat
streamlit run app.py
```
oder `start_Code-Agent.bat`. Ollama mit Modell (z. B. `deepseek-coder-v2:16B` / `llama3`) muss laufen.
Empfohlene UI-Defaults für Stabilität
Self-Healing-Repair: aus
Test-Modus: Schneller Test (Alltag) / Gründlich vor Export
Nur konkrete Findings anhaken (nicht pauschal alle Docstrings)
