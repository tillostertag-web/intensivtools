# IntensivTools — Projektspezifikation & AGENTS.md

> Für Claude Code · Stand: April 2026  
> Autor: Till · Vorlage: GynTools / BirthScan-Architektur

---

## Projektübersicht

**IntensivTools** = klinisches Entscheidungsunterstützungs-Tool für die internistische Intensivstation.  
Zwei separate Single-File-HTML-Anwendungen, analog zu GynTools (Gyn.html) und BirthScan.html.

| Datei | Tabs | Leitlinien |
|-------|------|------------|
| `IntensivScan1.html` | Sepsis · ARDS · AKI | SSC 2021, ARDSNet/Berlin, KDIGO 2012 |
| `IntensivScan2.html` | Kardiovaskulär · Gerinnung/DIC · Stoffwechsel | ESC 2023, ISTH 2023, ADA/AWMF |

---

## Architektur-Prinzipien (NICHT verhandeln)

```
1. Single-file HTML — kein Build-Step, kein Framework, kein CDN-Pflicht
2. Vanilla JS — keine npm-Dependencies
3. Alle Leitlinien-Logik inline im <script>-Block
4. CSS-Variablen für Design-Tokens (konsistent mit GynTools)
5. Mobile-first responsive (min-width: 320px)
6. Kein localStorage — kein Datenspeicher, kein Datenschutzproblem
7. Print-freundlich: @media print zeigt aktiven Tab ohne Chrome
```

---

## Design-System (GynTools-kompatibel)

```css
/* Exakt diese Variablen verwenden */
:root {
  --bg: #f4f6f9;
  --surface: #ffffff;
  --border: #dde2ea;
  --text: #1a2130;
  --text-muted: #5a6476;
  --accent: #1a5fa8;        /* ICU-Blau — konsistent mit GynTools */
  --accent-light: #e8f0fb;

  /* Ampel */
  --green: #1a7a4a;    --green-bg: #eaf6f0;
  --yellow: #8a6200;   --yellow-bg: #fef8e7;
  --orange: #b84a00;   --orange-bg: #fff1e8;
  --red: #b71c1c;      --red-bg: #fdecea;

  --radius: 8px;
  --shadow: 0 1px 4px rgba(0,0,0,0.08);
}
```

---

## Gemeinsame UI-Komponenten (beide Dateien)

### 1. Header
```html
<header>
  <div class="logo">IntensivTools</div>
  <div class="subtitle">Klinische Entscheidungsunterstützung · Intensivmedizin</div>
  <div class="badge">v1.0 · [Leitlinienstand]</div>
</header>
```

### 2. Tab-Bar
```html
<div class="tab-bar">
  <button class="tab-btn active" onclick="showTab('sepsis')">Sepsis</button>
  <button class="tab-btn" onclick="showTab('ards')">ARDS</button>
  <button class="tab-btn" onclick="showTab('aki')">AKI</button>
</div>
```

### 3. Ampel-Badge
```html
<!-- Immer diese 4 Klassen: ampel-gruen / ampel-gelb / ampel-orange / ampel-rot -->
<span class="ampel ampel-rot">SEPTIC SHOCK</span>
```

### 4. Score-Karte (Transparenz)
```html
<div class="score-card">
  <div class="score-header" onclick="toggleScore(this)">
    ▶ qSOFA-Score — Berechnung anzeigen
  </div>
  <div class="score-body hidden">
    <!-- Einzelkriterien mit Punkten -->
  </div>
</div>
```

### 5. Befund-Export-Button
```html
<button class="export-btn" onclick="exportBefund('sepsis')">
  📋 Befund kopieren
</button>
```
Export-Format: Plain-text, kliniküblich, direkt in KIS einfügbar.

### 6. Hinweis-Box
```html
<div class="hinweis hinweis-info">   <!-- info / warnung / kritisch -->
  <strong>Leitlinie:</strong> Surviving Sepsis Campaign 2021
</div>
```

