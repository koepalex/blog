---
title: "Inversion Of Control(-Container), Service Locator und Dependency Injection"
date: "2012-06-08"
categories: 
  - "dotnet"
tags: 
  - "dotnet"
  - "dependency-injection"
  - "di"
  - "inversion-of-control"
  - "ioc"
  - "service-locator"
---

IOC-Container und DI sind zurzeit in vielen Fachzeitschriften diskutiert. Hier der Versuch einer kompakte Übersicht, die IOC, DI, SL und IOC-Container im Zusammenspiel erklärt.

**_Inversion Of Control_**

Beschreibt ein Paradigma zum Entwerfen von Frameworks. Ein Unterschied zwischen einem Framework und einer Bibliothek besteht darin, dass in einem Framework Methoden vom Verwender des Framework aufgerufen werden. Dies verändert den Kontrollfluss, bei einer klassischen Bibliothek, ruft der Verwender eine Methode auf, deren Funktionalität wird abgearbeitet und die Methode kehrt zum Aufrufer zurück. Die Steuerung des Kontrollflusses liegt also in der Hand des Verwenders der Bibliothek. Ein gutes Beispiel ist das .NET Framework, der Verwender kann Methoden über Events an Ereignisse binden, dass .NET Framework entscheidet, wann es die Funktionalität des Verwenders aufruft.

Konkretisiert wurde das Thema von Martin Fowler: [http://martinfowler.com/bliki/InversionOfControl.html](http://martinfowler.com/bliki/InversionOfControl.html)

**_Dependency Injection_**

Ist eine Anwendung von Inversion of Control, speziell im Bereich der Objekterzeugung. Objekte werden nicht mehr direkt erzeugt und initialisiert, dies geschieht von dem Verwender der Funktionalität. In vielen wird diese Aufgabe durch eine über Meta konfigurierbare Komponente erfüllt. Martin Fowler unterscheidet drei Typen von Dependency Injection:

- Constructor Injection – die zu verwendenden Objekte werden beim Initialisieren des Objektes über den Konstruktor mitgegeben.
- Setter Injection – die zu verwendenden Objekte können über Eigenschaften der Klasse gesetzt werden.
- Interface Injection – das Framework Objekt stellt ein Interface bereit, welches vom Verwender zu implementieren ist. Darüber werden Abhängigkeiten zur Laufzeit aufgelöst.

Die Typen basieren darauf, die injizierte Abhängigkeit als Teil des Objektes zu speichern. Das funktioniert bei kleinen Objekten mit geringen Lebenszeiten sehr gut.

Oft gibt es aber auch eine Art Method Dependency Injection, bei welcher einer Methode über ihre Parameter, die zu verwendenden Objekte mitgegeben werden, anstatt dass die Methode sie selbst erstellt.

Weitere Informationen finden Sie unter [http://www.martinfowler.com/articles/injection.html](http://www.martinfowler.com/articles/injection.html)

**_Service Locator_**

Die Idee ist es im Programm eine „Anlaufstelle“ für Services zu besitzen. Dort können verschiedene Services registriert und auch abgefragt werden. In den meisten Programmen genügt als ServiceLocator eine Singleton Klasse mit einer einfachen Schnittstelle wie:

public interface IServiceLocator { T Load<T> (string idOfService); }

Die Möglichkeiten der Realisierung des Service Locator sind vielfältig:

- Konfiguration zur Laufzeit, dabei werden IDs und konkrete Instanzen hinterlegt.
- In Meta-Dateien beschriebene IDs mit Angabe von Assembly und Klassennamen, zur Instanzierung mittels Reflection.
- Über Reflektion alle Typen suchen, welche ein entsprechendes Custom-Attribut besitzen.
- usw.

Weite Informationen finden Sie unter [http://www.martinfowler.com/articles/injection.html#UsingAServiceLocator](http://www.martinfowler.com/articles/injection.html#UsingAServiceLocator)

**_IOC-Container_**

IOC-Container werden auch als Dependency Injection Container bezeichnet. Ihr Aufgabe besteht darin, eine Klasse von der Objekterzeugung zu entkoppeln. Ein Beispiel:

Stellen Sie sich einen AVL-Suchbaum vor, dieser arbeitet mit AVL-Knoten, die ein Interface erfüllen wie:

```csharp
public interface IAvlNode<T, K> 
{ 
  T Key{get; set;} 
  K Value {get; set;} 
  IAvlNode<T, K> LeftChild {get; set;} 
  IAvlNode<T, K> RightChild {get; set;}
}
```

Nun sollen Elemente in den AVL-Suchbaum eingefügt werden, dass bedeutet der Baum muss, einen neuen Knoten erstellen und ihn in den Baum einfügen und ggf. sich neu gewichten. Hier liegt das Problem, der Baum erstellt den Knoten. Das sorgt für eine hohe Kopplung zwischen Baum und Knoten. Wenn die Kopplung nicht existieren würde, könnte im Baum Knoten geben, welche Ihre Daten sofort auf Festplatte serialisieren, oder Knoten die Ihre Daten remote auf Servern hinterlegen oder anderes erledigen. In einen fragwürdigen Design würde man der Baum die Logik enthalten, wann welche Art von Knoten zu erstellen ist. Den entsprechenden Knoten über Dependency Injektion in die Baum-Klasse zuübergeben, ist auch nicht praktikabel z.B. wenn der Baum, dass Standard interface IDictionary<T,K> erfüllen soll. Für dieses Problem stellt der IOC-Container eine Lösung da, er wird mit den Abhängigkeiten befüllt. Und der Baum kann an dem Container mittels Resolve<IAvlNode>(); einen Knoten erstellen lassen und damit Arbeiten.

Es gibt eine lange Liste an bekannten IOC-Containern (siehe [http://www.hanselman.com/blog/ListOfNETDependencyInjectionContainersIOC.aspx](http://www.hanselman.com/blog/ListOfNETDependencyInjectionContainersIOC.aspx)) und Ihre Arbeitsweise ist sehr vielfältig. Es geht von im Meta-Dateien beschriebenen Abhängigkeiten, über Registrierung zur Laufzeit bis hin zu Callback basierten Systemen.

Besonders möchte ich auf dieses Video, in welchem ein einfacher IOC-Container in 15min entwickelt wird, hinweisen: [https://www.youtube.com/watch?v=cfvTfU5RK8Q](https://www.youtube.com/watch?v=cfvTfU5RK8Q)

_**Warum das Ganze?**_

- lose Kopplung (kein direktes Anlegen von Objekten)
- erhöhte Testbarkeit (es werden richtige Unit-Tests möglich)
- Erhöhung der Wiederverwendbarkeit (siehe Baum Beispiel)

**_Wie kommt meine Klasse zu dem IOC-Container?_**

Die Antwort ist ganz klar Dependency Injection oder Service Locator.

**_Welche Nachteile ergeben sich durch Verwendung von IOC(-Container), DI und SL?_**

Je nach verwendeter Technology kann es zu schwer zu verstehenden Config-Dateien kommen. Der Quelltext ist anfänglich auch schwerer zu lesen (wobei einige Befürworter natürlich sagen, der Quelltext ist leichter zu lesen). Im Allgemeinen überwiegen die Vorteile einfach.
