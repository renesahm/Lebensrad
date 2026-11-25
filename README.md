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
| `html` | fertige HTML-Version der Mail (Inline-Styles, Markenfarben, Balken pro Bereich) |
| `rad_svg` | das ausgefüllte Rad als SVG mit Inline-Styles |
| `quelle` | URL der Seite |

Jede Antwort mit HTTP-Status 2xx gilt als Erfolg.

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
