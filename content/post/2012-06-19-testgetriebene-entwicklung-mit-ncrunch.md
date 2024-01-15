---
title: "Testgetriebene Entwicklung (mit NCrunch)"
date: "2012-06-19"
categories: 
  - "c-net"
tags: 
  - "ncrunch"
  - "tdd"
  - "testgetrieben"
---

Beim Testgetriebenen Entwickeln (TDD) wird ein Test-First-Ansatz verfolgt, dass bedeutet, dass erst der Test erstellt wird. Danach wird der Quelltext geschrieben, welcher notwendig ist, damit der Test erfolgreich durchläuft (umgangssprachlich: „grün wird“). Beim Entwickeln mittels TDD sollte nichts programmiert werden, was nicht durch einen Test vorher notwendig wird. Das führt zu einer hohen Abdeckung vom produktiven Quelltext mit Tests sowie zu einem testbaren Entwurf.

Die Vorteile sind u.a.

- Dokumentation der Software (in den Test ist ersichtlich, wie die Klassen zu verwenden sind)
- Definierte Erwartungshaltung an die Implementierung (Refaktorisieren ist ohne Gefahr möglich, denn die Tests würden fehlschlagen, falls man etwas falsch gemacht hat).
- Wartbarkeit (testbarer Quelltext ist im Allgemeinen auch einfach lesbar u. verständlich)
- Schnelligkeit (die Tests sind UNIT-Tests das bedeutet, dass sie nur eine Einheit wie, eine Klasse oder Methode testen, alle Abhängigkeiten werden durch Techniken wie Dependency Injection, Inversion Of Control o.ä. entkoppelt)

Damit testgetriebene Entwicklung funktioniert und Spaß macht, muss das Arbeiten mit Tests einfach sein. Dafür gibt es eine Reihe von Erweiterungen für das Visual Studio.

Eine sehr bekannte Erweiterung ist **TestDriven.NET** ([http://www.testdriven.net/](http://www.testdriven.net/)). Sie ermöglicht die Ausführung von Tests und die Berechnung von Abdeckung des Quelltextes mit Tests (Coverage) aus dem Visual Studio. TestDriven.NET kann mit vielen verschiedenen Unit-Test-Frameworks zusammenarbeiten und ist für den privaten Gebrauch kostenlos.

Eine noch eher Unbekanntes Erweiterung ist **NCrunch** ([http://www.ncrunch.net/](http://www.ncrunch.net/)), es befindet sich momentan in Beta-Stadium und ist in dieser Phase vollkommen kostenlos. NCrunch überwacht den produktiven Quelltext sowie die Quelltexte der Tests. Sobald es zu einer Änderung kommt, werden die Tests automatisch ausgeführt. Es bietet darüber hinaus noch eine Reihe anderer interessanter Funktionalitäten:

- Im Quelltext-Editor werden pro Zeile Punkte am linken Rand angezeigt: _Schwarz_ bedeutet, dass die Zeile nicht getestet ist. _Grün_ bedeutet, dass die Zeile getestet ist und der Test erfolgreich. _Rot_ bedeutet, dass die Zeile getestet ist und der Test fehlgeschlagen. Ein _Rotes X_ markiert die Zeile durch die, der entsprechende Test fehlschlägt.
- Über die Punkte kann NCrunch anzeigen, welche Tests die entsprechende Quelltext-Zeile testen, darüber hinaus können die Tests oder die Fehlersuche (Debuggen) direkt gestartet werden.
- Grüne Punkte, die in der Mitte Gelb sind, deuten auf ein Performance-Problem hin, da die Ausführung der Quelltext-Zeile länger gedauert hat als andere.
- Die “Risk/Progress-Bar” ist besonders hilfreich, man erkennt u.a. sehr schnell, ob alle Tests noch erfolgreich sind. Oder ob sich der Quelltext z.Z. nicht übersetzen lässt.
- Über den Menü-Eintrag „NCrunch-> Metrics“ bekommt man einen Überblick der Quelltextabdeckung mit Tests angezeigt.

Im täglichen Gebrauch läuft NCrunch stabil, aber Arbeiten vielen mir nur zwei Punkte auf:

1. Manchmal sind die Background Threads eingefroren und eine aktive Referenz auf die Test-Assemblies gehalten. Durch das Beenden der entsprechenden Threads (z.B. über Process Explorer) wurde das Problem gelöst.
2. Nach dem erneuten Öffnen eines Projektes, zeigt die Coverage - Metrik falsche zahlen an. Direktes Aufrufen der Tests führte bei mir nicht zu einer Besserung. Sobald jedoch die automatische Erkennung von Quelltext-Änderungen die Tests ausgeführt hat, sind die Zahlen in der Metrik auch wieder korrekt.

Diese zwei Probleme stören aber nicht weiter und das Entwickeln mit NCrunch macht viel Spaß. Allen Interessierten empfehle ich, sich das Video auf der NCrunch Homepage anzuschauen.