---

## Tab-Spezifikationen

---

### TAB 1: SepsisScan (IntensivScan1.html)

**Leitlinien:** Surviving Sepsis Campaign 2021 · DGAI/DIVI · Sepsis-3-Definition

#### Sektionen:

**A) Screening**
- qSOFA-Score (0–3): RR syst ≤100, AF ≥22, GCS <15
  - 0–1: Grün (kein Hinweis)
  - 2–3: Rot (Sepsis-Verdacht → SOFA prüfen)
- SOFA-Score (vereinfacht): 6 Organsysteme, je 0–4 Punkte
  - SOFA ≥2 gegenüber Baseline = Sepsis-Definition erfüllt

**B) Klassifikation**
- Sepsis (SOFA ≥2 + Infektionsverdacht)
- Septischer Schock (Sepsis + Vasopressorbedarf + Laktat >2 mmol/l)
- Ampel-Ausgabe + klinische Konsequenz

**C) 1-Stunden-Bundle (SSC 2021)**
Checkliste mit Checkboxen (visuell, kein Datenspeicher):
- [ ] Laktat messen (Ziel <2 mmol/l)
- [ ] Blutkulturen × 2 (vor Antibiotikagabe)
- [ ] Breitspektrum-Antibiotikum i.v. (innerhalb 1h bei Schock)
- [ ] 30 ml/kg NaCl 0,9% bei Hypotonie/Laktat ≥4
- [ ] Vasopressoren bei MAP <65 mmHg

**D) Antibiotika-Hinweise**
- Community-acquired: Piperacillin/Tazobactam ± Aminoglykosid
- Nosokomial/VAP: Meropenem ± Vancomycin/Linezolid
- Hinweis: immer lokale Resistenzlage beachten (kein verbindlicher Algorithmus)

**E) Vasopressoren**
- Noradrenalin first-line (Ziel MAP ≥65)
- Vasopressin add-on bei NE >0,25 µg/kg/min
- Kortisol-Hinweis: Hydrocortison 200mg/d bei refraktärem Schock

**F) Monitoring-Ziele**
Tabelle: MAP ≥65, ScvO₂ ≥70%, Laktat-Clearance ≥10%/2h, Diurese ≥0,5 ml/kg/h

**Befund-Export:** qSOFA, SOFA, Klassifikation, Bundle-Status, Zeitstempel

---

### TAB 2: ARDSScan (IntensivScan1.html)

**Leitlinien:** Berlin-Definition 2012 · ARDSNet · ATS/ERS/ESICM 2017

#### Sektionen:

**A) Berlin-Klassifikation**
Eingaben: PaO₂/FiO₂-Ratio, PEEP ≥5 cmH₂O, beidseitige Infiltrate, kein kardiogenes Ödem
- PF-Ratio 200–300: Mild (Gelb)
- PF-Ratio 100–200: Moderat (Orange)
- PF-Ratio <100: Schwer (Rot)

**B) Beatmungs-Ziele (ARDSNet)**
- Tidalvolumen: 6 ml/kg IBW (Idealgewicht-Rechner: Formel nach ARDSNet)
- Plateaudruck: ≤30 cmH₂O
- PEEP-Tabelle (FiO₂/PEEP-Kombinationen: Low-PEEP vs. High-PEEP)
- Atemfrequenz 14–35/min, pH-Ziel 7,30–7,45
- SpO₂-Ziel 88–95%

**C) Idealgewichts-Rechner**
- Männer: 50 + 0,91 × (Körpergröße cm − 152,4)
- Frauen: 45,5 + 0,91 × (Körpergröße cm − 152,4)
- → Tidalvolumen-Ausgabe (6 ml/kg IBW) in ml

**D) Rescue-Therapien**
Stufenplan mit Ampel (nur bei schwerem ARDS):
1. Bauchlagerung ≥16h/d (Empfehlungsgrad A)
2. Neuromuskuläre Blockade (Cisatracurium, erste 48h)
3. iNO (Stickstoffmonoxid) — Bridge
4. ECMO-Kriterien (Murray-Score ≥3 oder pH <7,2)

