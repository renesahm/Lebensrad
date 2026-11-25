# Mein Lebensrad

Interaktives Lebensrad im Design-System René Sahm. Eine einzelne statische Seite (`index.html`), läuft auf GitHub Pages ohne Build-Schritt.

## Ablauf

1. **Werte:** 12 Bereiche (aus der PDF-Vorlage) auf einer Skala von 0 bis 10, per Tipp oder Ziehen am Rad oder per Regler. Datum ist voreingestellt.
2. **Reflexion:**
   - 01 Drei Fokusbereiche für die nächsten 12 Monate (Auswahl aus den 12 Bereichen, sortiert nach Wert)
   - 02 Je Bereich: Was fehlt für eine 10?
   - 03 Je Bereich: konkreter erster Schritt, heute
   - 04 Wann spüre ich, dass ich auf dem richtigen Weg bin?
3. **Ergebnis:** Rad mit hervorgehobenen Fokusbereichen, Antworten, E-Mail-Formular, CTA „Standortgespräch buchen“.

Alle Eingaben bleiben im `localStorage` des Browsers (Schlüssel `lebensrad-v1`), bis der Nutzer sie selbst per E-Mail verschickt.

## E-Mail-Versand anbinden

Der Button ist fertig, der Versand selbst noch nicht angebunden. Oben im `<script>` in `index.html`:

```js
const EMAIL_CONFIG = {
  endpoint: "",   // z. B. Webhook von Make, Zapier, n8n, Brevo oder eigenem Backend
  headers: {}
};
```

Solange `endpoint` leer ist, sieht der Nutzer: „Der E-Mail-Versand wird gerade eingerichtet.“ Die Daten werden dann nur in die Browser-Konsole geschrieben.

Mit Endpunkt schickt die Seite einen `POST` mit `Content-Type: application/json`:

| Feld | Inhalt |
| --- | --- |
| `email` | Adresse des Nutzers |
| `einwilligung` | immer `true` (Checkbox ist Pflicht) |
| `datum`, `datum_text` | `2026-11-25`, `25. November 2026` |
| `werte` | Liste mit `id`, `bereich`, `oberbereich`, `wert` für alle 12 Bereiche |
| `fokus` | Liste mit `id`, `bereich`, `wert`, `was_fehlt`, `erster_schritt` |
| `signal` | Antwort auf Frage 4 |
| `text` | fertige Klartext-Version der Mail |
| `html` | fertige HTML-Version der Mail als Mailtext (Inline-Styles, Markenfarben, Balken pro Bereich) |
| `rad_svg` | das ausgefüllte Rad als SVG mit Inline-Styles |
| `pdf_dateiname` | z. B. `Lebensrad-2026-11-25.pdf` |
| `pdf_base64` | der fertige Report als PDF, Base64-kodiert (ohne `data:`-Präfix), ca. 0,7–1 MB |
| `quelle` | URL der Seite |

Jede Antwort mit HTTP-Status 2xx gilt als Erfolg.

### PDF-Report

Beim Klick auf „Senden“ baut die Seite im Browser einen A4-Report und hängt ihn als `pdf_base64` an:

1. Titelblatt (Navy): „Mein Lebensrad“, Datum, die drei Fokusbereiche
2. Das Rad mit allen 12 Werten, nach Oberbereichen
3. Die drei Fokusbereiche mit „Was fehlt für eine 10“ und „Erster Schritt“, dazu die Antwort auf Frage 4 (bei langen Antworten wird automatisch auf weitere Seiten umbrochen)
4. Abschlussseite (Navy) mit „Standortgespräch buchen“, im PDF anklickbar

Der Versanddienst muss das PDF als Anhang an die Mail hängen: Dateiname aus `pdf_dateiname`, Inhalt aus `pdf_base64`. Viele Dienste nehmen Anhänge direkt als Base64 entgegen (bei Brevo nach meinem Kenntnisstand im Feld `attachment` mit `name` und `content`; bitte in der Doku des jeweiligen Dienstes prüfen). Falls die PDF-Erstellung im Browser scheitert, sind beide Felder `null` und die Mail geht trotzdem raus.

Die PDF-Seiten sind Bilder (aus dem HTML gerendert), der Text im PDF ist daher nicht markierbar. Dafür sieht der Report überall gleich aus.

Die Bibliotheken dafür liegen in `vendor/` (html2canvas 1.4.1, jsPDF 2.5.1, beide MIT-Lizenz). Sie werden erst beim Versand geladen und kommen vom eigenen Server, nicht von einem CDN.

Hinweise:

- Viele E-Mail-Programme (u. a. Gmail, Outlook) zeigen eingebettetes SVG nicht an. Für ein Bild des Rads in der Mail muss der Dienst `rad_svg` in PNG umwandeln oder als Anhang mitschicken. Die `html`-Version kommt ohne Bild aus und zeigt die Werte als Balken.
- Der Endpunkt muss CORS für die GitHub-Pages-Domain erlauben (Make- und Zapier-Webhooks tun das in der Regel).
- Der Einwilligungstext und ein Link zur Datenschutzerklärung sollten rechtlich geprüft werden. Ein Link ist noch nicht gesetzt.

## Inhalte ändern

- Bereiche und Oberbereiche: `AREAS` und `GROUPS` im Script
- CTA-Link: `#ctaBtn` (aktuell `https://www.renesahm.de/widget/bookings/standort-call`)
- Farben und Schriften: CSS-Variablen in `:root`, übernommen aus dem Design-System René Sahm

## Veröffentlichen mit GitHub Pages

Repository → Settings → Pages → Source: „Deploy from a branch“ → Branch `main`, Ordner `/ (root)` → Save.
