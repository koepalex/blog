---
title: "UnauthorizedAccessException beim Kopieren einer Datei"
date: "2011-11-07"
categories: 
  - "dotnet"
tags: 
  - "dotnet"
  - "datei"
  - "exception"
  - "kopieren"
  - "unauthorizedaccess"
---

Kopieren, verschieben, umbenennen und löschen von Dateien gehören zu häufigen Aufgaben eines Softwareentwicklers. In diesem Beitrag sollen ein paar Dinge aufgezeigt werden die einem beim kopieren von Datein vielleicht nicht bewusst sind. Wie kopiert man jetzt am besten eine Datei? Eine Variante ist die statische Methode der Klasse File:

```csharp
File.Copy(“C:\\input\\test.bmp“, “C:\\output\\test.bmp“, true);
```

Damit man sich Überprüfungen wie ob die Datei bereits existiert (File.Exist()) sparen kann, wird die Überladung der Copy-Methode verwendet, welche bestehende Dateien überschreibt. Solange man die Software nur mit administrativen Rechten betreibt, hat man alles richtig gemacht. Muss die Software auch mit Benutzer Rechten arbeiten, kann man hier eine UnauthorizedAccessException bekommen. Dies würde zum Beispiel eintreten wenn eine schreibgeschützte Datei, bereits im Zielverzeichnis existiert und erneut in dieses Verzeichnis kopiert werden soll. Die .Net Implementierung der Copy-Methode fordert Berechtigungen on demand an, das bedeutet erst, wenn besondere Zugriffsrechte benötigt werden, werden diese angefordert. Als Benutzer hat man aber nicht die benötigten Rechte um schreibgeschützte Dateien zu überschreiben. Um diesen speziellen Fall zu lösen, könnte man nach dem kopieren die Attribute der Datei verändern (dafür genügen die Benutzer Rechte):

```csharp
const string path = "C:\\output\\test.bmp";  
FileAttributes attr = File.GetAttributes(path);  
if ((attr & FileAttributes.ReadOnly) == FileAttributes.ReadOnly)  
{ 
    attr &= (~FileAttributes.ReadOnly); 
    File.SetAttributes(path, attr);
} 
 ```

Oder man schreibt die Datei nur, wenn sie nicht bereits in dem Zielverzeichnis existiert. Eine andere Möglichkeit ist es sowohl die Quelldatei als auch die Zieldatei als FileStream zu laden, und anschließend nur den Inhalt zu übernehmen. Dadurch fallen eventuelle zusätzliche Attribute (oder Alternative NTFS-Dateiströme [http://de.wikipedia.org/wiki/Alternativer\_Datenstrom](http://de.wikipedia.org/wiki/Alternativer_Datenstrom) ) weg. Damit solche Fehler nicht erst beim Einsatz der Software auftreten, empfiehlt es sich entsprechende Unit-Tests unter Benutzer-Rechten laufen zulassen. Als Tipp sei noch das Programm Process Monitor (ehemals FileMon & Regmon) genannt. Es zeichnet Zugriffe auf das Dateisystem, die Registrierung und mehr mit vielen zusätzlichen Informationen (wie Dateiattribute) auf. Weitere Informationen finden Sie auf der Seite: [http://technet.microsoft.com/en-us/sysinternals/bb896645](http://technet.microsoft.com/en-us/sysinternals/bb896645)