**E) Driving Pressure**
- Driving Pressure = Plateau − PEEP (Ziel <15 cmH₂O)
- Rechner: Eingabe Plateau + PEEP → Ausgabe + Ampel

---

### TAB 3: AKIScan (IntensivScan1.html)

**Leitlinien:** KDIGO AKI 2012 · KDIGO AKD 2020 · EuReCa

#### Sektionen:

**A) KDIGO-Staging**
Kreatinin-basiert UND Urinausscheidungs-basiert (worst of both):

| Stadium | Kreatinin | Urinausscheidung |
|---------|-----------|-----------------|
| 1 | ×1,5–1,9 Baseline oder +0,3 mg/dl in 48h | <0,5 ml/kg/h × 6–12h |
| 2 | ×2–2,9 Baseline | <0,5 ml/kg/h × ≥12h |
| 3 | ×3 / ≥4 mg/dl / Dialysebeginn | <0,3 ml/kg/h × ≥24h / Anurie ≥12h |

Ampel: 1=Gelb, 2=Orange, 3=Rot

**B) Ursachen-Screening**
Quick-Check: Prärenal (Volumen? Vasopressoren?) / Renal (GN, Medikamente?) / Postrenal (Sono?)
Medikamenten-Check-Hinweis: NSAID, ACEi/ARB, Aminoglykoside, Kontrastmittel → pausieren?

**C) Flüssigkeits-Management**
- Stadium 1–2: Vorsichtiger Volumenversuch bei Prärenalem
- Stadium 3: Restriktiv, Bilanzierung, RRT erwägen
- Vermeidung Hypervolämie: kumulatives Flüssigkeitsbalance-Ziel <10% KG

**D) RRT-Indikationen**
Absolute Indikationen (Checkliste):
- [ ] Hyperkaliämie ≥6,5 mmol/l refraktär
- [ ] Metabolische Azidose pH <7,15 refraktär
- [ ] Urämische Symptome (Perikarditis, Enzephalopathie)
- [ ] Flüssigkeitsüberladung mit Lungenödem refraktär
- [ ] Bestimmte Intoxikationen

**E) Dosierung CRRT**
- Effluentrate Ziel: 20–25 ml/kg/h (KDIGO)
- Hinweis: Antibiotikaanpassung bei CRRT (generischer Hinweis, keine Tabelle)

---

### TAB 4: KardioScan (IntensivScan2.html)

**Leitlinien:** ESC Herzinsuffizienz 2023 · ESC Schock 2024 · ESC ACS 2023

#### Sektionen:

**A) Schock-Klassifikation**
Killip-Klassen I–IV (bei kardiogenem Schock):
- Haemodynamic Profile (wet/dry + warm/cold) nach Stevenson
- Ampel: kalt+nass = Rot (höchste Mortalität)

**B) Kardiogener Schock — SCAI-Klassifikation**
SCAI Stage A–E mit klinischen Kriterien und Therapieempfehlung

**C) Vasopressoren/Inotropika**
Tabelle:
| Substanz | Indikation | Dosis | Cave |
|----------|-----------|-------|------|
| Noradrenalin | Vasodilatatorischer Schock | 0,01–3 µg/kg/min | Tachykardie |
| Dobutamin | Kardiogener Schock (CO↓) | 2–20 µg/kg/min | Tachyarrhythmie |
| Levosimendan | Akute Dekompensation | 0,05–0,2 µg/kg/min | Hypotonie |
| Adrenalin | Refrakt. Schock / CPR | 0,01–1 µg/kg/min | Laktatanstieg |
| Vasopressin | NE-Additiv | 0,03 U/min fix | — |

**D) ACS auf der ICU**
- STEMI: Reperfusionsziel, Zeitfenster
- NSTEMI GRACE-Score (vereinfacht): Mortalitätsrisiko
- Post-Interventions-Monitoring-Checkliste

