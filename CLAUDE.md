# Aufmaß-Rechner Malerbetrieb

## Kontext

Werkzeug für einen Malerbetrieb (Maler Betrieb Brinkmann, Saterland).
Gebaut vom Sohn des Inhabers, ca. 1 Stunde Arbeit pro Tag neben der Schule.
Der Inhaber ist der einzige Nutzer und gleichzeitig der Testkunde.

**Das Problem:** Ein Angebot zu schreiben dauert aktuell 45–50 Minuten.
Aufgeschlüsselt vom Inhaber selbst:

| Schritt | Zeit | Greift das Tool an? |
|---|---|---|
| Aufmaß vor Ort aufnehmen | 15 Min | Nein, er steht mit Zollstock im Raum |
| Passende Positionen raussuchen | 10 Min | Ja |
| Flächen berechnen und prüfen | 10 Min | Ja, Kernfunktion |
| Ins System eintippen und formatieren | 10 Min | Ja, später über Export |

Ziel: von 45–50 Minuten auf 15–20 Minuten.

## Wichtig: was NICHT gebaut wird

Der Betrieb nutzt **plancraft** als Handwerkersoftware. Die erzeugt bereits
fertige Angebots-PDFs mit Kundennummer, Projektnummer, Titelübersicht,
Fußzeile. Der gesamte Dokumententeil ist also gelöst.

Nicht bauen, unter keinen Umständen:

- PDF-Erzeugung, Angebotskopf, Briefbogen
- Kundenverwaltung, Kundendaten jeglicher Art
- Rechnungen, Zahlungen, Steuerkram
- Login, Benutzerverwaltung, Backend, Server
- Einen eigenen Leistungskatalog, der mit plancraft konkurriert

Das Tool ist **die Abkürzung durch den bestehenden plancraft-Katalog plus
die Rechnerei**, mehr nicht.

## Was das Tool macht

Räume rein → Flächen gerechnet → fertige Positionsliste raus, die er in
plancraft übernimmt.

1. Räume eingeben: Name, Länge, Breite, Höhe
2. Öffnungen pro Raum: Fenster und Türen (Breite x Höhe, Anzahl)
3. Flächen berechnen: Decke, Boden, Umfang, Wand brutto und netto
4. Leistungen pro Raum auswählen (aus einer kurzen Liste, siehe unten)
5. Positionsliste mit Mengen und Preisen ausgeben

## Rechenregeln

```
Deckenfläche      = Länge x Breite
Bodenfläche       = Länge x Breite
Umfang            = 2 x (Länge + Breite)
Bruttowandfläche  = Umfang x Höhe
Abzug             = Summe der Öffnungen, deren EINZELfläche >= ABZUG_AB_QM
Nettowandfläche   = Bruttowandfläche - Abzug
```

`ABZUG_AB_QM` steht als Konstante oben im JS und ist im UI änderbar.
Standard 2.5 (VOB/C: Öffnungen unter 2,5 m² werden übermessen).

**Offen:** Der Inhaber hat zwei Varianten genannt — VOB mit 2,5 m² oder
pauschal ab 2,0 m² nach Kundenabsprache. Deshalb einstellbar, nicht
fest verdrahtet. Muss noch endgültig geklärt werden.

## Technische Entscheidungen (nicht ändern)

- **Eine einzige `index.html`**, HTML + CSS + JS in einer Datei
- Kein Framework, kein Build-Prozess, kein npm-Paket
- Kein Backend, kein Server, keine Datenbank
- Speichern nur in `localStorage`
- Deployment später als statische Datei auf Cloudflare Pages

Grund: Der Entwickler hat 1 Stunde am Tag und Klausurphasen. Es darf
nichts geben, das ausfallen und ihn in der Schulwoche blockieren kann.

## Deutsche Eingaben

Nicht optional, das ist der häufigste Fehler:

- Dezimaltrennzeichen: Eingabe akzeptiert **Komma UND Punkt** (`3,45` und `3.45`)
- Ausgabe immer deutsch mit Komma, 2 Nachkommastellen
- Einheiten wie in plancraft: `m2`, `Lfm.`, `psch.`, `Stk.`, `Std.`, `Rolle`
- Mobil: `inputmode="decimal"`, große Felder, alles untereinander

## Datenschutz

Es dürfen **keine Kundennamen, Adressen oder sonstigen Personendaten**
ins Tool. Nur Räume, Maße und Leistungen. Damit ist das Thema erledigt,
und das soll so bleiben.

## leistungskatalog.json

Liegt im Repo. Enthält ~100 Leistungen aus den plancraft-Stammdaten,
mit Einheit, EK, Zuschlag, VK und einem `bezug`-Feld.

`bezug` sagt, woraus sich die Menge berechnet:

| bezug | Menge |
|---|---|
| `decke` | Deckenfläche |
| `wand` | Nettowandfläche |
| `wand_decke` | Wandfläche + Deckenfläche |
| `boden` | Bodenfläche |
| `umfang` | Raumumfang minus Türbreiten |
| `lfm_manuell` | laufende Meter, manuell |
| `stk` | Anzahl, manuell |
| `pauschal` | 1 |
| `stunden` | manuell |
| `fassade` | Außenfläche, nicht Teil der Raumberechnung |

**Achtung:** Die Datei wurde aus Screenshots abgetippt, nicht exportiert.
Es können Positionen fehlen und Tippfehler drin sein. Für die Entwicklung
reicht sie, vor echtem Einsatz muss der Inhaber drüberschauen.

Einzelne Einträge haben ein `achtung`-Feld: Dubletten, Tippfehler im
Originaltext, und zwei Positionen mit VK 0,00 € (Abklebearbeiten,
Gerüststellung).

Wichtig: Der Katalog hat ~100 Einträge, aber im Alltag braucht er nur
**10 bis 15**. Genau die kommen ins UI, der Rest bleibt außen vor.
Die Auswahl kommt vom Inhaber und wird hier ergänzt, sobald sie vorliegt.

## Stand

- [x] Interview mit dem Inhaber, Zeitaufschlüsselung
- [x] Zehn echte Angebote ausgewertet
- [x] Leistungskatalog als JSON
- [x] Repo, Node, Claude Code, erster Commit
- [ ] Flächenrechner (Räume, Öffnungen, Summen)
- [ ] Die 10–15 Alltagspositionen vom Inhaber einbauen
- [ ] Positionsliste mit Mengen ausgeben
- [ ] Gegenprobe: ein echtes Altangebot nachrechnen, Abweichung < 5 %
- [ ] Echttest bei einem Kundentermin, Zeit stoppen
- [ ] Optional ganz am Schluss: GAEB-Export (plancraft liest x80/x81/x82/x83)

## Arbeitsweise

- Jeden Tag ein Commit, auch wenn wenig passiert ist
- Keine ungefragten Zusatzfeatures. Was nicht in der Aufgabe steht, wird
  nicht gebaut. Wünsche kommen auf eine Später-Liste
- Erst wenn ein Schritt wirklich läuft, kommt der nächste
