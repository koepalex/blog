---
title: "Debugging von Performance Problemen in .NET"
date: "2013-11-27"
categories: 
  - "c-net"
tags: 
  - "perfvew"
  - "sos"
  - "sosex"
  - "windbg"
---

Die Analyse von Speicherproblemen ist eine Aufgabe die bei großen .NET Anwendungen häufiger vorkommt. In C++ wurde gesucht, wer welche Speicherblöcke angefordert und nicht wieder freigegeben hat und im .NET Umfeld wird eben gesucht warum der Gabarge-Collector den Speicher nicht freigeben kann. Oder man sucht warum einige Benutzeraktionen besonders lange benötigen. Für die Analyse gibt es eine ganze Reihe guter kommerzieller Programme, auf diese möchte ich jedoch nicht eingehen, sondern ein paar kostenlosen Alternativen vorstellen.

### [PerfView](http://www.microsoft.com/en-us/download/details.aspx?id=28567)

Dieses Programm von Microsoft ist klein, schnell und leistungsfähig. Zum einen bietet es die Möglichkeit mittels ETW (Event Tracing for Windows) Problemen in der Laufzeit (CPU, I/O, GC) auf die Spur zu kommen. Zum anderen kann es Speicherabbilder von Programmen erzeugen und diese **Vergleichen**! Ausführliche Informationen gibt es bei [Channel9](http://channel9.msdn.com/Series/PerfView-Tutorial) oder auf [Vance Morrison’s Webblog](http://blogs.msdn.com/b/vancem/).

### [WinDbg](http://msdn.microsoft.com/en-us/library/windows/hardware/ff551063.aspx) mit „_S_on _O_f _S_trikes“ Erweiterung

Microsoft liefert die sos.dll mit dem .NET Framework aus. Sie ermöglicht den WinDbg dazu managed Code zu debuggen. Für die den Themenkomplex _WinDbg und SOS_, gibt es viele gute Artikel/Tutorials von welchen ich zwei empfehlen möchte:

1. Der Blog von [Tess Ferrandez](http://blogs.msdn.com/b/tess/default.aspx?PageIndex=1&PostSortBy=MostRecent) beispielsweise mit den Artikeln „[.NET Finalizer Memory Leak](http://blogs.msdn.com/b/tess/archive/2007/10/19/net-finalizer-memory-leak-debugging-with-sos-dll-in-visual-studio.aspx)“ und „[New commands in SOS for 4.0](http://blogs.msdn.com/b/tess/archive/2010/03/01/new-commands-in-sos-for-net-4-0-part-1.aspx?Redirected=true)“.
2. Ein [Blogeintrag von Rex Tang](http://blogs.msdn.com/b/rextang/archive/2007/07/24/4026494.aspx) in welchem er weitere lesenswerte Artikel zu diesem Thema auflistet.

Auf der MSDN Webseite befindet sich die [Befehlsreferenz](http://msdn.microsoft.com/de-de/library/bb190764.aspx).

### WinDbg mit „[_SOS EX_tension](http://www.stevestechspot.com)“ Erweiterung

Die von Steve Johnson geschriebene sosex.dll bietet viele nützliche Erweiterungen die dass managed Code debugging im WinDbg deutlich vereinfachen. Beispielsweise kann die SOSEX mittels Befehl `!bhi` einen HeapIndex erstellen lassen. Dieser Index enthält alle benötigten Informationen über die Objekte auf dem Heap. Um anschließend z.B. alle GC-Roots eines Objektes zu finden (`!mroot <adr> -all`) muss die SOSEX nicht das gesamte Speicherabbild analysieren sondern nur den HeapIndex. Dieses Vorgehen funktioniert deutlich schneller als die Alternative der SOS (`!gcroot -all <adr>`), welche jedes mal das Speicherabbild durchsucht.

Eine andere Eigenschaft ist das die SOSEX [.prefer\_dml 1](http://www.wintellect.com/blogs/jrobbins/a-cool-windbg-sos-hidden-feature) verwendet d.h. dass Links in die Ausgabe des WinDbg eingebettet werden. Dadurch wird einem häufig das kopieren bzw. abtippen von teilen der Ausgabe erspart.

Weitere Funktionalitäten der SOSEX sind

- Anzeige von managed Locks und native CriticalSections (`!mlocks`)
- Anzeige von wartenden Thread inkl. dem worauf sie warten (`!mwaits`)
- Anzeige von Objekten in der Finalizer Queue (`!finq`)
- Anzeige von Objekten in der F-Reachable Queue (`!frq`)

_Lesen sie [hier](http://blogs.microsoft.co.il/sasha/2012/07/28/finalization-queue-or-f-reachable-queue-sosex-not-so-new-commands/) die Beschreibung des Unterschiedes zwischen Finalizer Queue und F-Reachable Quere._

Tip: Herr Johnson aktualisiert SOSEX regelmäßig, auch wenn er dazu keine Notiz auf seinen Blog hinterlässt. Deshalb lohnt es sich immer mal wieder das Zip-Archiv herunter zuladen und zu prüfen ob die enthaltene SOSEX neuer ist als die eigene.
