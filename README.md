# ESG-Kennzahlen und Mitarbeiterstimmung

Analysecode zur Bachelorarbeit *Zusammenhang zwischen ESG-Kennzahlen und
Mitarbeiterstimmung: Eine Sentimentanalyse von Glassdoor-Bewertungen im
IT-Sektor* (HAW Hamburg, Department Medientechnik, 2026).

Die Arbeit untersucht, ob die in Glassdoor-Bewertungen ausgedrückte
Mitarbeiterstimmung mit ESG-Kennzahlen von vier Unternehmen des S&P 500
Information-Technology-Sektors (AMD, Nvidia, HP, Dell) zusammenhängt.
Der zentrale Befund ist methodischer Natur: Vorzeichen und Signifikanz der
untersuchten Zusammenhänge hängen davon ab, auf welcher Analyseebene
gerechnet wird — gepoolt, unternehmensintern zentriert, klassenbalanciert
oder auf Basis der Jahresveränderungen.

---

## Ausführbarkeit

**Die Notebooks sind als Dokumentation der Methodik veröffentlicht, nicht
als lauffähige Pipeline.** Zwei Eingangsdatensätze können nicht
bereitgestellt werden:

- **Glassdoor-Bewertungen** — die Nutzungsbedingungen der Plattform stehen
  einer Weitergabe entgegen. Die daraus abgeleiteten aggregierten
  Jahresmittelwerte liegen in `data/panel/`.
- **Unternehmensberichte** (Sustainability Reports, 10-K, DEF 14A) — mehrere
  hundert Megabyte, öffentlich über die Investor-Relations-Seiten der
  Unternehmen und über SEC EDGAR abrufbar.

Die gespeicherten Zellausgaben zeigen die Ergebnisse jedes Schritts, sodass
die Analyse ohne erneute Ausführung nachvollziehbar ist. Die extrahierten
ESG-Daten in `data/esg/` sind der Ausgangspunkt für die Korrelationsanalyse
und ermöglichen es, die Ergebnisse ab Schritt 5 selbst nachzurechnen.

---

## Pipeline

Ausführungsreihenfolge und Umgebung:

| # | Notebook | Umgebung | Funktion |
|---|---|---|---|
| 1 | `V10_Pipeline1_Page_Filtering.ipynb` | Colab | Reduziert die PDF-Berichte per Keyword-Suche und gewichtetem Scoring auf die relevanten Seiten |
| 2 | `V10_Pipeline2_LLM_Data_Extraction.ipynb` | Colab | Extrahiert 51 ESG-Metriken aus den gefilterten Seiten über die Anthropic API |
| 3 | `01_merge_clean.ipynb` | lokal | Führt die nach Sternekategorie getrennt erhobenen Glassdoor-Dateien zusammen, dedupliziert und filtert nach Sprache |
| 4 | `02_sentiment_inference.ipynb` | Colab (GPU) | Wendet vier Sentimentmodelle an (VADER, nlptown, RoBERTa, Siebert) und validiert gegen die Sternebewertungen |
| 5 | `04_multicompany.ipynb` | lokal | Baut das Unternehmenspanel, berechnet Korrelationen, Fixed-Effects-Regressionen und sämtliche Robustheitsanalysen |
| — | `XAI_eXplainable_AI_for_ESG_Sentiment.ipynb` | Colab | SHAP-basierte Attributionsanalyse des RoBERTa-Modells |
| — | `pilot_superseded/03_esg_correlation.ipynb` | lokal | Pilotstudie mit zwei Unternehmen, **überholter Stand** (siehe unten) |

Die Colab-Notebooks binden Google Drive ein und verwenden `/content/`-Pfade.
Für `02_sentiment_inference.ipynb` ist eine GPU-Runtime erforderlich; die
Inferenz dauert dort etwa zwei Minuten je Unternehmen und Modell.

### Zur Pilotstudie

`03_esg_correlation.ipynb` dokumentiert die ursprüngliche Analyse mit AMD
und Nvidia. Sie ist **nicht der Stand der Arbeit** und wird ausschließlich
zur Nachvollziehbarkeit der in Kapitel 5 diskutierten Pilotbefunde
bereitgestellt. Bekannte Abweichungen zur finalen Analyse:

- Die Treibhausgasintensität wird als berichteter Wert übernommen statt aus
  Scope 1, Scope 2 und Umsatz berechnet.
- Die Metrikliste enthält Kennzahlen, die in der finalen Analyse wegen
  fehlender Vergleichbarkeit zwischen den Unternehmen ausgeschlossen wurden.
- Die Lag-Analyse ist als gelaggte Fixed-Effects-Regression implementiert,
  nicht als Lag-Korrelation.
- Es wird keine Korrektur für multiples Testen angewendet.

---

## Daten

```
data/
  esg/      AMD, Nvidia, HP und Dell — je 51 extrahierte ESG-Metriken
            für 2019–2024, nach manueller Bereinigung
  panel/    Aggregiertes Analysepanel: 20 Unternehmen-Jahre mit
            Sentiment-Jahresmitteln und 30 Analysevariablen
```

Die ESG-Dateien enthalten pro Metrik den Wert, die berichtete Einheit, das
Berichtsjahr und den Kontext der Fundstelle. Bei der Qualitätsprüfung wurden
drei Metriken identifiziert, die trotz identischer Bezeichnung zwischen den
Unternehmen nicht vergleichbar sind; sie sind von der Analyse ausgeschlossen
und in `results/analysis_metrics_selection.csv` mit Begründung dokumentiert.

## Ergebnisse

`results/` enthält die exportierten Analyseergebnisse:

| Datei | Abschnitt der Arbeit |
|---|---|
| `multicompany_panel.csv` | Datengrundlage aller Analysen |
| `analysis_metrics_selection.csv` | 3.2.3, 3.2.4 — Metrikauswahl und Ausschlussgründe |
| `multicompany_pooled_correlations.csv` | 4.4.1 — Tabelle 4.5, inklusive FDR-korrigierter p-Werte |
| `consistency_check.csv` | 4.4.3 — unternehmensübergreifende Konsistenz |
| `multicompany_lag_correlations.csv` | 4.4.4 — Lag-Analyse |
| `multicompany_fixed_effects.csv` | 4.5, Anhang A.2 — gesättigte Spezifikation |
| `multicompany_fixed_effects_sparse.csv` | 4.5, Anhang A.3 — sparsame Spezifikation |
| `multicompany_normalized_correlations.csv` | 4.6.1 — Normierungsvergleich |
| `within_firm_correlations.csv` | 4.6.3 — Within-Firm-Analyse |
| `multimodel_robustness.csv` | 4.6.8 — Multi-Modell-Prüfung |
| `analyseebenen.csv` | 5.1 — Vergleich der Analyseebenen |

---

## Installation

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Die lokalen Notebooks (`01`, `04`) benötigen keine GPU.

---

## Verwendete Modelle

| Modell | Rolle |
|---|---|
| `cardiffnlp/twitter-roberta-base-sentiment-latest` | primäres Sentimentmaß |
| `nlptown/bert-base-multilingual-uncased-sentiment` | Validierung gegen Sternebewertungen |
| `siebert/sentiment-roberta-large-english` | Robustheitsprüfung |
| VADER | lexikonbasierte Baseline |
| `claude-sonnet-4-20250514` | strukturierte ESG-Metrikextraktion |

---

## Lizenz

Code: MIT (siehe `LICENSE`).
Daten in `data/` und `results/`: CC BY 4.0.

Die ESG-Rohwerte stammen aus öffentlich zugänglichen Unternehmensberichten
und SEC-Filings; die Rechte an den Originaldokumenten liegen bei den
jeweiligen Unternehmen.

## Zitation

> Schuldt, A. (2026). *Zusammenhang zwischen ESG-Kennzahlen und
> Mitarbeiterstimmung: Eine Sentimentanalyse von Glassdoor-Bewertungen im
> IT-Sektor* [Bachelorarbeit]. Hochschule für Angewandte Wissenschaften
> Hamburg, Department Medientechnik.
