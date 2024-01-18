---
title: "DSLs, Extension Methods und Fluent Interfaces"
date: "2012-07-28"
categories: 
  - "dotnet"
tags: 
  - "dotnet"
  - "dsl"
  - "extension-methods"
  - "fluent-interfaces"
---

In den letzten Jahren wurde in der Softwareentwicklung viel über Domänenspezifische Sprachen (domain specific language, kurz: DSLs) geschrieben. DSLs sollen von einem Domänenexperten gelesen werden können, auch ohne das die Domänenexperten über Programmierkenntnisse verfügen. DSLs können auch Entwicklern helfen die Quelltexte leserlicher zu machen, sowie Fehler zu vermeiden. Die Quelltextzeile:

> if (!SessionEstablished)

Ist einfach verständlich (für diejenigen welche eine C ähnliche Syntax verstehen), als Alternative könnte folgende Quelltextzeile dienen:

> if (Not(SessionEstablished))

Die relevante Information, dass es darum geht, herauszufinden, ob eine Verbindung nicht aufgebaut ist, sticht deutlicher heraus. Zudem erleichtert die textuelle Beschreibung das Verständnis, man muss zum Verstehen nicht wissen das „!“ die Negation der folgenden Anweisung bedeutet.

Eine einfache Möglichkeit eine DSL mit C# .NET zu realisieren sind Erweiterungsmethoden (extension methods), als Beispiel diene eine Klasse, welche Relationen (Links) zwischen zwei Objekten (wie Datenbankentitäten) darstellt.

```csharp
public class Relation : IRelation  
{
    //...  
    public DomainObject Target {get; private set; }  
    public DomainObject Source {get; private set; }  
    public static IEnumerable<IRelation> GetAllRelations   (DomainObject source)
    {
      // ..
    }  
    //...  
}
```

Um zu wissen, ob ein DomainObject mit einem anderen verlinkt ist, ist die Implementierung einfach:  
if (Relation.GetAllRelations(obj).Any(rel => rel.Target == otherObject))

Für Entwickler, die keine LINQ-Anweisungen verstehen, ist der Quelltext jedoch nicht so einfach. Eine Extension Method, könnte in diesem Fall sehr zur Verständlichkeit beitragen.

```csharp
public static class ExtensionMethods  
{  
    public bool IsLinkedTo(this DomainObject, source, DomainObject target)  
    {  
        return Relation.GetAllRelations(source).Any( rel =>  rel.Target == target);  
    }  
}
```

Mit dieser Extension Methode sieht man das der Quelltext der Ursprünglichen vereinfacht wurde:

> if (obj.IsLinkedTo(otherObject))

Eine andere Möglichkeit um eine DSL zu realisieren sind Fluent Interfaces. Die Idee ist, dass der Aufruf einer Methode, ein Interface zurückliefert, an diesem können dann sofort weitere Methoden aufgerufen werden. Dadurch ergibt sich eine Art „Satz“ der schnell erfasst werden kann (und dies ohne if (foo != null) Kaskaden alias Weihnachtsbäume).

In dem wegweisenden Buch von Robert C Martin (Clean Code, Seite 235ff), wird sehr schön beschrieben, wie ein Commandline-Parser refaktorisiert wurde, damit dieser einfacher zu verstehen, zu warten, zu erweitern und zu testen ist. Das einzige was beim Verwender der Klasse Args von Robert C Martin stört, ist, dass man die Typen der Parameter nicht erkennen kann:  
Dazu erst einmal der originale Aufruf der Klasse:

> Args arg = new Args("l, p#, d\*", args);

Oder können sie erkennen, welchen Typen die Parameter "l", "p" und "d" besitzen? Nicht ohne zu wissen, was "\*" oder "#" in diesem Kontext bedeutet. Dazu muss der Leser der Methode, in die Dokumentation oder in den Quelltext schauen.

Eine Fluent Interface Variante könnte wie folgt aussehen:

```csharp
Args arg = Args.GetInstance()  
  .Argument<bool>("l").SetDefault(false)  
  .Argument<int>("p").SetDefault(42)  
  .Argument<string>("d").SetDefault("foo")  
  .Parse(args);
```

Diese Variante ist nicht so kompakt wie die ursprüngliche, dafür enthält sie mehr Informationen (über Typen und Defaultwerte). Die Mächtigkeit zeigen Fluent Interfaces, wenn es darum geht, bestehenden Implementierungen zu erweitern. Egal ob man alle Parameter über ein anderes Zeichen beginnen lassen möchte "-", "--", "/", ob man zusätzliche Hilfe Texte zu den Parametern hinterlegen, ob man eine Validierung (über Minimal- und Maximalwerte) hinzufügen oder eine Aktion auslösen möchte.

```csharp
Args arg = Args.GetInstance()  
  .SetParameterSymbol("—")  
    .Argument<bool>("l")  
      .SetDefault(false)  
      .AddDescription()  
        .Header("l means Log")  
        .Text("starting the programm with a verbose logging")  
      .SetAction()  
        .WhenValueEqualsTo(true)  
          .Do (() => {m_Logger = new VerboseLogger();})  
      .Argument<int>("p")  
        .SetDefault(23)  
        .AddValidation()  
          .Minimum(7)  
          .Maximum(42)  
        .SetAction()  
          .WhenValueOutsideRange()  
            .Do((val) =>  {  
              Console.WriteLine(Ressource.ErrorCountProcesses);  
              m_NumberOfProcesses = 23;  
            })  
            .OtherwiseDo((val) =>  {  
              m_NumberOfProcesses = val;  
            })  
        .Argument<string>("d")  
          .SetDefault("foo")  
    .Parse(args);
```

Das letzte Beispiel enthält sehr viel Logik, entscheiden sie selbst ob sie es nicht vielleicht trotzdem als einfach verständlich betrachten.
