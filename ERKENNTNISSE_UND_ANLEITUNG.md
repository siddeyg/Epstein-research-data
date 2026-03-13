# Epstein-Akten: Erkenntnisse & Arbeitsanleitung

*Stand: 13. März 2026 — basierend auf forensischer Analyse der DOJ-Veröffentlichung*

---

## TEIL 1: WICHTIGSTE FORENSISCHE ERKENNTNISSE

### 1. Das DOJ hat nach der Erstveröffentlichung massiv eingegriffen

Die vielleicht bedeutsamste Entdeckung: Das Justizministerium hat die Akten **nach** der öffentlichen Veröffentlichung vom 30. Januar 2026 still und leise verändert.

| Befund | Zahl |
|--------|------|
| Dokumente vom DOJ-Server **entfernt** (HTTP 404) | **67.784** |
| Dokumente mit **veränderten Dateigrößen** (nachträgliche Modifikation) | **23.989** |
| Gesamt als auffällig markierte Dokumente | **96.112** |
| Belegte Änderungseinheiten (Change Units) in Dokumenten | **212.730** |

**Was wurde entfernt?** Laut den ausgewerteten Metadaten vor allem:
- Anklagepapiere und Schulderkenntnis-Vereinbarungen (Plea Agreements)
- Grand-Jury-Vorladungen (Subpoenas)
- FBI-Berichte und Strafverfolgungskorrespondenz
- Gerichtsunterlagen aus SDNY- und Florida-Verfahren

Beispiel: `EFTA00192747` — ein Dokument der DOJ-Abteilung für Kindesausbeutung (CEOS), das einen Schuldanerkenntnis-Prozess dokumentiert — wurde vollständig entfernt.

---

### 2. Gezielte Inhaltsveränderungen in verbliebenen Dokumenten

Von den **21.803 klassifizierten Alterierungen** sind die häufigsten Typen:

| Änderungstyp | Anzahl | Bedeutung |
|--------------|--------|-----------|
| `NAME_REMOVAL` | **16.269** | Namen wurden aus Dokumenten gelöscht |
| `EMAILS_REMOVED` | **1.831** | E-Mail-Adressen entfernt |
| `PHONE_REMOVAL` | **421** | Telefonnummern entfernt |
| `FINANCIAL_RECORD` | **18** | Finanzdaten verändert |
| `CONTENT_REDUCTION` | **43** | Inhalt reduziert/überschrieben |
| `METADATA_STRIP` | **350** | Dokumenten-Metadaten entfernt |

Von der KI-Klassifikation: **659 Fälle HIGH-sensitivity**, **18.937 MEDIUM**, 2.199 LOW.

**Beispiel aus den Finanzdaten** (`EFTA01577304`, DS10): Mehrere Bankkontonummern wurden aus einem Finanzdokument entfernt und durch Leerzeilen ersetzt — 6 Zeilen gelöscht, nichts hinzugefügt.

**Beispiel aus E-Mails** (`EFTA00968412`, DS9): Inhalt aus einer E-Mail entfernt, darunter laut KI-Analyse die Erwähnung von *"compromising pics"*.

---

### 3. 146.209 gezielt entfernte Entitäten

Aus den Dokumenten wurden zwischen Versionen **146.209 Entitäten** entfernt:
- Namen (häufigster Typ)
- Kontonummern (z.B. Konto `0043811863` — erscheint in 1.201 verschiedenen EFTA-Dokumenten und wurde systematisch entfernt)
- Telefonnummern
- E-Mail-Adressen

---

### 4. Schwärzungsanalyse: Millionen versiegelter Textstellen, teils wiederherstellbar

- **2,6 Millionen Schwärzungsrechtecke** in 850.000 Dokumenten gefunden
- **39.588 Seiten** mit wiederhergestelltem Text unter fehlerhaft angewendeten Schwärzungen
- Bei unsachgemäß geschwärzten Dokumenten konnte der ursprüngliche Text per OCR zurückgewonnen werden

---

### 5. Das Frankreich-Netzwerk (dokumentiert in den Akten)

Das EFTA-Korpus enthält umfangreiche Belege für Epsteins Operationen in Frankreich:

**Jean-Luc Brunel & MC2 Model Management:**
- `EFTA00023920`: *"Jeffrey Epstein has told me that he has slept with over 1,000 of Brunel's girls."*
- `EFTA01266359`: Epsteins Testament vermacht Brunel **5.000.000 US-Dollar**
- `EFTA01746878` (Jan 2015): Brunel an Epstein: *"I don't want to speak over the phone or email."*
- `EFTA00906771`: Modell Svetlana über MC2-Visum-Abhängigkeit: *"I m very worried about mc2 visa situation"* — Modelle wurden durch Visumkontrolle in Abhängigkeit gehalten

**22 Avenue Foch, Paris:**
- Epsteins Pariser Apartment (gehalten über SCI JEP, 999/1.000 Anteile)
- Bei einer Polizeidurchsuchung im September 2019 sichergestellt: Fotos unbekleideter junger Frauen, 30 Videos, eine CD-ROM mit 62 Frauennamen

**Französische Rechtshilfeanfrage (MLAT):**
- `EFTA00077200–77206`: Frankreich ersuchte die USA offiziell um Rechtshilfe (8. Juli 2020)
- Ermittlungsbehörde: Pariser Jugendschutzabteilung
- Vorwürfe: Vergewaltigung Minderjähriger unter/über 15 Jahren, kriminelle Vereinigung
- Beschuldigte: Epstein, Brunel, Maxwell und Mittäter

---

### 6. EFTA-Nummersystem: Keine Dokumente zurückgehalten (innerhalb des Systems)

Eine frühere Analyse behauptete, fast die Hälfte der Dokumente sei zurückgehalten. Das war **falsch**:

- EFTA-Nummern sind **seitenbasierte** Bates-Stempel (nicht dokumentbasiert)
- 100 % der EFTA-Seitennummern sind lückenlos belegt
- Scheinbare "Lücken" zwischen Datensätzen sind mehrseitige Dokumente an Datensatzgrenzen
- Beispiel: Die Lücke DS3→DS4 (EFTA00005587–00005704) gehört zu `EFTA00005586`, einem 119-seitigen Dokument

---

## TEIL 2: PRAKTISCHE ARBEITSANLEITUNG

### Schritt 1: Setup

#### Voraussetzungen
```bash
# SQLite prüfen (fast immer vorinstalliert)
sqlite3 --version

# GitHub CLI prüfen (empfohlen für Download)
gh --version
```

#### Datenbanken herunterladen

```bash
# === HAUPTDATENBANK (Pflicht) ===
# full_text_corpus.db — 6,3 GB, alle 2,77 Mio. Seiten mit FTS5-Suche
gh release download v5.0 --repo rhowardstone/Epstein-research-data \
  --pattern "full_text_corpus.db.gz.*"
cat full_text_corpus.db.gz.part_aa full_text_corpus.db.gz.part_ab > full_text_corpus.db.gz
gunzip full_text_corpus.db.gz
rm full_text_corpus.db.gz.part_*

# === ERGÄNZENDE DATENBANKEN (sehr empfohlen) ===
gh release download v5.1 --repo rhowardstone/Epstein-research-data --pattern "*.db.gz"
gh release download v4.0 --repo rhowardstone/Epstein-research-data --pattern "*.db.gz"
gh release download v4.0 --repo rhowardstone/Epstein-research-data --pattern "*.db"
gunzip *.db.gz
```

#### Setup prüfen
```bash
sqlite3 full_text_corpus.db \
  "SELECT COUNT(*) || ' Dokumente, ' || (SELECT COUNT(*) FROM pages) || ' Seiten' FROM documents;"
# Erwartete Ausgabe: 1385916 Dokumente, 2771231 Seiten
```

---

### Schritt 2: Suchen — die wichtigsten SQL-Muster

#### A) Schnellste Methode: FTS5-Volltextsuche
```sql
-- Terminal öffnen:
sqlite3 full_text_corpus.db

-- Suche nach Begriff (schnellste Methode)
SELECT p.efta_number, p.page_number, substr(p.text_content, 1, 500)
FROM pages_fts fts
JOIN pages p ON p.rowid = fts.rowid
WHERE pages_fts MATCH 'Leon Black'
AND p.char_count > 50
LIMIT 20;

-- Phrasensuche (exakter Ausdruck)
WHERE pages_fts MATCH '"Deutsche Bank"'

-- UND-Verknüpfung
WHERE pages_fts MATCH 'Epstein AND Maxwell'

-- Präfix-Suche
WHERE pages_fts MATCH 'Ghislain*'
```

