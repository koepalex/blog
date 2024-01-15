---
title: "Neues vom GC in .NET 4.0 und 4.5"
date: "2013-10-21"
categories: 
  - "c-net"
tags: 
  - "gc"
  - "largeobjectheapcompactionmode"
  - "loh"
  - "lowlatency"
  - "soh"
  - "sustainedlowlatency"
---

Heute möchte ich etwas über Erneuerungen im .NET 4.0 / 4.5 sprechen. Das genannte bezieht sich auf den Workstation Garbage-Collector. Die Informationen für den Server Garbage-Collector entnehmen sie bitte den Links. Beginnen wir jedoch zunächst mit einer kurzen Auffrischung.

### Wiederholung

Das .NET Framework unterteilt seinen Heap in verschiedene Generationen.

- In der **Gen0** werden fast alle Objekte erstellt. Die Anfangsgröße beträgt rund 256KB.
- In der **Gen1** werden Objekte gespeichert die _eine_ Garbage-Collection überlebt haben. Die Anfangsgröße beträgt rund 2 MB.
- In der **Gen2** werden Objekte gespeichert die _mehr als eine_ Garbage-Collection überlebt haben Die Anfangsgröße beträgt 10 MB.

Die soeben genannten Anfangsgrößen können zur Laufzeit variiert werden. Am stärksten wird sich i.A. die Größe der Gen2 verändern (wachsen), da die Gen2 als _Endlager_ für alle länger benötigten Objekte dient.

Die Gen2 teilt sich logisch in zwei Bereiche auf:

- **SOH** der _Small Object Heap_ dieser enthält alle Objekte die durch mindestens zwei Garbage-Collections hierher verschoben wurden.
- **LOH** der _Large Object Heap_ welcher alle Objekte mit einer Größe größer gleich 85 KB enthält.

Eine Garbage-Collection läuft in zwei Schritten ab. Die erste Phase _Mark_ markiert alle Objekte die nicht mehr benötigt werden. Im zweiten Schritt _Sweep_ wird der Speicher der markierten Objekte freigegeben. Um zu entscheiden, ob ein Objekt noch benötigt wird, sucht der GC Wurzeln _Roots_. Wurzeln können u.a. statische Felder, Methodenparameter, lokale Variablen oder CPU Register sein. Hat ein Objekt (oder eine Gruppe von Objekten) keinen Verweis auf eine Wurzel ist es aus dem Programmcode nicht mehr zugreifbar und kann aus dem Speicher entfernt werden. Sollte ein Objekt eine Garbage-Collection überleben und noch nicht in Gen2 sein, wird es um eine Generation verschoben.

Zu einer Fragmentierung der 0. und 1. Generation kann es bei diesem Entwurf nicht kommen. Denn ein Objekt wird entweder verschoben (von 0 nach 1 bzw. von 1 nach 2) oder gelöscht.

Der SOH (der Gen2) kann über die Zeit fragmentieren, deswegen wird in der Sweep-Phase der Heap zusammen gepackt (komprimiert/defragmentiert). Für eine solche Garbage-Collection wird der Main-Thread vom Garbage-Collector-Thread unterbrochen. **Der LOH kann über die Zeit auch fragmentieren hier wird jedoch keine Defragmentierung durchgeführt.** Das kann zu Speicherknappheit (OutOfMemory-Abstürzen) in .NET Programmen führen, obwohl insgesamt genügend freier Speicher vorhanden ist.

### .NET 4.0

Die Gen0 und Gen1 arbeitet der Garbage-Collector unverändert ab. Die Gen2-Collection läuft jetzt jedoch meist im Hintergrund. Diese **nicht blockierende Garbage-Collection** hat die Einschränkung, dass der SOH dabei nicht defragmentiert wird. Für eine Defragmentierung ist eine blockierende Garbage-Collection notwendig. Die Zeit in welcher die Garbage-Collection ein Programm blockiert wurde dadurch drastisch reduziert. [Mehr zur Hintergrund Garbage-Collection](http://msdn.microsoft.com/en-US/library/ee787088.aspx)

* * *

Eine weitere Optimierung stellt der **LowLatency** Modus dar, dieser unterbindet jegliche Gen2-Collection _komplett_. Nur eine _low memory_ Notifikation des Betriebssystems oder ein expliziter Aufruf von GC.Collect mit 2 als Parameter haben eine Gen2-Collection zur Folge. Das aktivieren des LowLatency-Modus ist nur für besonders Zeitkritische Aufgaben zu empfehlen und sollte deshalb nur vorübergehend genutzt werden. Aktiviert wurde der neue Modus mittels:

`GCSettings.LatencyMode = GCLatencyMode.LowLatency;`

[Weiterführende Dokumentation zum LowLatency Mode](http://msdn.microsoft.com/en-us/library/bb384202.aspx)

### .NET 4.5

Der Latency-Modus wurde um die Option **SustainedLowLatency** erweitert, welche im Gegensatz zum _LowLatency_ zusätzliche Hintergrund Gen2-Collections durchführt. Der neue Modus ist dazu geeignet das Programm über einen längeren Zeitraum im LowLatency laufen zulassen. Der SustainedLowLatency Modus wurde übrigens von Microsoft auf für .NET 4.0 mit dem [Update 3](http://support.microsoft.com/kb/2600211) nachgeliefert.

* * *

Das Garbage-Collector-Team hat auch etwas für den LOH getan unter anderem das finden von freien Speicherblöcken bei einer Allokation verbessert:

> In preparation for LOH allocation requests, the GC builds up a list of free (available) memory blocks after a collection. As part of an allocation, the GC consults this these free memory blocks one by one to determine if any block in the list will satisfy the allocation. If a given memory block is a candidate, it is used for the allocation. In earlier releases of the .NET Framework, once a memory block was rejected as a candidate for an allocation, it was removed as a candidate for subsequent allocations.

* * *

Den größten Nutzen werden große Programme wohl davon haben, dass es jetzt möglich ist den **_LOH zu Komprimieren_**! Dazu ist nur folgender Quelltext notwendig:

`GCSettings.LargeObjectHeapCompactionMode = GCLargeObjectHeapCompactionMode.CompactOnce; //force Gen0 to Gen2 blocking GC GC.Collect();`

* * *

Für weitere Verbesserungen des wie **Allokation von Objekten über 2 GB** siehe [The .NET Framework 4.5 includes new garbage collector enhancements for client and server apps](http://blogs.msdn.com/b/dotnet/archive/2012/07/20/the-net-framework-4-5-includes-new-garbage-collector-enhancements-for-client-and-server-apps.aspx).