**E) Akute Herzinsuffizienz**
- LVEF-basierte Stratifizierung
- Diuretika-Algorithmus: Furosemid i.v. (Startdosis nach Vormedikation)
- Ziel: Diurese 0,5–1 ml/kg/h

---

### TAB 5: GerinnungScan (IntensivScan2.html)

**Leitlinien:** ISTH DIC 2023 · AWMF Gerinnungsmanagement · DGAI Massivtransfusion

#### Sektionen:

**A) DIC-Score (ISTH)**
Voraussetzung: Grunderkrankung mit DIC-Risiko (Sepsis, Trauma, Malignome)

| Parameter | Punkte |
|-----------|--------|
| Thrombozyten >100: 0, 50–100: 1, <50: 2 | 0–2 |
| PT-Verlängerung <3s: 0, 3–6s: 1, >6s: 2 | 0–2 |
| Fibrinogen ≥1 g/l: 0, <1 g/l: 1 | 0–1 |
| D-Dimer: normal: 0, mäßig↑: 2, stark↑: 3 | 0–3 |

- ≥5 Punkte: Manifeste DIC (Rot)
- <5 Punkte: Verdacht / Non-overt DIC (Gelb)

**B) Therapie-Algorithmus DIC**
- Grunderkrankung behandeln (primär)
- Thrombozyten: Transfusion bei <50 G/l + aktive Blutung / <20 G/l prophylaktisch
- FFP: bei PT >1,5× + Blutung
- Fibrinogen: Ziel >1,5 g/l (Fibrinogenkonzentrat oder Kryopräzipitat)
- Heparin: nur bei thrombotischer DIC-Dominanz (selten ICU)

**C) Massivtransfusions-Protokoll**
Trigger: >10 EK in 24h oder >4 EK in 1h + hämodynamische Instabilität
Ratio-Empfehlung: EK : FFP : TK = 1 : 1 : 1
Ziele: Fibrinogen >1,5 g/l, Thrombozyten >50 G/l, Ca²⁺ >1,1 mmol/l

**D) HIT (Heparin-induzierte Thrombozytopenie)**
4T-Score (vereinfacht): 0–8 Punkte
- ≤3: Niedrig (Heparin fortführen)
- 4–5: Intermediär (Heparin stopp, Alternative)
- 6–8: Hoch (Heparin stopp, Direktthrombinhemmer)

**E) Antikoagulation auf der ICU**
- TVT-Prophylaxe: NMH vs. UFH bei Niereninsuffizienz
- Therapeutische Antikoagulation: Dosierungshinweise NMH/UFH, Monitoring

---

### TAB 6: StoffwechselScan (IntensivScan2.html)

**Leitlinien:** ADA 2024 · AWMF DKA · DGE Ernährung Intensiv · DGEM

#### Sektionen:

**A) DKA-Klassifikation**
Kriterien: BZ >250 mg/dl, pH <7,3, Bikarbonat <18, Ketonurie/-ämie

| Schweregrad | pH | Bikarbonat | Bewusstsein |
|-------------|-----|------------|-------------|
| Mild | 7,25–7,30 | 15–18 | Alert |
| Moderat | 7,00–7,24 | 10–14 | Alert/Schläfrig |
| Schwer | <7,00 | <10 | Sopor/Koma |

Ampel: Mild=Gelb, Moderat=Orange, Schwer=Rot

**B) DKA-Therapie-Algorithmus**
1. Flüssigkeit: NaCl 0,9% 1L/h erste Stunde → dann nach Na⁺/BZ
2. Insulin: 0,1 IE/kg/h i.v. (KEIN Bolus außer bei Schwergrad 3)
3. Kalium-Ersatz: vor Insulin-Start wenn K⁺ <3,5 mmol/l!
4. Bikarbonat: nur bei pH <6,9 (kontroversiell)
5. Glucose 5%: zumischen wenn BZ <250 mg/dl (Insulin fortführen)
6. Anionenlücke-Rechner: Na − (Cl + HCO₃), Ziel <12

