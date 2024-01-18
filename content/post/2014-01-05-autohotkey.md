---
title: "AutoHotkey"
date: "2014-01-05"
categories: 
  - "miscellaneous"
tags: 
  - "autohotkey"
  - "markdown"
---

Durch Scott Hanselman’s Blog Artikel [2014 Ultimate Developer and Power Users Tool List for Windows](http://www.hanselman.com/blog/ScottHanselmans2014UltimateDeveloperAndPowerUsersToolListForWindows.aspx). bin ich auf **[AutoHotkey](http://www.autohotkey.com)** aufmerksam geworden. Es biete wie AppleScript unter Mac OS X, die Möglichkeit verschiedene (lästige) Aufgaben zu automatisieren bzw. Programme um Funktionalitäten zu erweitern.

* * *

Damit könnte man zum Beispiel einen beliebiges Programm mit einer Eingabemaske um _Snippets_ erweitern. Als Besonderheit können AutoHotkey-Skript auch in eine ausführbare Datei kompiliert werden und somit auf Rechnern ohne AutoHotkey verwendet werden.

Als praktisches Beispiel für ein AutoHotkey-Skript finde ich die Hotkeys für die Markdown-Syntax sinnvoll. Es gibt ein paar Punkte welche beim Markdown schreiben für Frust sorgen:

- Leerzeilen vor und nach Aufzählungen
- Quelltext Formatierung

Um Probleme in Zukunft zu vermeiden, habe ich das Github Projekt [AutoHotkeyskript für Markdown](https://github.com/koepalex/autohotkey-markdown) erstellt. Das dort verfügbare Skript bietet aktuell folgende einfache Hotkeys:

- alt + i (Beginnt einen Kursiv dargestellten Text)
- alt + b (Beginnt einen Fett dargestellten Text)
- alt + c (Beginnt Bereich zur Quelltexteingabe)
- alt + q (Beginnt ein Zitat)
- alt + . (Start einer nicht sortierten Liste)
- alt + , (Start einer sortierten Liste)
- alt + t (Füge 4 Leerzeichen ein)
- alt + - (Füge Horizontale Line ein)

Zusätzlich gibt es noch einige verbesserte Hotkeys, welche eine Oberfläche benutzen zur besseren Benutzersteuerung:

- alt + l (Starte Dialog zum ein Link hinzuzufügen)
- alt + p (Starte Dialog um ein Bild hinzuzufügen)
- alt + # (Starte den „Code Beautifier“)

Der **Code Beautifier** sucht alle `<code></code>`Sektionen im generierten HTML-Dokument und führt einige Ersetzungen durch:

1. Ersetze alle Tabs durch 4 Leerzeichen
2. Ersetze alle Leerzeichen durch `&nbsp;`
3. Ersetze alle Zeilenende durch `<br />`

Viel Spaß beim Einsatz von AutoHotkey.
