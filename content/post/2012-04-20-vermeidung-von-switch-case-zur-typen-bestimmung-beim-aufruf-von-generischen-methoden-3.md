---
title: "Vermeidung von switch-case zur Typen Bestimmung beim Aufruf von generischen Methoden"
date: "2012-04-20"
categories: 
  - "dotnet"
tags: 
  - "dotnet"
  - "case"
  - "cast"
  - "generic"
  - "reflection"
  - "switch"
  - "type"
---

In der Entwicklung mit .NET hat man öfter folgendes Problem:

Methoden sind generisch implementiert und zu verwendende Objekte sind nur in Form von Referenzen auf Basistypen vorhanden (z.B. System.Array, System.Object). Diese Referenzen können nicht direkt an generische Methode übergeben werden (ergibt einen Compile Fehler).

Häufig liegen daher Typinformationen als String vor (über FullName-Property, aus Konfigurationsdatei, …). Diese Informationen werden anschließend verwendet, um den Typ der Referenz über switch-case aufzulösen.

Zum Beispiel:
```csharp
public void Foo (Array array)
{
    var type = array.GetType();
    switch (type.FullName)
    {
        case "System.Boolean[]" :
            var result = Bar((bool[]) array);
            break;
        case "System.Int32[]":
            var result = Bar((int[]) array);
            break;
        case "System.String[]":
            var result = Bar((string[]) array);
            break;
        case "my.KnownType[]":
            var result = Bar((KnownType[]) array);
            break;
        case "other.SecondKnownType[]":
            var result = Bar((SecondKnownType[]) array);
            break;
        case "different.ThirdKnownType[]":
            var result = Bar((ThirdKnownType[]) array);
            break;
        //...
    }
    //do something
 }

 public T Bar<T>(T[] elements)
 {
    //do something
 }
 ```

Außer, dass der Quelltext davon aufgebläht wird, funktioniert die Methode nur mit vorher bekannten Typen. Eine Alternative dazu ist die Benutzung von Reflection:

```csharp
public void Foo (Array array)
{
    var type = array.GetType();
    var methodInfo = GetType().GetMethod("Bar");
    var genericMethodInfo = methodInfo.MakeGenericMethod(type.GetElementType());
    var result = genericMethodInfo.Invoke(this, new object[] {array});
}
```

Im Falle von ref, bzw. out Parametern, bei denen eine konkrete Instanz gebraucht wird, kann diese über Activator.CreateInstance(), bzw. Array.CreateInstance() erstellt werden.