**C) HHS (Hyperosmolares Hyperglykämisches Syndrom)**
- BZ oft >600 mg/dl, Osmolalität >320 mosmol/kg
- KEIN Ketoazidose-Muster
- Therapie: vorsichtige Rehydratation (Risiko Hirnödem!), niedrig-dosiertes Insulin

**D) Elektrolyt-Störungen**
Mini-Algorithmus je Störung:

**Hyponatriämie:**
- Akut (<48h) vs. chronisch → Korrekturgeschwindigkeit!
- Ziel: max. +8–10 mmol/l / 24h (Risiko: osmotische Demyelinisierung)
- Symptomatisch schwer: NaCl 3% 100 ml über 10 min × 3

**Hyperkaliämie:**
| K⁺ | Maßnahme |
|----|----------|
| 5,5–6,0 | Ursache suchen, Diät |
| 6,0–6,5 | Kationentauscher, Schleifendiuretikum |
| >6,5 / EKG-Veränderungen | Kalzium i.v. + Insulin/Glukose + Bikarbonat + RRT erwägen |

**Hypokalzämie:** Symptomatisch → Kalziumglukonat i.v.
**Hypophosphatämie:** <0,6 mmol/l → i.v. Substitution (Refeeding-Syndrom-Risiko!)

**E) Ernährung Intensiv**
- Start: enterale Ernährung innerhalb 24–48h wenn möglich
- Kalorienziel: 25 kcal/kg/d (akute Phase: 70%)
- Proteinziel: 1,2–2,0 g/kg/d
- Blutzucker-Ziel: 140–180 mg/dl (ADA/SCCM)
- Insulin-Protokoll: klinischer Hinweis (kein konkreter Algorithmus)

---

## JavaScript-Logik-Struktur

```javascript
// Grundgerüst für alle Tabs
const IntensivTools = {
  
  // Tab-Navigation
  showTab(tabId) {
    document.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
    document.getElementById(tabId).classList.add('active');
    event.target.classList.add('active');
  },

  // Ampel-Ausgabe
  ampel(level) {
    // level: 'gruen' | 'gelb' | 'orange' | 'rot'
    const labels = { gruen: 'Normal', gelb: 'Erhöht', orange: 'Hoch', rot: 'Kritisch' };
    return `<span class="ampel ampel-${level}">${labels[level]}</span>`;
  },

  // Score-Toggle
  toggleScore(header) {
    header.nextElementSibling.classList.toggle('hidden');
    header.textContent = header.textContent.startsWith('▶')
      ? header.textContent.replace('▶', '▼')
      : header.textContent.replace('▼', '▶');
  },

  // Befund-Export
  exportBefund(tab) {
    const text = this.generateBefundText(tab);
    navigator.clipboard.writeText(text).then(() => {
      this.showToast('Befund in Zwischenablage kopiert');
    });
  },

  showToast(msg) {
    const t = document.createElement('div');
    t.className = 'toast';
    t.textContent = msg;
    document.body.appendChild(t);
    setTimeout(() => t.remove(), 2500);
  }
};
```

---

## Datei-Grundgerüst HTML

```html
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>IntensivScan1 — Sepsis · ARDS · AKI</title>
  <style>
    /* === DESIGN TOKENS === */
    /* === RESET === */
    /* === LAYOUT === */
    /* === HEADER === */
    /* === TAB-BAR === */
    /* === CARDS === */
    /* === AMPEL === */
    /* === SCORE-KARTEN === */
    /* === INPUTS === */
    /* === BUTTONS === */
    /* === TOAST === */
    /* === PRINT === */
    /* === RESPONSIVE === */
  </style>
</head>
<body>
  <header>...</header>
  <div class="tab-bar">...</div>
  <main>
    <!-- TAB: SEPSIS START -->
    <div id="sepsis" class="tab-panel active">...</div>
    <!-- TAB: SEPSIS END -->

    <!-- TAB: ARDS START -->
    <div id="ards" class="tab-panel">...</div>
    <!-- TAB: ARDS END -->

    <!-- TAB: AKI START -->
    <div id="aki" class="tab-panel">...</div>
    <!-- TAB: AKI END -->
  </main>
  <script>
    // === UTILITIES ===
    // === SEPSIS-LOGIK ===
    // === ARDS-LOGIK ===
    // === AKI-LOGIK ===
  </script>
</body>
</html>
```

