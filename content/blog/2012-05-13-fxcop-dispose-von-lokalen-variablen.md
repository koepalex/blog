---
title: "FxCop Dispose von lokalen Variablen"
date: "2012-05-13"
categories: 
  - "c-net"
tags: 
  - "custom-rule"
  - "dispose"
  - "fxcop"
  - "local"
  - "variable"
---

Die Freigabe von Ressourcen (verwaltet und nativ) wird im .Net mittels Implementierung der IDisposable Schnittstelle (bzw. Anwendung des IDisposable Patterns realisiert). Der Aufruf der Dispose-Methode ist so wichtig, dass Microsoft spezielle FxCop Regeln implementiert hat (FxCop ist ein Tool zur statischen Analyse von .Net Assemblys). Die Regel „Disposable fields should be disposed“ (CA2213) besagt, dass eine Klasse welche Member enthält, die IDisposable implementieren, selber IDisposable implementieren soll. In der eigenen Dispose-Methode, werden dann die Dispose-Methoden der Members aufgerufen. Genauso relevant wie Member freizugeben, ist es auch lokale Variablen wieder freizugeben. Für FxCop 1.36 bietet Microsoft keine entsprechende Regel.

In der Version 10 von FxCop gibt es die Regel „Dispose objects before losing scope“ (CA2000). Diese Regel verursacht einen FxCop Fehler, sollte an einem Objekt nicht die Dispose-Methode aufgerufen werden, bevor alle Referenzen auf dem Objekt „out of scope“ sind. Zusätzlich sollte der Dispose-Aufruf in einem Finally-Block erfolgen, damit Exceptions nicht eine Finalisierung/Freigabe des Objektes unterbinden. Die Anzahl an Fehlermeldungen, welche keine sind, ist bei dieser Regel relativ hoch, zum Beispiel werden Create-Methoden als Fehler erkannt:

public static DisposableChild RuleNine()
{    
      var child = new DisposableChild();    
      if (child.Init()) { return child; }    
      Debug.Fail("unable to create object");    
      return null;
}

Die von mir für FxCop 1.36 entworfene Regel arbeitet auf wie folgt:

1\. Alle lokalen Variablen überprüfen, ob sie oder eine Basis-Klasse IDisposable implementiert, wobei Compiler-Generierte lokale Variablen ignoriert werden.

2\. Alle Aufrufe von Dispose-Methoden suchen und zu lokalen Variablen zuordnen.

3\. Alle Return-Anweisungen überprüfen, ob sie lokale Variablen, welche IDisposable implementieren zurück liefern.

4\. Ausgabe eines FxCop-Fehlers, von lokalen Variablen, die nicht mittels Return-Anweisung zurückgegeben werden und bei denen auch kein Aufruf der Dispose-Methode erfolgte.

Den kompletten Quelltext inkl. der benötigte Xml-Datei  (kompiliert als eingebettete Ressource) befindet sich auf pastebin.com:

[FxCop Rule: Dispose local variables](http://pastebin.com/QMpZ3pt9)

Viel Erfolg mit der Fehlersuche!
