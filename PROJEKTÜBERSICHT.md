# Epstein-Research-Data – Projektübersicht

## Was ist dieses Projekt?

Das DOJ (US-Justizministerium) veröffentlichte am 30. Januar 2026 rund **2,73 Millionen Seiten** aus den Jeffrey-Epstein-Strafakten – aufgeteilt auf 12 Datensätze unter [justice.gov/epstein](https://www.justice.gov/epstein). Dieses Repository ist eine **forensisch aufbereitete, vollständig durchsuchbare Datenbanksammlung** dieser Akten.

**Kernzahlen:**
- **1.385.916 Dokumente** aus 12 EFTA-Datensätzen + FBI Vault (DS98) + House Oversight Committee (DS99)
- **2.771.231 Seiten** mit extrahiertem Volltext
- **~17 GB** Daten (unkomprimiert), aufgeteilt auf 13 SQLite-Datenbanken
- Über **165 forensische Analyseberichte** im Schwesterrepository [rhowardstone/epstein-research](https://github.com/rhowardstone/epstein-research)

---

## Was werde ich in diesem Projekt finden?

### 1. Volltextsuche aller Dokumente (`full_text_corpus.db`, 6,3 GB)
Die wichtigste Datenbank. Jede Seite jedes Dokuments ist indexiert und per FTS5 (Full-Text Search) durchsuchbar. Enthält auch Tabellen für schnelle Suche nach EFTA-Nummern und Datensätzen.

### 2. Produktionsmetadaten und E-Mail-Struktur (`concordance_complete.db`, 729 MB)
DOJ-Produktionsmetadaten: Originaldateinamen, E-Mail-Header (Von/An/CC/Betreff/Datum), Ordnerpfade, Custodians, MD5-Hashes. Ideal für die Suche nach E-Mail-Korrespondenzen.

### 3. Dokumenten-Veränderungen nach Veröffentlichung (`alteration_results.db`, 8,2 GB)
Tracking von **212.730 Änderungseinheiten**: Welche Dokumente wurden nachträglich verändert oder entfernt? Enthält LLM-Klassifikation, Pixel-Diff-Ergebnisse und Sensitivitätsbewertungen.

### 4. Schwärzungsanalyse (`redaction_analysis_v2.db`, 1,0 GB)
**2,6 Millionen erkannte Schwärzungsrechtecke** in 850.000 Dokumenten – inkl. OCR-Wiederherstellung von Text unter fehlerhaft angewendeten Schwärzungen (39.588 Seiten wiederhergestellt).

### 5. Bildanalyse (`image_analysis.db`, 762 MB)
**92.095 aus PDFs extrahierte Bilder**, analysiert mit Qwen2-VL-7B Vision-Modell. FTS5-durchsuchbar nach Beschreibung, Personen, Objekten und Umgebung.

### 6. DOJ-Entfernungsaudit (`doj_audit/`)
- **67.784 Dokumente** wurden nach der Erstveröffentlichung vom DOJ-Server entfernt (HTTP 404)
- **23.989 Dokumente** mit abweichenden Dateigrößen (nachträgliche Modifikation)
- **96.112 als verdächtig markierte Dokumente**

### 7. Alterierungsanalyse (`alteration_analysis/`)
- **21.803 klassifizierte Alterierungen** (Typen: CONTENT_REDUCTION, EMAILS_REMOVED etc.)
- **146.209 aus Dokumenten entfernte Entitäten** (Namen, Kontonummern, Telefonnummern)

### 8. Personenregister und Wissensgraph
- **`persons_registry.json`**: 1.614 namentlich erfasste Personen mit Aliasen, Kategorien und Quellenangaben (aus 9 Quellen zusammengeführt)
- **`knowledge_graph_entities.json`**: 606 kuratierte Entitäten (Personen, Firmen, Orte, Flugzeuge)
- **`knowledge_graph_relationships.json`**: 2.302 Beziehungen zwischen Entitäten mit Gewichtung

### 9. Weitere Spezial-Datenbanken
| Datenbank | Inhalt |
|-----------|--------|
| `handwriting_transcriptions.db` | Handschriftliche FBI/AUSA-Dokumente, transkribiert per KI |
| `transcripts.db` | 1.530 Audio-/Video-Dateien, 375 mit Sprachinhalt transkribiert |
| `prosecutorial_query_graph.db` | Analyse von Grand-Jury-Vorladungen: Was wurde verlangt, was geliefert? |
| `spreadsheet_corpus.db` | Native Tabellendaten aus DS8 |
| `ocr_database.db` | Tesseract-OCR-Ergebnisse für gescannte Seiten |
| `communications.db` | Extrahierte Kommunikationsmetadaten |

---

## Wie ist das Projekt gegliedert?

```
Epstein-research-data/
│
├── CLAUDE.md                        # Technische Hauptdokumentation (Englisch)
├── README.md                        # Projektübersicht und Datenbankstruktur
│
├── 📊 Strukturierte Daten (direkt nutzbar)
│   ├── persons_registry.json        # 1.614 Personen mit Metadaten
│   ├── knowledge_graph_entities.json
│   ├── knowledge_graph_relationships.json
│   ├── extracted_entities_filtered.json
│   ├── extracted_names_multi_doc.csv
│   ├── phone_numbers_enriched.csv
│   ├── image_catalog.csv.gz
│   ├── efta_dataset_mapping.csv/.json
│   └── document_summary.csv.gz
│
├── 🔍 Auditberichte
│   ├── doj_audit/                   # Entfernte/veränderte DOJ-Dokumente
│   └── alteration_analysis/         # Inhaltsveränderungen zwischen Versionen
│
├── 🛠️ tools/                        # Python-Skripte für Analyse und Datenbankaufbau
│
├── recovered_corrupted_pdfs/        # Forensisch wiederhergestellte PDFs
│
└── 💾 Datenbanken (via GitHub Releases, lokal herunterladen)
    ├── full_text_corpus.db          # Hauptdatenbank (FTS5, alle Seiten)
    ├── concordance_complete.db      # E-Mails, Metadaten
    ├── alteration_results.db        # Veränderungstracking
    ├── redaction_analysis_v2.db     # Schwärzungsanalyse
    ├── image_analysis.db            # Bildanalyse
    └── ... (8 weitere Datenbanken)
```

### Das EFTA-Nummersystem

EFTA-Nummern sind **seitenbasierte Bates-Stempel**, keine Dokument-IDs. Ein 10-seitiges Dokument belegt 10 aufeinanderfolgende EFTA-Nummern. Das ist der universelle Schlüssel durch alle Datenbanken.

| Datensatz | EFTA-Bereich | Inhalt |
|-----------|-------------|--------|
| DS1–DS7 | 00000001 – 00009664 | Frühe Strafverfolgungsakten, Gerichtsunterlagen |
| **DS8** | 00009676 – 00039023 | Native Dateien (Tabellen, Medien) |
| **DS9** | 00039025 – 01262781 | FBI-Ermittlung (531.000 Dokumente) |
| **DS10** | 01262782 – 02205654 | SDNY-Strafverfolgung (503.000 Dokumente) |
| **DS11** | 02205655 – 02730264 | Verteidigungsunterlagen (332.000 Dokumente) |
| DS12 | 02730265 – 02858497 | Erweiterung März 2026 |

---

## Wie arbeite ich am effektivsten mit diesem Projekt?

### Schritt 1: Datenbanken herunterladen

```bash
# Hauptdatenbank (Volltextsuche, ~2,3 GB komprimiert)
gh release download v5.0 --repo rhowardstone/Epstein-research-data --pattern "full_text_corpus.db.gz.*"
cat full_text_corpus.db.gz.part_aa full_text_corpus.db.gz.part_ab > full_text_corpus.db.gz
gunzip full_text_corpus.db.gz

# Weitere Datenbanken
gh release download v5.1 --repo rhowardstone/Epstein-research-data --pattern "*.db.gz"
gh release download v4.0 --repo rhowardstone/Epstein-research-data --pattern "*.db.gz"
gunzip *.db.gz
```

### Schritt 2: Grundlegende Suchabfragen

#### Volltextsuche (schnellste Methode)
```sql
-- Suche nach einem Begriff mit FTS5
SELECT p.efta_number, p.page_number, substr(p.text_content, 1, 500)
FROM pages_fts fts
JOIN pages p ON p.rowid = fts.rowid
WHERE pages_fts MATCH 'Leon Black'
AND p.char_count > 50
LIMIT 20;
```

#### LIKE-Suche (für Teilübereinstimmungen)
```sql
SELECT efta_number, page_number, substr(text_content, 1, 500)
FROM pages
WHERE text_content LIKE '%Deutsche Bank%'
LIMIT 20;
```

#### Dokument komplett lesen
```sql
SELECT page_number, text_content
FROM pages
WHERE efta_number = 'EFTA00074206'
ORDER BY page_number;
```

#### Zwei Personen im selben Dokument
```sql
SELECT DISTINCT p1.efta_number
FROM pages p1
JOIN pages p2 ON p1.efta_number = p2.efta_number
WHERE p1.text_content LIKE '%Bill Clinton%'
AND p2.text_content LIKE '%Jeffrey Epstein%'
LIMIT 20;
```

### Schritt 3: Tools nutzen

Die Python-Skripte in `tools/` erkennen das Datenverzeichnis automatisch (kein Pfad-Editieren nötig):

| Tool | Verwendung |
|------|-----------|
| `tools/person_search.py` | Suche nach Personen mit Co-Occurrence-Erkennung |
| `tools/find_missing_efta.py` | Lücken in der EFTA-Produktion finden |
| `tools/mirror_coverage.py` | Prüfen, welche Spiegel welche Dokumente haben |
| `tools/congressional_scorer.py` | Dokumente nach politischer Relevanz bewerten |

### Schritt 4: Originaldokumente aufrufen

Jedes EFTA-Dokument ist als PDF über mehrere Quellen abrufbar:

| Quelle | URL-Muster |
|--------|-----------|
| **DOJ** (kanonisch, Age-Gate) | `https://www.justice.gov/epstein/files/DataSet%20{N}/EFTA{NUMBER}.pdf` |
| **RollCall** (kein Gate) | `https://media-cdn.rollcall.com/epstein-files/EFTA{NUMBER}.pdf` |
| **Kino/JDrive** (kein Gate) | `https://assets.getkino.com/documents/EFTA{NUMBER}.pdf` |

> **Wichtig:** Dataset-Nummer immer per SQL-Abfrage prüfen – nie raten!
```sql
SELECT dataset FROM documents WHERE efta_number = 'EFTA00074206';
```

### Datensätze datenbankübergreifend verknüpfen

```sql
-- Volltext + E-Mail-Metadaten kombinieren
ATTACH 'concordance_complete.db' AS conc;

SELECT p.efta_number, p.page_number,
       substr(p.text_content, 1, 300),
       c.original_filename, c.email_from, c.email_to, c.date_sent
FROM pages p
JOIN conc.documents c ON c.efta_number = p.efta_number
WHERE p.text_content LIKE '%wire transfer%'
LIMIT 20;
```

---

## Kritische Hinweise

1. **Opferschutz**: Echte Namen von Opfern niemals veröffentlichen. Pseudonyme (Jane Doe, JD#1) oder EFTA-Referenzen verwenden.

2. **Abwesenheit ≠ Nicht-Existenz**: Ein fehlendes Dokument kann versiegelt, in einem separaten Verfahren oder außerhalb der EFTA-Produktion liegen.

3. **Daten zeigen, nicht interpretieren**: Dokumenteninhalte präsentieren – Schlussfolgerungen dem Leser überlassen.

4. **Verifizieren vor Zitieren**: EFTA-Nummern und Dataset-Zuweisungen immer per Datenbankabfrage bestätigen.

---

## Weiterführende Ressourcen

- **Analyseberichte**: [rhowardstone/epstein-research](https://github.com/rhowardstone/epstein-research) – 165+ forensische Berichte mit EFTA-Quellenangaben
- **Web-Interface mit KI**: [epstein-data.com](https://epstein-data.com) – Visuelle Suche mit Claude-KI-Assistent
- **Community-Tools**: [COMMUNITY_PLATFORMS.md](https://github.com/rhowardstone/epstein-research/blob/main/COMMUNITY_PLATFORMS.md) – 78+ Community-Werkzeuge und Spiegel
- **Methodik**: [METHODOLOGY.md](https://github.com/rhowardstone/epstein-research/blob/main/METHODOLOGY.md)
