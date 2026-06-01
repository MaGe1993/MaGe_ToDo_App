# MaGe ToDo — Anleitung

[English](README.md) | [Deutsch](README.de.md)

MaGe ToDo ist eine schlanke Windows-Desktop-App, um die tägliche Arbeit über mehrere Projekte hinweg als Kanban-Boards zu organisieren. Sie verfolgt Fälligkeitsdaten mit relativen Bezeichnungen, berechnet je Projekt einen Gesundheitswert und zeigt alles auf einem Dashboard mit Kennzahlen, Trend-Sparklines und Diagrammen. Sie läuft vollständig offline — die Daten liegen neben der App, es gibt kein Konto und kein Backend. Ein eingebauter MCP-Server erlaubt KI-Agenten (z. B. Claude Code), die App lokal zu lesen und zu steuern. Erstellt mit Tauri, Rust und SolidJS; Installation über ein kleines NSIS-Setup.

Eine persönliche To-do-App für mehrere Projekte und die tägliche Planung. Sie wird über einen Setup-Assistenten (Deutsch oder Englisch) installiert, speichert alle Daten lokal auf Ihrem Rechner und benötigt weder Konto noch Internetverbindung. Die App öffnet sich auf dem Dashboard und ist mit einigen Beispielprojekten vorbefüllt, damit Sie sie sofort ausprobieren können. Diese Anleitung erklärt die Bedienung: was jede Schaltfläche tut und was jeder Bildschirm bietet.

## 1§ Das Fenster

Die Titelleiste oben enthält von links nach rechts: das App-Logo und den Namen sowie eine Reihe von Schaltflächen.

- Einstellungen (Zahnrad) — öffnet die Einstellungsseite.
- Dokumentation (Fragezeichen) — öffnet ein Menü: Über, Anleitung, Dokumentation, MCP-Server, Lizenz. Zum Öffnen darüberfahren oder klicken.
- Unterstütze mich (Herz) — öffnet die Ko-fi-Seite des Entwicklers im Browser.
- Minimieren, Maximieren/Wiederherstellen, Schließen — die üblichen Fenstersteuerungen. Ein Doppelklick auf eine freie Stelle der Titelleiste schaltet ebenfalls das Maximieren um.

## 2§ Projekte und die Seitenleiste

Die linke Seitenleiste listet Ihre Projekte auf. Die Reihenfolge hat Bedeutung: Die Position eines Projekts ist seine Priorität, oben = 1. Durch Ziehen eines Projekts nach oben oder unten ändern Sie die Reihenfolge — eine Einfügelinie zeigt, wo es landet.

- Dashboard — der oberste Eintrag öffnet die Übersicht über alle Projekte (siehe 5§).
- Projekte filtern — das Suchfeld filtert die Seitenleiste während der Eingabe nach Namen. Bei eingeklappter Seitenleiste auf die Lupe klicken, um sie aufzuklappen und das Feld zu fokussieren.
- Projekt öffnen — anklicken, um das zugehörige Board anzuzeigen.
- Neues Projekt (Plus) — fügt ein neues Projekt hinzu und öffnet es.
- Seitenleiste einklappen — die Pfeil-Schaltfläche oben verschmälert die Seitenleiste auf reine Symbole.

## 3§ Das Board

Ein Board zeigt ein Projekt als Satz von Listen (Kanban-Spalten). Die letzte Liste ist immer `Done` (Erledigt) und kann nicht entfernt werden.

### 3.1§ Board-Kopfzeile

Die Kopfzeile zeigt das Projekt-Emoji und den Namen, die Prioritätszahl, eine Gesundheits-LED, einen Fortschrittsbalken sowie zweizeilige Zähler für heute fällige und überfällige Aufgaben. Rechts liegen die Board-Steuerungen:

- Nach Präfix gruppieren — gruppiert Aufgaben in jeder Liste nach ihrem ersten Wort, wenn es sich wiederholt.
- Nach Fälligkeitsdatum gruppieren — gruppiert Aufgaben stattdessen nach Fälligkeitsdatum.
- Alle auf-/zuklappen — öffnet oder schließt alle Gruppen auf einmal.
- Suchen — filtert Aufgaben im gesamten Board nach Text.
- Liste hinzufügen — fügt eine neue Liste hinzu (direkt vor `Done` eingefügt).
- Projekt bearbeiten (Stift) — schaltet den Inline-Bearbeitungsmodus ein (siehe 3.4§).
- Projekt löschen (Papierkorb) — löscht das Projekt nach einer Bestätigung. Deaktiviert, wenn nur noch ein Projekt übrig ist.

### 3.2§ Aufgaben

Jede Aufgabe trägt einen Text, ein optionales Fälligkeitsdatum und nach Abschluss ein Erledigungsdatum.

- Aufgabe hinzufügen — in das Feld `Eintrag hinzufügen…` am unteren Rand einer Liste tippen und Enter drücken. Optional vorher im Datumsfeld daneben ein Fälligkeitsdatum wählen.
- Datumsfeld — das Datum direkt eintippen (es formatiert sich während der Eingabe selbst, z. B. wird aus `12032026` der Wert `12.03.2026`) oder auf das Kalendersymbol klicken, um es auszuwählen. Das Format folgt Ihrer Einstellung.
- Aufgabe abschließen — auf den Häkchenkreis klicken oder die Aufgabe in die Liste `Done` ziehen. Damit wird das heutige Datum als Erledigungsdatum gesetzt.
- Aufgabe wieder öffnen — aus `Done` herausziehen; das Erledigungsdatum wird gelöscht.
- Aufgabe bearbeiten — anklicken, um Text und Fälligkeitsdatum inline zu bearbeiten. Wird der Text geleert, wird die Aufgabe gelöscht.
- Aufgabe verschieben — in eine andere Liste ziehen; eine Einfügelinie zeigt die Ablageposition.

### 3.3§ Fälligkeits-Bezeichnungen

Das Fälligkeitsdatum einer Aufgabe wird relativ angezeigt, wenn es nahe ist: `Heute`, `in N Tagen` oder `+N Tage` bei Überfälligkeit. Weiter entfernte Daten erscheinen als absolutes Datum im gewählten Format. Heute ist in der Warnfarbe hervorgehoben, überfällig in der Fehlerfarbe.

### 3.4§ Bearbeitungsmodus

Die Stift-Schaltfläche verwandelt Projekttitel, Emoji sowie jeden Listentitel und jedes Listen-Emoji in bearbeitbare Felder. Jede Liste erhält eine Löschen-Schaltfläche (außer `Done`). Erneut auf den Stift klicken, um zu beenden.

## 4§ Umsortieren per Ziehen und Ablegen

Drei Dinge lassen sich per Ziehen umsortieren, jeweils mit einer Einfügelinie als Anzeige:

- Projekte in der Seitenleiste (ändert die Priorität).
- Listen innerhalb eines Projekts (die Liste `Done` bleibt immer zuletzt).
- Aufgaben zwischen Listen.

## 5§ Das Dashboard

Das Dashboard fasst alle Projekte zusammen. Nichts hier wird separat gespeichert — jeder Wert wird aus Ihren Aufgabendaten neu berechnet.

### 5.1§ Obere Reihe

- Gesamtgesundheit — eine Anzeige des durchschnittlichen Projektstatus (alles gesund ist grün, alles kritisch ist rot).
- Ohne Fälligkeitsdatum — ein Ring, der zeigt, wie sich die offenen Aufgaben ohne Fälligkeitsdatum auf die Projekte verteilen; die Mitte zeigt die Gesamtzahl.
- Uhren — eine große lokale Uhr rechts fixiert, plus zusätzliche Weltzeituhren, die Sie in den Einstellungen hinzufügen.

### 5.2§ Kennzahlenleiste

Sechs Kennzahlen, jeweils mit einem Tagesvergleichs-Pfeil und einer 7-Tage-Sparkline: Abschluss, Gesamtaufgaben, Überfällig, Heute fällig, Gesamtverzug, Durchschnittlicher Verzug. Pfeil und Sparkline sind danach eingefärbt, ob die Veränderung gut oder schlecht ist, nicht bloß nach ihrer Richtung.