#### B) Flexible Suche mit LIKE (langsamer, aber findet Teilstrings)
```sql
SELECT efta_number, page_number, substr(text_content, 1, 500)
FROM pages
WHERE text_content LIKE '%Deutsche Bank%'
AND char_count > 50
LIMIT 20;
```

#### C) Komplettes Dokument lesen
```sql
SELECT page_number, text_content
FROM pages
WHERE efta_number = 'EFTA00074206'
ORDER BY page_number;
```

#### D) Zwei Personen im selben Dokument (Co-Occurrence)
```sql
SELECT DISTINCT p1.efta_number
FROM pages p1
JOIN pages p2 ON p1.efta_number = p2.efta_number
WHERE p1.text_content LIKE '%Bill Clinton%'
AND p2.text_content LIKE '%Jeffrey Epstein%'
LIMIT 20;
```

#### E) Kontext um einen Namen herum anzeigen
```sql
SELECT p.efta_number, p.page_number,
       substr(p.text_content,
              MAX(1, INSTR(LOWER(p.text_content), LOWER('Ghislaine Maxwell')) - 200),
              600) AS kontext
FROM pages p
WHERE p.text_content LIKE '%Ghislaine Maxwell%'
AND p.char_count > 50
LIMIT 30;
```

#### F) Dataset herausfinden und PDF-URL bauen
```sql
-- Dataset zu einer EFTA-Nummer
SELECT efta_number, dataset, total_pages, file_path
FROM documents
WHERE efta_number = 'EFTA00074206';
-- Beispiel: dataset = 9
-- DOJ-URL: https://www.justice.gov/epstein/files/DataSet%209/EFTA00074206.pdf
-- RollCall-URL: https://media-cdn.rollcall.com/epstein-files/EFTA00074206.pdf
```

#### G) E-Mail-Suche in Konkordanzdatenbank
```sql
ATTACH 'concordance_complete.db' AS conc;

SELECT bates_begin, email_from, email_to, email_subject, date_sent, efta_number
FROM conc.documents
WHERE email_from LIKE '%maxwell%'
AND email_subject IS NOT NULL
ORDER BY date_sent
LIMIT 20;
```

#### H) Wiederhergestellten Text unter Schwärzungen suchen
```sql
sqlite3 redaction_analysis_v2.db

SELECT efta_number, page_number, substr(ocr_text, 1, 300), confidence
FROM redactions
WHERE ocr_text IS NOT NULL
AND length(ocr_text) > 30
AND efta_number LIKE 'EFTA001%'
LIMIT 20;
```

#### I) Bildsuche
```sql
sqlite3 image_analysis.db

SELECT image_name, efta_number, page_number, substr(analysis_text, 1, 300)
FROM images_fts
WHERE images_fts MATCH 'swimming pool'
LIMIT 10;
```

#### J) Entfernte Dokumente (DOJ-Audit) prüfen
```bash
# In der Shell (kein sqlite3 nötig):
grep "EFTA00192747" doj_audit/CONFIRMED_REMOVED.csv

# Alle entfernten Dokumente einer Kategorie
awk -F',' '$6=="prosecution"' doj_audit/FLAGGED_documents_details.csv | head -20
```

---

### Schritt 3: Datenbanken kombinieren (Cross-DB-Abfragen)

```sql
sqlite3 full_text_corpus.db

-- Volltext + E-Mail-Metadaten
ATTACH 'concordance_complete.db' AS conc;

SELECT p.efta_number, p.page_number,
       substr(p.text_content, 1, 300) AS seitentext,
       c.original_filename, c.email_from, c.email_to, c.date_sent
FROM pages p
JOIN conc.documents c ON c.efta_number = p.efta_number
WHERE p.text_content LIKE '%wire transfer%'
AND p.char_count > 50
LIMIT 20;
```

---

### Schritt 4: Python-Tools nutzen

```bash
# Alle Tools erkennen das Datenverzeichnis automatisch

# Person suchen (mit Co-Occurrence und CSV-Export)
python3 tools/person_search.py "Leon Black"

# Lücken in der EFTA-Produktion finden
python3 tools/find_missing_efta.py

# Dokumente nach politischer Relevanz bewerten
python3 tools/congressional_scorer.py

# Prüfen, welche Spiegel welche Dokumente haben
python3 tools/mirror_coverage.py
```

