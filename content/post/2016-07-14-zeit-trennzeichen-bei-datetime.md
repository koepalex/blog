---
title: "Zeit-Trennzeichen bei DateTime"
date: "2016-07-14"
categories: 
  - "dotnet"
tags: 
  - "dotnet"
  - "datetime"
  - "formatstring"
---

Diese Woche hat es das .NET geschafft mich zu überraschen, ein Programm ist beim Kunden mit italienischen Windows immer wieder abgestürzt. Nach langem Suchen hat ein Kollege das Problem erkannt, was durch das folgende Beispiel veranschaulicht wird:

```csharp
var ci = new CultureInfo("it-IT");
var dateTime = DateTime.Now;
var str = dateTime.ToString("dd.mm.yyyy hh:mm:ss", ci);
Console.WriteLine(str);
```
Ausgabe (.NET 3.5): 14.07.2016 **20.45.30**  
Ausgabe (.NET 4.0): 14.07.2016 **20:45:30**  

In der Ausgabe erkennt man, dass abhängig von der .NET Version, das Zeit-Trennzeichen ändert. Aber warum wird das Trennzeichen überhaupt verändert, es wurde doch eine feste Format-Zeichenkette angegeben? Die [Dokumentation](https://msdn.microsoft.com/en-us/library/8kb3ddd4(v=vs.110).aspx#escape) beschreibt:  
The "d", "f", "F", "g", "h", "H", "K", "m", "M", "s", "t", "y", "z", **":"**, or "/" characters in a format string are interpreted as custom format specifiers rather than as literal characters. To prevent a character from being interpreted as a format specifier, you can precede it with a backslash (\\), which is the escape character.  
  
Es ist beschrieben, dass der Doppelpunkt "escaped" werden muss, jedoch muss man erstmal darauf kommen selbige zu lesen...

Richtig wäre also:  
```csharp
var str = dateTime.ToString(@"dd.mm.yyyy hh\:mm\:ss", ci);
```