---

## Build-Reihenfolge für Claude Code

### Phase 1 — Grundgerüst (ein Commit)
```
Aufgabe: Erstelle IntensivScan1.html als leeres Grundgerüst.
- Alle CSS-Variablen und Design-Tokens aus der Spezifikation
- Header, Tab-Bar (3 Tabs: Sepsis, ARDS, AKI), leere Tab-Panels
- JS-Utility-Funktionen: showTab, ampel, toggleScore, exportBefund, showToast
- Responsive Layout, Print-CSS
- Noch KEINE klinische Logik — nur die Hülle
Testkriterium: Alle 3 Tabs klickbar, Design korrekt, Mobile OK
```

### Phase 2 — Tab 1: SepsisScan
```
Aufgabe: Befülle den Sepsis-Tab in IntensivScan1.html.
Inhalt gemäß Spezifikation: qSOFA, SOFA, Klassifikation, 1h-Bundle, Vasopressoren, Monitoring
Keine anderen Tabs anfassen.
```

### Phase 3 — Tab 2: ARDSScan
```
Aufgabe: Befülle den ARDS-Tab in IntensivScan1.html.
Inhalt: Berlin-Klassifikation, IBW-Rechner, Tidalvolumen-Ausgabe, PEEP-Tabelle, Driving Pressure, Rescue
```

### Phase 4 — Tab 3: AKIScan
```
Aufgabe: Befülle den AKI-Tab in IntensivScan1.html.
Inhalt: KDIGO-Staging, RRT-Indikationen, CRRT-Dosierung, Medikamenten-Check
```

### Phase 5 — Datei 2 analog
```
IntensivScan2.html: Gleiche Phasen für Kardio, Gerinnung, Stoffwechsel
```

### Phase 6 — QA & Export
```
- Alle Befund-Export-Buttons testen
- Cross-Browser (Chrome/Safari/Firefox)
- Mobile (iOS Safari, Android Chrome)
- Print-Layout
- Leitlinien-Quellen im Footer
```

---

## Leitlinien-Referenzen (Footer beider Dateien)

```
IntensivScan1:
• Surviving Sepsis Campaign: International Guidelines 2021 (Evans et al., Crit Care Med 2021)
• ARDS Berlin Definition: JAMA 2012; ARDSNet Protocol 2000
• KDIGO Clinical Practice Guideline for Acute Kidney Injury 2012; AKD 2020

IntensivScan2:
• ESC Guidelines for Acute Heart Failure 2023
• ISTH DIC Guidance 2023
• ADA Standards of Care in Diabetes 2024
• DGEM/DGE Leitlinie Enterale Ernährung 2022
```

---

## Disclaimer (beide Dateien, Footer)

```html
<footer class="disclaimer">
  IntensivTools unterstützt klinische Entscheidungen — ersetzt nicht die ärztliche Beurteilung.
  Alle Empfehlungen basieren auf den zitierten Leitlinien. Keine Haftung für Therapieentscheidungen.
  Für den Einsatz als Medizinprodukt (EU MDR) ist eine Konformitätsbewertung erforderlich.
</footer>
```

---

## Deployment

- Netlify Drop (wie GynTools): beide HTML-Dateien einzeln deployen
- Alternativ: ein Netlify-Projekt mit `index.html` → Links zu beiden Dateien

---

*Ende der Spezifikation · IntensivTools v1.0*