### 5.3§ Diagramme

Ein Bereich mit Diagrammauswahl und gemeinsamem Datumsbereich (1M / 3M / 1J / MAX):

- Aufgaben — ein gestapeltes Balkendiagramm anstehender offener Aufgaben nach Fälligkeitstag, eine Farbe je Projekt.
- Fortschritt — der Abschlussprozentsatz jedes Projekts über die Zeit.
- Priorität — ein Rang-Verlaufsdiagramm der Projektpriorität über die Zeit. Die Linie eines Projekts beginnt an dem Tag, an dem es zuerst auftauchte; der heutige Rang folgt der aktuellen Reihenfolge in der Seitenleiste.

### 5.4§ Projektkarten

Eine Karte je Projekt mit Gesundheits-LED, Fortschrittsbalken und einer Aufschlüsselung der offenen Aufgaben je Liste (Abzeichen werden warnend oder fehlerhaft, wenn eine Liste heute fällige oder überfällige Aufgaben enthält). Ein Klick auf eine Karte öffnet das jeweilige Projekt.

## 6§ Einstellungen

Die Einstellungen erreichen Sie über das Zahnrad in der Titelleiste. Abschnitte:

- Darstellung — Design (Dunkel / Hell / System), Akzentfarbe (Farbfeld plus HEX/RGB/HSL-Felder) und ein Animationsschalter.
- Sprache & Region — Oberflächensprache (Englisch / Deutsch), Zeitzone, Datumsformat, Zahlenformat und die Liste der Dashboard-Uhren (benannte Weltzeituhren hinzufügen; jede Zeile hat eine Löschen-Schaltfläche, eine Bezeichnung und eine Zone).
- Benachrichtigungen — ein Hauptschalter für Desktop-Benachrichtigungen, ein Ton-Schalter (spielt einen sanften Klang), Ruhezeiten und Schalter je Ereignis: Heute fällige Aufgaben, Überfällige Aufgaben, Tageszusammenfassung, Aufgabe abgeschlossen. Jedes zeitgesteuerte Ereignis hat eine Uhrzeitauswahl, wann es auslöst (Standard: heute fällig 09:00, überfällig 10:00, Tageszusammenfassung 14:00). Der Abschluss einer Aufgabe meldet sofort; die zeitgesteuerten lösen einmal täglich zur eingestellten Zeit aus. Jede Benachrichtigung berücksichtigt den Hauptschalter, ihren eigenen Schalter und die Ruhezeiten.
- Projekte — konfiguriert die Projektgesundheit. Abgeschlossene Aufgaben zählen nicht zur Gesundheit. Die treibende Größe ist das Risiko = der schlechtere von zwei Prozentwerten: überfällige offene Aufgaben gegenüber allen offenen Aufgaben sowie der Gesamtverzug der offenen Aufgaben gegenüber der geplanten Projektspanne. Zwei Steller legen die Schwellen fest (Standard 5 und 20): Risiko bis zur ersten ist gut, über der zweiten ist Fehler, dazwischen ist Warnung. Die Karte zeigt die drei Zustände mit ihren LEDs.

## 7§ Dokumentationsseiten

Das Fragezeichen-Menü in der Titelleiste öffnet:

- Über — App-Name, Version und Danksagungen.
- Anleitung — diese Anleitung.
- Dokumentation — die technische Projektspezifikation.
- MCP-Server — wie ein externer KI-Agent sich mit dieser App verbindet.
- Lizenz — der Lizenztext.

## 8§ Steuerung durch KI-Agenten

Jede laufende Instanz stellt zusätzlich einen lokalen Steuerungsserver bereit, damit ein externer KI-Agent (etwa Claude Code) Ihre Projekte lesen und die App steuern kann — Aufgaben hinzufügen und abschließen, Projekte umsortieren, Einstellungen ändern und mehr. Er lauscht nur auf Ihrem eigenen Rechner und erfordert ein Token. Einzelheiten finden Sie auf der MCP-Server-Seite in der App.
