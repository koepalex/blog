---
title: "GetHashCode dein Freund und Sorgenkind"
date: "2013-10-20"
categories: 
  - "dotnet"
tags: 
  - "dotnet"
  - "gethashcode"
  - "hash"
  - "performance"
---

Eine wichtige Methode beim Arbeiten im .NET Umfeld ist GetHashCode. Sie gibt einen 32 Bit Integer zurück der das Objekt identifizieren soll. Der Hash-Code beschreibt also die Identität des Objektes (im Gegensatz zur Speicherreferenz auf das Objekt).

Daraus leitet sich die Frage ab _Wann zwei Objekte die selbe Identität besitzen?_ Im Falle einer Object-Relational-Mapper Klasse beispielsweise, wenn die Instanzen der O/R-Mapper Klasse auf ein und die selbe Zeile(n) der selben Tabelle(n) der selben Datenbank(en) verweisen. Klassischer weise durch eine Kette von Primary-Keys. Der Hash-Code ist auch ein Sorgenkind eines Entwicklers da schon hier die Probleme beginnen. Eine einfach Implementierungen könnte sein: 
```csharp
return (tableKey.GetHashCode()+ key1.GetHashCode() + keyN.GetHashCode());
```
Diese ist jedoch Falsch denn die Addition von unterschiedlichen Summanden kann die gleichen Summe ergeben. Eine verbesserte Implementierung ist: `return string.Concat(tableKey, key1, keyN).GetHashCode();` Jedoch beinhaltet auch diese Implementierung einen unscheinbaren Fehler.

- key1: "ab" keyN: "c" ergibt _"abc"_.GetHashCode()
- key1: "a" keyN "bc" ergibt auch _"abc"_.GetHashCode()

Eine weiter verbesserte Implementierung ist: 
```csharp
return string.Format("{0} : {1} : {2}", tableKey, key1, keyN) .GetHashCode();
```
Um jedoch einen noch besseren Hash-Code zu berechnen müsste man die Hash-Code der einzelnen Felder generieren und diese mittels einer Hash-Funktion verarbeiten. Beispiele für Hash-Funktionen sind:

1. [Jenkins](http://en.wikipedia.org/wiki/Jenkins_hash_function)
2. [Murmur](http://en.wikipedia.org/wiki/Murmurhash)
3. [CityHash](http://en.wikipedia.org/wiki/CityHash)
4. [Goulburn](http://www.webcitation.org/query?url=http://www.geocities.com/drone115b/Goulburn06.pdf&date=2009-10-25+21:06:51)

Empfehlenswert ist diese letzte Variante nur für Programme die mit großen Datenmengen arbeiten müssen oder mit besonders langen Laufzeiten haben (z.B. Server-Anwendungen).

* * *

Die Symbole _tableKey_, _key1_ und _keyN_ im oberen Beispiel sollten am besten als **readonly** gekennzeichnet werden und zwar aufgrund der [Microsoft GetHashCode Guidelines](http://msdn.microsoft.com/en-us/library/system.object.gethashcode.aspx)

> **"the integer returned by GetHashCode should never change"**

Ein veränderlicher Hash-Code ist unter anderem Problematisch wenn:

1. Ein Objekt in einer Datenstruktur (wie Hashset) enthalten ist.
2. Das Objekt so modifiziert wird, dass sich sein Hash-Code verändert.
    - Nun ist es eher vom Zufall abhängig ob sich das Objekt noch im richtigen [**bucket**](http://msdn.microsoft.com/en-us/library/ms379571.aspx) der Datenstruktur befindet. Wenn nicht ist das Objekt in der Datenstruktur verwaist.
    - Existieren andere Instanzen die auf die gleiche Identität zeigen sollten beinhaltet man die Software ab diesem Zeitpunkt einen schwer zu lokalisierenden Fehler.

In der Realität ist es nicht immer möglich den erzeugten Hash-Code stabil zu halten, meist weil man Ressourcen sparend programmieren muss (z.B. Objekte wieder verwenden da _teure_ Ressourcen benutzen). Deswegen hat sich der Microsoft Mitarbeiter _Eric Lippert_ solcher Probleme angenommen und in seinem Blog den sehr lesenswerten Eintrag [Guidelines and rules for GetHashCode](http://blogs.msdn.com/b/ericlippert/archive/2011/02/28/guidelines-and-rules-for-gethashcode.aspx) verfasst. Darin heißt es unter anderem:

> **"Rule: the integer returned by GetHashCode should never change while the object is contained in a data structure that depends on the hash code remaining stable"**

* * *

Eine andere Problematik ergibt sich aus der Tatsache, dass der Hash-Code _nur_ 32 Bit groß ist. Der rund 4,2 Milliarden Zahlen fassende Zahlenraum kann nur effektiv ausgenutzt werden, wenn die verwendete Hash-Funktion die Hash-Codes möglichst gleichmäßig über den Zahlenraum verteilt.

Selbst mit einer Optimalen Hash-Funktion gilt jedoch, dass je mehr Hash-Codes ermittelt werden desto höher ist die Wahrscheinlichkeit einer Hash-Kollision. Eine Kollision entsteht wenn zwei unterschiedliche Identitäten existieren, für welche derselbe Hash-Code generiert wird. _Eric Lippert_ hat ermittelt, dass bei nur 9300 generierten Hashes die Chance einer Kollision 1 % beträgt. Für 77000 Objekte beträgt die Chance bereits 50% [vgl. Socks, birthdays and hash collisions](http://blogs.msdn.com/b/ericlippert/archive/2010/03/22/socks-birthdays-and-hash-collisions.aspx). Daher ist es eine schlechte Idee, alle Objekte in einer Datenstruktur zu verwalten und somit den Hash-Code als globalen Identifizier zu verwenden.

* * *

Die dritte Problematik welche ich ansprechen möchte ist Performance. Falsch Implementierte oder schlecht ausgewählte Hash-Funktionen können die Laufzeit eines Programms deutlich verschlechtern. Die Performance-Verschlechterungen treten u.a. auf wenn eine Hash-Funktion:

- zu lange braucht um den Hash-Code zu berechnen (Kryptographische Hash-Funktionen eigenen sich i.d.R. nicht für GetHashCode)
- nicht den gesamten Zahlenraum nutzen, dann dauert das einfügen von Elementen in eine Datenstruktur sehr lange (häufige Kollisionen)

Der letzte Punkt ist eine bekannte Falle bei der Verwendung von _structs_. Bei einem Struct sollte die GetHashCode-Methode immer überschrieben werden, da die Standardimplementierung nur einen kleinen Teil des Zahlenraums ausnutzt siehe [C# FAQ: Why are hashtable lookups so slow with struct keys](http://tutorials.csharp-online.net/index.php?title=CSharp_FAQ:_Why_are_hashtable_lookups_so_slow_with_struct_keys).
