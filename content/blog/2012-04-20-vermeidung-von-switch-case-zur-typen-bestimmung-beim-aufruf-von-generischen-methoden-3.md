---
title: "Vermeidung von switch-case zur Typen Bestimmung beim Aufruf von generischen Methoden"
date: "2012-04-20"
categories: 
  - "c-net"
tags: 
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

  1: public void Foo (Array array)

  2: {

  3: 	var type = array.GetType();

  4: 	switch (type.FullName)

  5: 	{

  6: 		case „System.Boolean\[\]“ :

  7: 			var result = Bar((bool\[\]) array);

  8:                            break;

  9: 		case „System.Int32\[\]“:

 10: 			var result = Bar((int\[\]) array);

 11:                            break;

 12: 		case „System.String\[\]“:

 13: 			var result = Bar((string\[\]) array);

 14:                            break;

 15: 		case „my.KnownType\[\]“:

 16: 			var result = Bar((KnownType\[\]) array);

 17:                            break;

 18: 		case „other.SecondKnownType\[\]“:

 19:  			var result = Bar((SecondKnownType\[\]) array);

 20:                            break;

 21: 		case „different.ThirdKnownType\[\]“:

 22: 			var result = Bar((ThirdKnownType\[\]) array);

 23:                            break;

 24:     		//...

 25: 	}

 26: 	//do something

 27: }

 28:

 29: public T Bar<T>(T\[\] elements)

 30: {

 31: 	//do something

 32: }

Außer, dass der Quelltext davon aufgebläht wird, funktioniert die Methode nur mit vorher bekannten Typen. Eine Alternative dazu ist die Benutzung von Reflection:

  1: public void Foo (Array array)

  2: {

  3: 	var type = array.GetType();

  4: 	var methodInfo = GetType().GetMethod(„Bar“);

  5: 	var genericMethodInfo = methodInfo.MakeGenericMethod

  6: 		          (type.GetElementType());

  7: 	var result = genericMethodInfo.Invoke(this,

  8: 			new object\[\] {array});

  9: }

Im Falle von ref, bzw. out Parametern, bei denen eine konkrete Instanz gebraucht wird, kann diese über Activator.CreateInstance(), bzw. Array.CreateInstance() erstellt werden.
