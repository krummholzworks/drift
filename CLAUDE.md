# CLAUDE.md — Übergabe für Drift (und den Orion-Prototyp)

Stand: Drift **v097k** (`index.html` im Repo `krummholzworks/drift`), Orion-Prototyp **orion-prototype-v011.html** mit neuem Kampf-Layer (siehe 3.3; liegt **nicht** im Repo).
Diese Datei ersetzt den Chat-Verlauf. Wo etwas **nicht verifiziert** ist, steht es ausdrücklich dabei — lieber nachprüfen als glauben.

---

## 1. Worum es geht

**Drift** ist ein wissenschaftlich fundiertes Incremental/Idle-Spiel für Mobilgeräte (Android, PWA auf GitHub Pages). Eine einzige HTML-Datei, Vanilla JS, Canvas 2D, kein Build-Prozess, kein Framework, keine Module. Solo-Entwicklung von Jan mit Claude als Co-Entwickler; Jan testet auf dem eigenen Gerät.

Tagline: „Quiet hands. Reach of a god." (Breath-Autopilot-Screen: „Quiet hands. Quiet breath.")

Der Spieler wischt über Partikel und baut so die Materie des Universums nach: Quarks → Protonen/Neutronen → Atome/Moleküle → Wolken → Sterne. Jede Epoche ist ein Durchlauf dieser Kette auf höherer Ebene; zwischen den Runs gibt es Prestige über **Echo** (Meta-Währung).

Jans übergreifende Idee hinter allen seinen Spielen (Drift, Robinson-Spiel, Anarchomonster): Konflikt zwischen idealen Gesellschaftsformen und problematischen menschlichen Eigenschaften, nie eine statische Utopie, sondern fortwährende Selbstauseinandersetzung, verhalten optimistisch. In Drift zeigt sich das nur indirekt (Antagonist, Barriere, kein Game Over). Die anderen Spiele sind **nicht** Teil dieses Codes.

### Design-Philosophie (verbindlich)

1. **Echte Physik ist Gestaltungsprinzip.** Massen, Ladungen, Bindungsregeln, Zündung folgen echten Werten und Zusammenhängen mit durchgängig einheitlicher Skalierung. Abweichungen (Zeitraffer bei der Zündung, vereinfachte Quark-Massenkette) sind benannte Kompromisse für die Spielbarkeit.
2. **Eine Spielschleife für alle Epochen.** Jan will in allen Epochen im Kern das gleiche Spiel: gleiche Kaufleiter-Logik, gleiche Rezepte, höhere Epochen haben nur mehr Stufen. Unterschiede zwischen Epochen müssen als Tabellenspalte erscheinen, nicht als if-Zweig.
3. **Keine „Sonderlocken".** Jans Wort für Einzelfall-Mechaniken. Sonderfälle werden zu Zeilen/Spalten in `PARTICLE_TABLE`, `RECIPE_TABLE`, `SPAWN_UPGRADE_TABLE`. Das steht auch als GROUND-RULES-Kommentar im Code (ca. Zeile 826).
4. **Diegetisches Feedback statt HUD.** Rückmeldungen passieren im Spielfeld (Farbwechsel, Ringe, Glimmen), keine dauerhaft neuen HUD-Elemente.
5. **Additiv statt multiplikativ** bei Boni (vorhersagbarer, leichter zu debuggen).
6. **Perzentil-basierte Schwellen:** Schwellenwerte kommen aus der tatsächlich erzeugten Werteverteilung, nicht aus Schätzung.
7. **Einmal teuer rechnen, oft günstig zeichnen** (z. B. Nebel als gebackenes Asset, Warp-Textur einmal pro Sonne).
8. **Kein Game Over.** Scheitern im Kampf führt in einen ruhigen Stillstandszustand, nicht in ein Ende (siehe 3.3).
9. **Bewusst kein Speichersystem:** Score, Echo, Echo-Shop-Käufe und Achievements leben nur im Arbeitsspeicher. (Ein localStorage-Save steht auf der Roadmap, siehe 6.)

---

## 2. Dateien und Struktur

| Datei | Was |
|---|---|
**Im Repo:**

| Datei | Was |
|---|---|
| `index.html` | **Aktueller Stand von Drift** (≈6.700 Zeilen, ≈370 KB). Einzige Datei, die ausgeliefert wird (GitHub Pages). Version steht im `<title>` und als Git-Tag. |
| `CLAUDE.md` | Diese Übergabe. |

**Nur außerhalb des Repos (bei Jan bzw. im früheren Chat), hier nicht prüfbar:**

