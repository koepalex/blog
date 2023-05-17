---
title: "Pipe aus eigenen Programmen nutzen"
date: "2013-10-13"
categories: 
  - "c-net"
tags: 
  - "cmd"
  - "pipe"
  - "81301"
---

Mittels der von Unix Systemen bekannten Pipe (_|_) ist es auch unter Windows (via CMD-Line) möglich einzelne Kommandos zu verbinden. Die Pipe ermöglicht es die Ausgabe vom dem vorherigen Kommando direkt als Input des aktuellen Kommandos zu verwenden. Beispielsweise den Inhalt einer Datei einem Skript zu übergeben: `type input.md | perl Markdown.pl > output.html`

Logisch gesehen Ersetzt der Inhalt der Pipe ein einzelnes Konsolen-Argument. Um diese Funktionalität auch in den eigenen (Konsolen-) Programme verwenden zu können, sind im Allgemeinen nur zwei Erweiterungen notwendig.

### 1

Das Lesen der Eingabedaten aus der Standardeingabe: `string pipedData = Console.In.ReadLine (); if (string.IsNullOrEmpty (pipedData)) { Console.WriteLine ("no piped argument"); } else { dataModel.Input = pipedData; }`

### 2

Eine Veränderung in der Logik welche die Konsolen-Argumente interpretiert. Die Logik muss Wissen dass eines der Argumente (wahrscheinlich das Hauptargument) nun optional ist. Da der Wert des Argumentes aus der Pipe gelesen wird.

Tipp: es empfiehlt sich die Verwendung einer Open-Source Bibliothek wie der [Command Line Parser Library](http://commandline.codeplex.com). Bei dieser muss um ein Argument als Optional zu Kennzeichnen, der Attributparameter **Required** auf _false_ gesetzt werden.