---

### Schritt 5: Originaldokumente aufrufen

Drei Quellen — alle liefern identische PDFs (SHA-256-verifiziert):

| Quelle | URL | Hinweis |
|--------|-----|---------|
| **DOJ** | `https://www.justice.gov/epstein/files/DataSet%20{N}/EFTA{NUMMER}.pdf` | Age-Gate, Dataset-Nr. nötig |
| **RollCall** | `https://media-cdn.rollcall.com/epstein-files/EFTA{NUMMER}.pdf` | Kein Gate, einfachste Option |
| **Kino/JMail** | `https://jmail.world/drive/EFTA{NUMMER}.pdf` | Viewer mit Seitennavigation |

> **Wichtig für DS8:** Abweichendes URL-Schema bei Kino/JMail:
> `https://assets.getkino.com/documents/vol00008-official-doj-latest-efta{nummer_kleinbuchstaben}.pdf`

**Dataset-Nummer IMMER per SQL prüfen — nie raten:**
```sql
SELECT dataset FROM documents WHERE efta_number = 'EFTA00192747';
```

---

## TEIL 3: STRUKTUR-ÜBERSICHT AUF EINEN BLICK

```
Worauf schaue ich zuerst?
│
├── Ich suche nach einem Namen oder Begriff
│   └── → full_text_corpus.db, Tabelle pages_fts (FTS5-Suche)
│
├── Ich suche nach E-Mails oder Dateimetadaten
│   └── → concordance_complete.db, Tabelle documents
│
├── Ich will wissen, was geschwärzt wurde
│   └── → redaction_analysis_v2.db, Tabelle redactions
│
├── Ich will sehen, was nachträglich verändert wurde
│   └── → alteration_results.db ODER alteration_analysis/*.csv
│
├── Ich will wissen, welche Dokumente vom DOJ entfernt wurden
│   └── → doj_audit/CONFIRMED_REMOVED.csv (67.784 Einträge)
│
├── Ich suche nach Bildinhalten
│   └── → image_analysis.db, Tabelle images_fts
│
├── Ich suche nach Audio-/Videotranskripten
│   └── → transcripts.db, Tabelle transcripts
│
├── Ich will Verbindungen zwischen Personen sehen
│   └── → knowledge_graph_entities.json + knowledge_graph_relationships.json
│
└── Ich suche eine bestimmte Person
    └── → persons_registry.json (1.614 Einträge mit Aliases)
```

---

## TEIL 4: GOLDENE REGELN

1. **Dataset-Nummer immer per SQL ermitteln** — nie aus der EFTA-Nummer ableiten
2. **FTS5 für Geschwindigkeit, LIKE für Flexibilität** — bei OCR-Fehlern ist LIKE besser
3. **Abwesenheit bedeutet nicht Nicht-Existenz** — fehlende Dokumente können versiegelt sein
4. **Daten zeigen, nicht interpretieren** — Belege präsentieren, Schlüsse dem Leser überlassen
5. **Opfernamen niemals nennen** — nur Pseudonyme (Jane Doe, JD#1) oder EFTA-Referenzen
6. **Vor dem Zitieren verifizieren** — EFTA-Nummer und Inhalt per Datenbankabfrage bestätigen
7. **RollCall als Backup** — wenn DOJ-Links nicht funktionieren (Age-Gate oder 404)

---

## TEIL 5: RESSOURCEN

| Ressource | Link |
|-----------|------|
| Originaldaten (DOJ) | https://www.justice.gov/epstein |
| Analyseberichte (165+) | https://github.com/rhowardstone/epstein-research |
| Web-Interface mit KI | https://epstein-data.com |
| DOJ-Entfernungsaudit | Bericht im Companion-Repo: `institutional/DOJ_DOCUMENT_REMOVAL_AUDIT.md` |
| Alterierungsforensik | Bericht: `institutional/DOJ_DOCUMENT_ALTERATION_FORENSICS.md` |
| Community-Tools (78+) | `COMMUNITY_PLATFORMS.md` im Companion-Repo |

---

*Alle Zahlen basieren auf direkter Auswertung der Datenbankdateien und CSV-Exporte in diesem Repository. Quellen sind per EFTA-Nummer verifizierbar.*
