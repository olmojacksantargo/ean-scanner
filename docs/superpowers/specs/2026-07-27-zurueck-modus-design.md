# v21 — „Zurück" als eigener First-Level-Modus

**Datum:** 2026-07-27 · **Status:** vom Nutzer freigegeben (Chat)

## Ziel

Drei klar getrennte Abläufe, direkt auf oberster Ebene erkennbar:

| Modus | Zweck | Schreibt in Airtable (Inventory Office) |
|---|---|---|
| 📥 Wareneingang | Echte Lieferungen (Indien/Bangladesch) | Status → `✅ Da`, `Anzahl Gescannt` +1, `Lieferung` = Session, `Datum`, EAN, Link |
| 📤 Versand | Ware geht raus (Shooting, Veredler …) | `Versand Status → 📦 Außerhaus`, `Versand an` = Session |
| ↩️ Zurück | Rückläufer kommen zurück | **nur** `Versand Status → ↩️ Zurück` + `Rückkehr von` = Session |

## Entscheidungen (aus dem Chat)

1. **„Zurück" first-level** im Session-Start-Segment neben Wareneingang und Versand.
   Der bisherige Raus/Zurück-Umschalter *innerhalb* der Versand-Session entfällt
   (Versand = immer Außerhaus).
2. **Rückkehr ist benennbar** (z.B. „Shooting S1 gopackshot"). Dafür neue Airtable-Spalte
   **„Rückkehr von"** (`fld6ecGFZzf8Akfhq`, singleLineText). `Versand an` bleibt als
   Historie unangetastet, `Lieferung` bleibt exklusiv für echte Lieferungen.
3. **Kein „Unerwartet" bei Rückläufern:** Der Zurück-Modus legt nie neue Zeilen an und
   setzt nie `🔵 Unerwartet` — jede SKU im Bestand wird auf ↩️ Zurück gesetzt, egal welche
   Größe. SKUs ohne Bestandszeile: warnen + überspringen (wie im Versand-Modus).
4. **Außerhaus-Schutz im Wareneingang:** Scannt man dort einen Artikel mit
   `Versand Status = 📦 Außerhaus`, warnt die App und überspringt ihn (Hinweis auf den
   Zurück-Modus), statt `Lieferung` zu überschreiben und doppelt zu zählen.

## Umsetzung (index.html, v21)

- Seg-Control mit 3 Buttons: `📥 Eingang · 📤 Versand · ↩️ Zurück` (kompakte Labels).
- `currentSession.mode ∈ {eingang, versand, zurueck}`; Feld `direction` entfällt.
  Migration: gespeicherte Session mit `mode=versand, direction=zurueck` → `mode=zurueck`.
- `buildVersandItem` setzt `target` aus dem Session-Modus (VS_AUSSER bzw. VS_ZURUECK).
- `batchSave`: Versand → `Versand Status` + `Versand an`; Zurück → `Versand Status` +
  `Rückkehr von`. Beides PATCH auf bestehende Zeile, nie POST.
- Pending-Liste: Zurück-Zeilen mit Tag `↩️ Zurück`, Zusatz „(war nicht raus)" wenn der
  Artikel gar nicht Außerhaus war.
- Scan-Log: vorhandene Kinds `ausserhaus`/`zurueck` bleiben; Wareneingang-Schutz loggt
  mit Status `error` + Kind `ausserhaus`.

## Nicht geändert

Match-Kette EAN → `EAN_CLEAN` → SKU → `SKU stabil`, Unerwartet-Logik im Wareneingang,
Batch-Speichern, Swipe-to-delete, Kamera/Scanner, Sendungsnummer (bleibt manuell).