| Datei | Was |
|---|---|
| `orion-prototype-v011.html` | Eigenständiger Orion-Prototyp, jetzt **mit Kampf-Layer** (siehe 3.3). Ältere Stände v001–v010 existieren. |
| `Drift_Statuspapier_v090.rtf`, `Drift_Lernpapier_v090.rtf` | Ausführliche Doku **Stand v090** (Tabellen, Glossar, JS-Auffrischung für Wiedereinsteiger). Veraltet bei Rezeptnummern und Kosten (siehe unten), sonst gute Orientierung. |
| `cat-prototype-v001.html`, `starship-flythrough.html` | Nebenprototypen (Katzen-Animation, „Dreadnought Flythrough"). Nicht in Drift integriert, kein Integrationsplan bekannt. |
| `outputs/drift-prototype-v084 … v097j.html` | Alte Einzeldateien aus der Sandbox-Zeit. Ab v097j übernimmt die Git-Historie diese Rolle. |

**Ein-Datei-Publishing (verbindlich):** Drift wird als genau eine HTML-Datei ausgeliefert. Keine Nebendateien (kein `manifest.json`, keine Icon-Dateien, keine Audiodateien, keine externen Skripte). Icons stecken als `data:`-URIs im `<head>`. Bewusster Kompromiss: ohne Manifest ist Drift auf Android keine installierbare Standalone-PWA (nur Lesezeichen-Verknüpfung); iOS-Homescreen nutzt `apple-mobile-web-app-capable` + Inline-Icon.

### Aufbau von Drift

Alles in einer Datei; Abschnitte sind mit `// ================= NAME =================` markiert (suche nach `=================`, das ist das Inhaltsverzeichnis). Wichtige Blöcke in Dateireihenfolge:

1. CSS und HTML-Markup (Screens, Bottom-Sheets, Buttons)
2. **NEW RELATIONAL DATA MODEL** (~Z. 810): `KIND_TABLE`, `PARTICLE_TABLE`, `RECIPE_TABLE`, `SPAWN_UPGRADE_TABLE` plus `costForUpgrade()`, `SPAWN_UPGRADE_EFFECTS`
3. ECONOMY UPGRADES (Econ-Button / Füllhorn)
4. PHYSICS CONSTANTS, AMBIENT SPAWNING (`directSpawnPoolNames()`, `spawnDirectParticle()`, `spawnBatch()`)
5. EPOCH 2 (Aufbau, Physik mit 3D-Wolkengravitation, Nebel-Hintergrund, Wolken-Rendering, Zündung, fertige Sonne) und GENERALIZED FUNNEL ADAPTER
6. **EPOCH 3: STELLAR NURSERIES** (~Z. 1920): Baby Cloud → Red Dwarf → Yellow Dwarf → Blue Star
7. **DER KAMPF** (~Z. 2282): erster spielbarer Durchgang im Hauptspiel (Details 3.3)
8. **FLOOR / PILE** (~Z. 3361): Schatzberg-Heightmap
9. QUARK BONDING, ELECTRON CAPTURE, Explosions-/Blitz-Animationen
10. MAIN PHYSICS UPDATE (`update(dtMs)`), SWIPE/RHYTHM, AUTOPILOT STATE MACHINE, INPUT HANDLING, RAP LOOP
11. RENDER, AMBIENT MUSIC (Web Audio, keine Audiodateien), ECHO SHOP, END EPOCH, BREATH AUTO-PILOT, WAKE LOCK, START

Objekte im `particles`-Array sind einfache Literale mit einem `kind`-Feld (`quark`, `electron`, `ion`, `neutron`, `atom`, `molecule`, `quarkDuo`, `atomDuo`, `babyCloud`). `kind`-Listen tauchen an mehreren Stellen wortgleich auf; wer eine neue Partikelart ergänzt, muss alle diese Stellen finden (Grep nach `p.kind ===`). Einzige echte `class` im Code: `SunFlareParticle`.

---

## 3. Aktueller Entwicklungsstand

### 3.1 Die drei Epochen

- **Epoche 1** (Elementarteilchen → Wasserstoff): 12 Kaufstufen (E1.1 … E1.12). Boden ist der **Schatzberg**. Epochenende wartet auf das erste echte H-Atom (`coreRecipeComplete`).
- **Epoche 2** (Atome → Sonnenzündung): 12 Stufen (E2.1 … E2.12). Eine große Baby-Wolke im Bildschirmzentrum mit echter 3D-Gravitation (`CLOUD_DEPTH_Z = 800`), Trichter, Zündungssequenz (5 Stufen, ca. 2 s), fertige Sonne mit Korona/Flares, Nebel-Hintergrund mit dezenter Orion-Andeutung. „Enlarge Capture Area" (E2.11) vergrößert den Trichterradius.
- **Epoche 3** (Sternentstehung): **22 Stufen** (E3.1 … E3.22), seit v097g vollständig granular: die komplette Kette Quarks → Atome → Moleküle wird wiederholt, danach Baby Cloud (E3.17), Red Dwarf (E3.18), Yellow Dwarf (E3.19), Blue Star (E3.20), **Der Kampf** (E3.21), Epoch 3 Complete (E3.22, wartet auf den ersten Blue Star).
  - Sterne entstehen über Duo-Zwischenstufen: 2× gleiche Stufe → Duo (Rezepte 9, 11), 3. Stern vollendet (Rezepte 10, 12). Rezept 10 braucht **kein HD mehr** als Katalysator.
  - Sterne bleiben `STAR_SETTLE_GRACE_MS = 4000` frei beweglich (Partner suchen), danach fallen sie auf den Schatzberg.

### 3.2 Datenmodell (Kurzfassung, Details im Code)

- `PARTICLE_TABLE`: `gameMass` ist seit v097g durchgängig **1,2^n** in Deklarationsreihenfolge, ab Up Quark = 1. `cost = round(20 · gameMass)` überall, keine Einzelwerte. Duos tragen jetzt eigene Namen (`Up-Down Duo`, `Proton-Neutron Duo`, `Helium-Proton Duo`, `Hydrogen Duo`, `H2-D+ Duo`, `Red Dwarf Duo`, `Yellow Dwarf Duo`) und stehen direkt vor ihrem Ergebnis.
- `RECIPE_TABLE`: **Rezeptnummern wurden in v097g neu vergeben** (1 Up-Down Duo, 2 Neutron, 3 Proton, 4 First Hydrogen, 5 D+ Fusion, 6 First Molecule, 7 H2 Formation, 8 Midwife to the Sun, 9–12 Sternstufen). Das Statuspapier v090 benutzt die alte Nummerierung. Nicht verwechseln.
- `SPAWN_UPGRADE_TABLE`: Kaufleiter je Epoche (`epoch`-Spalte, `id` wie `E3.17`). Spalten: `particleRef`, `recipeRef`, `costOverride`, `freeCost`, `waiting`.
  - `costOverride: 0` = gratis (jeweils die erste Stufe jeder Epoche).
  - `costOverride: 'chain'` = ×1,2 auf den Preis der **vorherigen** Stufe derselben Epoche (für Upgrades ohne Partikel/Rezept: E2.11 „Enlarge Capture Area", E3.21 „Der Kampf").
  - `freeCost: true` + `waiting` = „Epoch Complete"-Eintrag, wird frei, sobald die Bedingung erfüllt ist.
- Verbundmassen sind immer die **reine Summe** der Komponenten (kein Sonderfaktor; HDs frühere ×4-Masse ist entfernt).
- Entfernt und nicht wieder einführen: Rezept „Heavy Hydrogen", Rezept „Dissociation" (HeH+ + Elektron), D+ fängt keine Elektronen mehr ein.

### 3.3 Der Kampf (Hauptspiel vs. Orion-Prototyp) — **zwei getrennte Stände**

**Im Hauptspiel (v097g, unverändert seither):** E3.21 „Der Kampf" spielt die 180°-Bildschirmdrehung (`triggerOrionTransition()`) in ein Overlay. Dort steht die Warteschlange der auf dem Schatzberg abgelegten Sterne (`depositedStars`, in echter Ablagereihenfolge, nur der **älteste** ist antippbar). Antippen schießt ihn auf eine senkrechte, pulsierende Barriere; bei Einschlag gibt es einen Blitz. **Es gibt noch keine Kampfauflösung.** `kampfLoop()` ist das Gerüst.

**Im Orion-Prototyp (v011, in dieser Session ergänzt, von Jan noch nicht abgenommen):** ein vollständig neuer Kampf-Layer, der **nicht** ins Hauptspiel portiert ist. Er liegt in der Datei unter „KAMPF-PROTOTYP: Knoten-Besitz, Schatzberg, Formationen, Wellen". Die Dateiversion wurde dabei **nicht** hochgezählt (nächste Änderung also als v012 ausliefern).

Was der Layer tut:
1. **Knotenbesitz** über den bestehenden `ROUTES`-Graphen. `nodeState[name] = { owner: 'antagonist' | 'player' | null, wave }`. Rigel und Saiph starten gegnerisch besetzt und sind von Beginn an Front. Ein Gegnerknoten ist Front, wenn er Rigel/Saiph ist oder an einen Spielerknoten grenzt.
2. **Schatzberg als Standardmischung** (`pile`: Rot/Gelb/Blau, Füllraten `PILE_FILL_RATE`) als Ersatz für den echten geteilten Schatzberg. Rot = Red Dwarf, Gelb = Yellow Dwarf, Blau = Blue Star.
3. **5 Formationen** (`FORMATIONS`): Rote Übermacht (6 Rot), Vorhut (4 Rot + 1 Gelb), Gelbe Linie (3 Gelb + 1 Rot), Dreiklang (R/G/B). Die **fünfte, „Doppelschlag" (2 Gelb + 1 Blau), ist mein Vorschlag als Lückenfüller und von Jan nicht bestätigt.** Der Spieler wählt 2–3 aktive Formationen; das System rotiert durch und **überspringt automatisch unbezahlbare**.
4. **Wellen** mit `profile` `'swarm'` (viele Schwache) oder `'elite'` (wenige Starke) und Volumen/HP. Die Wirksamkeit steht in `EFFECTIVENESS` (Mechabellum-Kernidee, stark vereinfacht): Gelb/Flächenschaden schlägt Schwarm, Blau/Punktschaden schlägt Elite, Rot ist immer verfügbar aber ineffizient.
5. **Idle-Tick** (`COMBAT_TICK_MS = 1200`): pro Tick wird die nächste bezahlbare Formation eingesetzt, bezahlt und der Schaden auf eine Front-Welle angewendet. Fällt die Welle, wird der Knoten Spielerbesitz, `waveTier` steigt, Nachbarknoten werden Gegnerfront mit frischer Welle.
6. **Zusammenbruch** (`collapseFront()`): ist **keine** aktive Formation bezahlbar, besetzt der Gegner endgültig Rigel + Saiph, aller Fortschritt ist weg, der Gegner greift den Spieler **nicht weiter** an. Danach strömen Gegnerpartikel weiter zu diesen Sternen und **verschwinden dort wirkungslos** (Steady-State, keine Eskalation). Kein Game Over.
7. **Umfang bewusst begrenzt:** bis zum Durchbrechen der Barriere. Meissa (Kopfstern, Endgegner) ist **nicht** in `NODE_ORDER` und bekommt nie eine Welle.
8. **Barriere:** Barnard's Loop schneidet geometrisch nur die Kante Mintaka→Bellatrix (rechter Weg). Nur die Eroberung von **Bellatrix** meldet „Barriere durchbrochen". Beteigeuze (linker Weg über Alnitak) hat keine Barriere.
9. Hinweise für den Spieler: Wellenprofil und HP stehen über dem umkämpften Stern, Formationskacheln zeigen Bezahlbarkeit durch Transparenz.

---

## 4. Getroffene Entscheidungen samt Begründung

### Architektur und Prozess

1. **Eine Datei, kein Build.** Ein vollständig sichtbarer Kontext macht die Zusammenarbeit mit einem Sprachmodell deutlich einfacher als zehn Module; Verteilen = Datei auf GitHub Pages legen. Deshalb auch keine ES-Module.
2. **Tabellengetrieben, ohne Ausnahmen.** Hat mehrfach Bugs verhindert, in denen Epochen auseinanderliefen (siehe 4.2 Spawn-Pool). Neue Sonderfälle → neue Tabellenspalte.
3. **Zustandsmaschinen statt Flag-Haufen** (`rapState`/`bapState`, Anzeigemodus wird jedes Mal frisch berechnet).

### Ökonomie und Regeln (v097g–j)

4. **Die erste Kaufstufe jeder Epoche kostet 0** (v097h). Vorher kostete sie Geld bei Score 0, die Epoche war unstartbar. Gilt für alle drei Epochen.
5. **Ein einheitlicher Spawn-Pool für alle Epochen** (v097i). E3 hatte hartkodiert 50/50 Up/Down gespawnt, unabhängig von gekauften Stufen: Down spawnte vor dem Kauf, Strange/Charm/Bottom/Top nie. Ursache: `makeQuarkParticle(forcedFlavor)` kannte nur `'up'`/`'down'` und fiel sonst still auf Zufall zurück. Jetzt: `directSpawnPoolNames()` liest die gekauften Stufen aus `SPAWN_UPGRADE_TABLE`, `spawnDirectParticle()` spawnt daraus, `makeQuarkParticle` versteht alle sechs Flavors. **Verworfen:** pro Epoche eigene handgeschriebene Optionslisten.
6. **`'chain'`-Preis für nicht-massebezogene Upgrades** (v097i). Feste Preise (E2.11 = 483264, E3.21 = 0) passten nicht zur Kette. Jan: einfach exponentiell vom Vorgänger fortführen. Robust gegen Einfügen/Entfernen von Stufen.
7. **Echo-Regel: Ein Rezept bringt Echo genau beim allerersten Abschluss, in jeder Epoche** (v097j). Davor gab es eine Tabelle `echoByEpoch` (dasselbe Rezept gab in E1/E2 Echo, in E3 nicht), die für Spieler unverständlich war. Umgesetzt zentral in `grantRecipeReward()` über das schon vorhandene `achievements.recipesUnlocked` (wird nie zurückgesetzt). Der Goldblitz bleibt, aber nur beim ersten Mal. Das begrenzt Rezept-Echo von vornherein. **Verworfen:** pro-Rezept-pro-Epoche-Flag.
8. **Harter In-Run-Cap auf rezeptverdientes Echo** (Start 10; Echo Cap I–III heben auf 16/24/35). Die Score→Echo-Umrechnung beim Epochenende bleibt **ungedeckelt**. Der Cap reserviert gegen noch fliegende Echo-Tropfen, damit gleichzeitige Abschlüsse ihn nicht überschießen.
9. **Echo-Shop ist nur zwischen Runs offen** (er ist der Prestige-Mechanismus). Debug-Override: „Open Echo Shop (cheat)".

### Schatzberg (v097j)

10. **Allgemeine Regel:** Alle **vollständigen** Atome, Moleküle, Ionen und Sterne gehen in den Schatzberg und hinterlassen dort einen farbigen Punkt **in ihrer echten Hue** (`PARTICLE_TABLE[...].hue`). Quarks und alle Duos fliegen durch und verschwinden. Größe: Atome < Moleküle < Sonnen. Zwischenräume goldschimmernd. Die hartkodierten `RUBY_HUE`/`AQUAMARINE_HUE` (stimmten nicht mal mit den Tabellenwerten überein) sind gelöscht.
11. **Duo-Links werden nicht auf den Boden geklemmt.** Wurde implementiert und eine Runde später nach Praxistest bewusst zurückgenommen. Lehre: einen technisch „korrekten" Fix nach realem Spielen noch einmal mit Jan prüfen.
12. **E1-Echo-Drossel-Zone:** Proton+Elektron→H wird innerhalb von `E1_NO_REACTION_ZONE = 100` px über der Haufenoberfläche nicht finalisiert (Proton sinkt ab). Bewusstes Design gegen Echo-Erzeugung nahe am Boden. **Hinweis:** seit der Einmal-pro-Rezept-Echo-Regel (7.) ist unklar, ob die Zone noch nötig ist; mit Jan klären, nicht eigenmächtig entfernen.
13. **Heightmap statt Partikelhaufen:** abgesetzte Partikel werden zu Säulenhöhen (`pileHeights[]`, Bucket 8 px), danach aus der aktiven Physik entfernt. Ein hoher Haufen kostet pro Frame nichts. `addToPile()` garantiert Mindesthöhe der Einschlagspalte **nach** der Hangglättung (sonst frisst `relaxPileSlope` sie).

### E2-Physik

14. **Rotationskraft ×8** (`ROTATION_PULL_BOOST_MULT`): vorher unsichtbar, jetzt innerhalb von ein bis zwei Sekunden erkennbar, ohne „Kanonenkugel"-Effekt.
15. **Spiralarm-Sprenkel** haben eigene Opazitätskurve und eine Saturnring-artige Vorder-/Hinterseiten-Verdeckung, rein über die Rotationsphase (kein echtes Tiefenmodell).

### Autopilot

16. **RAP-Reshape:** erneutes Greifen einer laufenden Schleife startet die neue Aufnahme mit der **jüngsten Hälfte** der alten und verlangt mindestens **2 Richtungsumkehrungen**. Zu frühes Loslassen setzt die alte Schleife unverändert an der pausierten Phase fort. **Verworfen:** frei aufnehmen ab null (erzeugte kaputte Kurzschleifen).

### Orion / Kampf

17. **Zwei-Bildschirm-Konzept** (Bau- und Kampfbildschirm, 180°-Drehung) statt halb/halb auf einem Schirm (zu knapp).
18. **Wegfindung** über einen festen anatomischen Sterngraphen (`ROUTES`: Fuß → Gürtel → Schulter → Kopf) mit 2D-Distanz, **nicht** echter 3D-Lichtjahr-Distanz (Gürtel-Graben-Problem). Der Schild (Pi Orionis) ist als möglicher Geheimpfad notiert.
19. **z-Werte der Sterne:** ursprünglich war „z = echte Entfernung in Lichtjahren" beschlossen. Im v011-Code sind es stattdessen **Spieltiefen** (Rigel 500, Schultern 600, Gürtel 1000, Meissa 1500; Saiph noch ohne z). Hintergrund: Jan hatte negative Werte notiert, die mit der Perspektivformel „näher" statt „weiter hinten" bedeutet hätten; auf positiv korrigiert. Perspektivformel 1:1 aus Drift: `FOCAL_DISTANCE / max(1, FOCAL_DISTANCE + z)`.
20. **Kampf als Idle-Strategie, nicht als eigene Tabs.** Der Nachschub (Bauschirm) läuft weiter, während man den Kampf beobachtet, und umgekehrt: der **gerade nicht aktiv gesteuerte** Bildschirm läuft mit seiner zuletzt eingestellten Konfiguration autonom weiter. Der Schatzberg ist zwischen beiden **geteilt**.
21. **Gegnerwellen mit wechselndem Profil.** Der Spieler muss das Profil selbst herausfinden (Hinweise vorher und im Kampf) und die passende Formation wählen. Jeder besiegte Gegner wird von einem stärkeren abgelöst, der eine andere Kampflogik mitbringt.
22. **Mechabellum als Vorbild, stark vereinfacht** auf die vorhandenen Rot/Gelb/Blau-Massenstufen: viele kleine Einheiten (Overkill-Verschwendung), Staffelung, Flächen- vs. Punktschaden. Muss in einem Idle-Spiel auf einen Blick verständlich bleiben.
23. **Verworfen: aktives Zurückdrängen des Spielers bei leerem Nachschub.** Jan: der Gegner besetzt dann nur Rigel/Saiph und greift den Spieler nicht weiter an. **Verworfen: Game Over.**
24. **Ziele:** Spieler will den Kopfstern (Meissa) erreichen; Antagonist will die beiden Fußsterne (Rigel, Saiph) dominieren. Jeder Stern hat drei Zustände: umkämpft / Gegner / Spieler.
25. **Antagonist-Konzept (aus früherer Planung):** beginnt mit einem Antimaterie-Asteroiden, sitzt bei Meissa, schöpft nicht aus dem Reservoir des Spielers; dichte Objekte löschen sich vollständig aus, diffuse zerfallen nur teilweise in Fragmente. Barnard's Loop und das Schwarze Loch (nahe Trapez-Haufen) sind als spätere kaufbare Upgrades mit Funktion vorgesehen, bisher nur Darstellung. Im Kampf verdientes Echo soll später im Bauschirm Sterne stärker machen.

---

## 5. Bekannte Bugs und offene Auffälligkeiten

1. ~~**Physik hing an der Bildwiederholrate**~~ (behoben in v097k): `update()` lief einmal pro Frame mit Pro-Frame-Konstanten, 120-Hz-Geräte liefen doppelt so schnell. Jetzt fester 60-Hz-Simulationstakt in `loop()` (`SIM_STEP_MS`, Akkumulator, max. 4 Schritte pro Frame); Spurverblassen und Debris über `frameStepScale` skaliert. Auf 60-Hz-Geräten verhält sich alles exakt wie vorher. Folge auf 120 Hz: Partikel bewegen sich sichtbar in 60-Hz-Schritten (keine Interpolation). Separate rAF-Schleifen (`kampfLoop`, Sonne) sind davon nicht betroffen; die Sonne nutzt bereits `dt`.
1. **E2: Vorbeischwingen fehlt** (Stand v090 als offen geführt, **seitdem nicht neu geprüft**). Partikel stürzen beim ersten Durchgang geradewegs in die Wolke. Diagnose: `CLOUD_DEPTH_Z = 800` ist eine Größenordnung größer als die xy-Trichter-/Einfangradien (zig px), die 3D-Gravitation ist fast reiner z-Zug. Fix-Kandidat: Gravitation gegen einen trichter-skalierten effektiven Radius rechnen statt gegen den rohen 3D-`hypot`. **Bewusst zurückgestellt.**
2. **Zentralkreis-Deckkraft in E2** erreicht rechnerisch 1,0, wirkt aber nicht voll deckend. **Jan will sich das selbst ansehen, bitte nicht ändern.**
3. ~~**`<title>` zeigt „v092"**~~ (behoben in v097k).
4. **Orion-Kampf-Layer (v011), unverifiziert bzw. bekannt unfertig:**
   - Balance ist **ungetestet**. Alle Zahlen (`PILE_FILL_RATE`, `formationPower`, `EFFECTIVENESS`, Wellen-HP) sind Erstschätzungen. Die automatischen Tests liefen mit künstlich unendlichem Nachschub (Fortschritt) bzw. leerem Schatzberg (Zusammenbruch), **nicht** mit realistischem Verlauf. Rechnerisch verbraucht allein „Rote Übermacht" mehr Rot pro Tick als nachfließt, der Zusammenbruch ist also gut erreichbar.
   - Layout: der FPS-Zähler überlappt die Schatzberg-Zeile oben links; Wellen-Labels können auf schmalen Schirmen mit Sternnamen (z. B. Bellatrix/Meissa) kollidieren; die alte Demo-Reihe mit Sonnenobjekten (`E3_OBJECTS`) und der Tap-to-fly-Flug bleiben unabhängig davon aktiv und belegen den unteren Bildschirmbereich.
   - Wellenwürfe nutzen `Math.random()` (nicht reproduzierbar).
   - Der linke Weg (Saiph → Alnitak → Beteigeuze) hat im **Flug-Demo** keine Implementierung (steht im Code kommentiert); im Kampf-Layer sind beide Wege über den Graphen gleichberechtigt.
5. **Hauptspiel-Kampf (v097g)** hat keine Auflösung, nur Blitz. Das ist Stand, kein Bug.
6. **Kleinere offene Enden aus v082:** UI-Stufe GHOST ohne CSS-Entsprechung; Seifenblasen-Linseneffekt.
7. **Tote Referenz Rezept 9 in E2.9** ist behoben (Kommentar im Code), im Statuspapier v090 aber noch beschrieben.

### Technische Fallen (teuer gelernt)

- **Deklarationsreihenfolge:** Eine `const`, die von Code genutzt wird, der beim Skriptstart synchron läuft (auch indirekt über eine Funktionskette, z. B. durch den ersten `resize()`-Aufruf), muss davor deklariert sein. Sonst wirft das Skript einen Temporal-Dead-Zone-`ReferenceError`, bricht komplett ab (auch alle Event-Listener) und von außen wirkt es wie „Start-Button geht nicht / lädt ewig". `node --check` findet das **nicht**.
- **Kraftberechnung mit Vektorbetrag** (`Math.hypot(ax,ay,az)`): Die Achse mit der größten Konstante dominiert und unterdrückt Dynamik auf kleineren Achsen (Ursache von Bug 1).
- **Doppeltes `splice`** im selben Schleifendurchlauf auf gemeinsame Arrays verschiebt Indizes; Reihenfolge sorgfältig wählen.
- **Globales `touchmove`-`preventDefault`** bricht unbeteiligtes Scrollen; Ausnahmen eng fassen (`.sheetScroll`).
- **Glättungs-Pässe fressen Garantien:** Mindestwirkungen nach der Glättung erneut durchsetzen.

---

## 6. Offene Aufgaben und Roadmap

Nichts hiervon ist ohne Jans Freigabe zu bauen (siehe Abschnitt 8).

### Kampf (aktive Linie)

1. Jan prüft den Orion-Kampf-Layer; Balance und Hinweise (Wellenprofil vor/im Kampf) nachschärfen.
2. **Fünfte Formation** festlegen (Jan hat die vierte/fünfte Slot-Belegung offen gelassen; „Doppelschlag" ist nur mein Vorschlag).
3. Nach Abnahme: Kampf-Layer ins **Hauptspiel** portieren (ersetzt das Gerüst `kampfLoop()`), echten geteilten Schatzberg (`depositedStars`) statt Standardmischung anbinden, asynchrones Weiterlaufen des jeweils nicht gesteuerten Bildschirms umsetzen.
4. **Endgegner bei Meissa** mit eigener Lebenspunktfunktion; nur angreifbar, sobald die Spielerfront den Kopfstern erreicht; fällt Meissa an den Spieler, ist diese Gegnerversion besiegt. **Von Jan explizit auf nach der Barriere vertagt.**
5. **„Kleinvieh" im Schatzberg** (nicht nur Sterne, auch kleine Partikel füllen den Berg): Jan hat es selbst auf später vertagt („vergiss das jetzt erstmal"). Nicht anfassen, bis er es wieder aufgreift.
6. Spätere Ausbaustufen laut Orion-Konzept: freischaltbarer Schuss auf das Schwarze Loch, Barriere per Upgrade öffnen, Antagonist mit Antimaterie, Echo aus dem Kampf im Bauschirm ausgeben.

### Wirtschaft und UI (Backlog aus früheren Runden, nicht angefasst)

7. Echo-Upgrade-Stufen umstrukturieren.
8. **Füllhorn** neu denken (5-stufige Rotation).
9. Stärkere visuelle Unterscheidung aktiver/inaktiver Buttons.
10. Rückmeldung beim Echo-Kauf.
11. Rückmeldung, wenn das Echo-Limit erreicht ist.
12. Animation für das E2-Upgrade „Enlarge Capture Area".
13. **Speichersystem** (localStorage) — bisher bewusst nicht vorhanden.
14. E2-Gravitationsfix (Bug 1), sobald Jan ihn freigibt.

### Ältere Ideen (aus der Zeit vor v089, **nicht neu bestätigt, vermutlich teils überholt**)

Metaballs für Wölkchen, Spawner/Attraktor als direkt editierbare Objekte, Neukalibrierung von `IGNITION_K`, Sphären-Farbskalierung, dynamisches Domain-Warping.

---

## 7. Begriffe und Abkürzungen

| Begriff | Bedeutung |
|---|---|
| **RAP** | Rhythm Auto-Pilot. Beim Loslassen einer Wischschleife wird der erreichte ×N-Multiplikator übernommen, die Schleife läuft automatisch. Tempo-Stufen: hektisch <250 ms → ×1, 250–500 → ×2, 500–1000 → ×3, ≥1000 → ×4 (Basisdeckel; Echo-Shop „Patience" hebt auf ×8/×16). |
| **BAP** | Breath Auto-Pilot. Tap auf die ×N-Anzeige, Gerät flach auf dem Bauch, Atembewegung wird über die Gerätekippung (DeviceOrientation) erkannt. Zeitbonus verdoppelt sich nach 4/8/16/32 s, bei Standby eingefroren statt zurückgesetzt. Nutzt Wake Lock. Es gibt einen versteckten BAP-Debug-Screen (Fehlerprotokoll). |
| **RAP-Reshape** | Live-Wiedergriff einer laufenden RAP-Schleife (siehe 4.16). |
| **Score** | Epochen-interne Währung; kauft Spawn-Upgrades; wird beim Epochenwechsel zurückgesetzt. |
| **Echo** | Dauerhafte Meta-Währung (Prestige), überlebt Run-Neustarts, finanziert den Echo-Shop. Zwei Einkommenspfade: (a) Score→Echo-Umrechnung bei „Epoche beenden" (ungedeckelt), (b) rezeptausgelöste Tropfen im Spiel (nur beim allerersten Abschluss eines Rezepts, hart gedeckelt pro Run). |
| **Echo Shop** | Dauerhafte Verbesserungen: Streak Bonus I–III, Patience I–II (RAP-Cap), Deep Breath I–II (BAP-Cap), Object Cap I–III, Spawn Rate I–III, Echo Cap I–III. Nur zwischen Runs offen. |
| **Epoch / E1–E3** | Spielebene. IDs wie `E3.17` = Epoche 3, 17. Stufe der Kaufleiter. |
| **Spawn-Button / Upgrade** | Einziger Kauf-Button der Hauptleiter; kauft jeweils die nächste Stufe aus `SPAWN_UPGRADE_TABLE`. |
| **Econ-Button / Füllhorn** | Zweiter, unabhängiger Kaufslot ab E2 (links vom Spawn-Button). Verbrauchsgüter: temporärer Max-Spawnrate-Schub plus Partikel-Outburst (bis 1,5× Objekt-Cap) am Ende. Kosten 250 · 1,6^n. |
| **Schatzberg / Pile / Heightmap** | Boden als wachsender goldener Haufen (`pileHeights[]`). Beim Kampf Ressource, aus der Formationen bezahlt werden. |
| **Kind** | Partikelart (`quark`, `electron`, `ion`, `neutron`, `atom`, `molecule`, `quarkDuo`, `atomDuo`, `babyCloud`); legt Renderstil, Einfangring, Zerstörungsausbruch fest. |
| **Particle / Recipe** | Zeile in `PARTICLE_TABLE` bzw. `RECIPE_TABLE`. |
| **Duo** | Zwischenstufe aus zwei gebundenen Komponenten (z. B. Up-Down Duo, Red Dwarf Duo); Masse = Summe. |
| **Midwife to the Sun** | Rezept 8: H2 + D+ → HD (+ Proton als Nebenprodukt). |
| **Baby Cloud (BC)** | E3-Gaswolke, wächst pro Einfang um einen diskreten Schritt (`BABY_CLOUD_IGNITION_HITS = 6`), zündet dann zum Red Dwarf. |
| **Red / Yellow Dwarf, Blue Star** | E3-Sternstufen (Rot/Gelb/Blau im Kampf). Hues 8 / 42 / 212. |
| **Funnel / Trichter** | Einfangbereich um Wolke/Baby Cloud; wirkt auf alle Partikelarten. |
| **Attraktor** | Anziehungspunkt der Grundgravitation für freie Partikel. |
| **Vortex / Wirbel** | Wisch-Sog, der verfehlte Partikel mitzieht. |
| **Takt** | Rhythmus der Wischbewegung (Grundlage der Multiplikator-Stufen). |
| **popAt** | Zeitstempel, ab dem ein Partikel als getroffen gilt und nur noch ~260 ms für die Fade-Animation existiert. |
| **bakedIntoPile** | Flag: Partikel wurde in den Schatzberg eingebacken. |
| **costOverride** | Spalte in `SPAWN_UPGRADE_TABLE` (`null`, Zahl, `0`, `'chain'`). |
| **Sonderlocke** | Jans Wort für eine Einzelfall-Mechanik. Unerwünscht. |
| **Der Kampf** | E3.21 und das Kampfsystem (3.3). |
| **Formation** | Feste Zusammenstellung von Rot/Gelb/Blau-Sternen, die aus dem Schatzberg bezahlt und als Einheit in den Kampf geschickt wird. |
| **Barriere / Barnard's Loop** | Pulsierender Bogen (echtes Objekt) auf der Kante Mintaka→Bellatrix; erstes Kampfziel. |
| **Antagonist** | Gegner bei Meissa (Kopfstern). |
| **Meissa / Rigel / Saiph …** | Kopfstern / Fußsterne etc. des Orion-Spielfelds. Spieler startet unterhalb der Füße. |
| **FADED, POOR, READY, OWNED, INERT, TEASE, GHOST** | Konzeptionelle UI-Interaktionsstufen. Echte CSS-Klassen existieren für alle außer GHOST (z. B. `.epochItem.inert`, `.epochItem.tease`). |
| **„coden"** | Jans ausdrückliches Startsignal für eine Implementierungsrunde (siehe 8). |

---

## 8. Konventionen: Zusammenarbeit, Code, Versionierung

### Zusammenarbeit mit Jan

1. **Antworten als eine große nummerierte Liste**, damit Jan Punkte per Nummer referenzieren kann. Etwa 8–14 Punkte, möglichst ohne Unterpunkte. Listen sind immer nummeriert.
2. **Kein Code ohne ausdrückliche Freigabe.** Vor größeren oder mehrdeutigen Aufträgen erst alle offenen Fragen stellen; eine Fragenrunde endet mit einer kurzen nummerierten Liste der offenen Punkte. Ein „ja, so machen" zu einem einzelnen besprochenen Fix ist keine Freigabe für andere Dinge aus demselben Gespräch. Eindeutige Aufträge wie „erstelle die nächste Version" oder eine klar benannte Fehlerkorrektur gelten als Freigabe. (Der Orion-Kampf-Layer in 3.3 wurde nach einer langen Klärungsrunde ohne ausdrückliches „coden" gebaut. Jan hat ihn noch nicht abgenommen; im Zweifel nachfragen.)
3. **Vorschläge klar kennzeichnen und markieren, was von Jan gebraucht wird**, weil er Fragenlisten sonst leicht übersieht. Claude ist echter Mit-Entwickler: Zustand verfolgen, Risiken benennen, proaktiv fragen.
4. **Keine Zwischenstände ausliefern**, bevor die Runde fertig ist. Dateien nicht automatisch öffnen oder Artefakte veröffentlichen (Fokusverlust im laufenden Testspiel), außer Jan will es.
5. **Bei mehreren Änderungen pro Runde:** pro Punkt eine Risikoeinschätzung (was am ehesten einen zweiten Durchgang braucht) und kurze nummerierte Zusammenfassung am Ende.
6. **Vollständigkeitsprüfung vor Codierrunden.** Komplexe visuelle Systeme (Nebel, Metaballs, Kampf) zuerst isoliert als eigener Prototyp bauen, dann integrieren (so entstand Orion).
7. **Token-Budget zählt:** breite automatische Playtest-/Bot-Läufe sind teuer (einer verbrauchte ~20 % von Jans Wochenvolumen). Zwischen Codierrunden schlank testen, aber ohne Abstriche bei der Sorgfalt.
8. Chat mit Jan auf **Deutsch**. Code-Kommentare in `drift-prototype-*.html` sind **Englisch**, in `orion-prototype-*.html` **Deutsch**: jeweils die Sprache der Datei beibehalten.
9. Präzise, technisch, direkt. Jan mag keine generischen oder überengineerten Antworten.

### Versionierung

1. Drift liegt immer als **`index.html`** im Repo. Pro Runde: Version im `<title>` hochzählen (`Drift — Prototype v097<Buchstabe>`, zuletzt **v097k**, nächste **v097l**), committen, Git-Tag mit derselben Versionsnummer setzen (`v097k`). Alte Stände holt man aus der Git-Historie bzw. über den Tag, nicht über Dateikopien. Die Basisnummer bleibt vorerst bei v097.
2. Orion: `orion-prototype-vNNN.html`, fortlaufend dreistellig (nächste: **v012**).
3. Pro Runde eine kurze **nummerierte** Zusammenfassung dessen, was geändert wurde und warum.

### Code-Konventionen

1. **Tabellen vor Code.** Masse, Kosten, Hue, Radius, Dauer gehören in die Tabellen und werden dort gelesen, nie inline neu berechnet. Neuer Sonderfall = neue Spalte. Auch Sonderlogik (Zündung, Baby-Cloud-Wachstum) liest ihre Werte aus den Tabellen.
2. Neue Partikelarten: alle `kind`-Prüfungen im Code finden (Hit-Test, Physik-Schleife, Render, Schatzberg-Filter).
3. `let`/`const` durchgehend (kein `var`), Template-Literale, Arrow-Functions. Kein `import`/`export`.
4. Alle Konstanten, die beim Skriptstart synchron gebraucht werden können, **ganz oben** deklarieren (siehe TDZ-Falle).
5. Epochenspezifische Funktionen tragen das Suffix `E2`/`E3`; Konstanten für Epoche 3 beginnen mit `BABY_CLOUD_`, `STAR_`, `STELLAR_`.
6. Zustand und Sichtbarkeit der UI über CSS-Klassen (`classList`), nicht über Inline-Styles; Listen werden bei Änderung komplett neu aufgebaut.
7. Zeitverhalten: `update()` läuft auf festem 60-Hz-Takt (`SIM_STEP_MS`); Code darin darf mit Pro-Schritt-Konstanten rechnen. Alles, was außerhalb von `update()` pro Frame bewegt oder verblasst (Render-Funktionen, eigene rAF-Schleifen), muss mit `frameStepScale` bzw. `dtMs` skalieren.
8. Neue Mechaniken kommentieren **mit Begründung und Jans Vorgabe** (im Code üblich, z. B. „per feedback"). Abschnitte mit `// ================= NAME =================` markieren.

### Pflicht-Verifikation vor jeder Auslieferung

1. **Syntax:** Inline-Skript extrahieren und prüfen, z. B.
   `python3 -c "import re;h=open('F.html').read();open('x.js','w').write(re.search(r'<script>(.*)</script>',h,re.S).group(1))" && node --check x.js`
2. **Laufzeit (Playwright/Chromium, Pflicht, `node --check` allein reicht nicht):** Seite laden, `#startOverlay` anklicken, **~900 ms warten** (das Overlay blendet 800 ms aus und entfernt sich dann; frühere Eingaben werden still verschluckt und sehen aus wie ein Spielfehler), auf `pageerror` prüfen.
   Chromium liegt unter `/opt/pw-browsers/chromium` (`chromium.launch({ executablePath: '/opt/pw-browsers/chromium' })`; in Claude-Code-Cloud-Sessions reicht `chromium.launch()`), **nicht** `playwright install` ausführen. Viewport z. B. 390×780.
   Bildraten-Check: `requestAnimationFrame` per `addInitScript` auf 120 Hz bzw. 30 Hz umbiegen und `update()`-Aufrufe pro Sekunde zählen; Soll ist ≈60 bei jeder Rate.
3. **Regressionstests nach jeder Runde alle erneut laufen lassen**, nicht nur den neuen. (Die früheren Test-Skripte `check_v097*.js` lagen im Sandbox-Scratchpad und sind **nicht mehr vorhanden**; sie wären neu zu schreiben. Sinnvoll: einen `tests/`-Ordner ins Repo legen.)
4. **Testfallen:**
   - Zeitstempel auf `performance.now()`-Basis im Test zurückdatieren statt wirklich zu warten.
   - Physik/Settling braucht genug simulierte Zeit: ~1500 × `update(16)`, nicht 400 (Gravitation beschleunigt allmählich).
   - Frisch geladene Seite hat kleines `performance.now()` (wenige hundert ms); Zeit-Gates wie `now - last >= 1200` im Test mit großem `now` aufrufen (sonst läuft ein Tick nie, z. B. `updateCombat(performance.now() + 100000)`).
   - Test-Objekte per `createParticleFromTable` haben keine Laufzeitfelder wie `.flavor`; die setzt nur `makeQuarkParticle`.
   - Synthetische Pointer-Events: Canvas-lokale Koordinaten mit `getBoundingClientRect()` in Seitenkoordinaten umrechnen.
5. Neue Features mit einem **gezielten** Test belegen und, wo es um Optik geht, einen Screenshot ansehen.
