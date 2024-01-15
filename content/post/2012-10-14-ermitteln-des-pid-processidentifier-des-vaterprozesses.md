---
title: "Ermitteln des PID (Processidentifier) des Vaterprozesses"
date: "2012-10-14"
categories: 
  - "c-net"
tags: 
  - "ntdll"
  - "parent"
  - "performancecounter"
  - "pid"
  - "wmi"
---

In manchen Fällen ist es notwendig heraus zu finden, welches der Vaterprozess eines Prozesses ist. Dafür gibt es im Allgemeinen drei verschiedene Lösungen im Windows .NET Umfeld.

Die häufigste Lösung ist die Verwendung von **Performancecountern**.

> var process = FindProcess();  
> using (var pC = new PerformanceCounter(
> 
>     "Process",  
>     "Creating Process ID",  
>     string.Format("{0}#{1}", process.ProcessName, 1),    
>     process.MachineName))  
> {  
>        int pid = (int)pC.NextValue();  
>        Console.WriteLine("parent pid = {0}", pid);  
> }

Die Verwendung von Performancecountern im Allgemeinen kann zwei mögliche Nachteile haben:

1. Vor Windows Vista war es ausreichend, dass der Account des Benutzers lokale Administratorrechte besitzt, um Performancecounter zu nutzen. Seit Windows Vista, muss der Account zusätzlich Mitglied in der Gruppe "Performance Monitor Users" sein oder der Programmierer muss mittels UAC (User Account Control) zur Laufzeit die entsprechenden Rechte beantragen.
2. Performancecounter funktionieren im wesentlichen dadurch, dass ihre Werte über die Registry abgefragt werden können. Auch werden alle Performancecounter über die Registry bekannt gegeben, somit ist eine Nutzung aus unterschiedlichen Programmen möglich (vgl. [http://msdn.microsoft.com/en-us/library/windows/desktop/aa371643(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/windows/desktop/aa371643%28v=vs.85%29.aspx)). Die Registry kann in verschiedenen Fällen jedoch korrumpiert werden, dies führt zu einer "System.FormatException" beim anlegen eines Performancecounters.  Es ist natürlich ohne weiteres Möglich die Exception zu fangen, jedoch wird man den Wert des Performancecounters nicht bekommen. Auf der Microsoft Support Webseite findet sich eine Anleitung um die Registry manuell zu reparieren ([http://support.microsoft.com/kb/300956](http://support.microsoft.com/kb/300956)).

Eine zweite Lösung ist mittels **P/Invoke die Windows Systembibliothek "ntdll.dll"** anzusprechen. Über die Methode NtQueryInformationProcess kann man Zugriff auf ein PROCESS\_BASIC\_INFORMATION-Struktur erlangen. Das Element InheritedFromUniqueProcessId liefert anschließend die gewünschte PID. Eine Beispielimplementierung ist zu finden unter [http://www.pinvoke.net/default.aspx/ntdll.ntqueryinformationprocess](http://www.pinvoke.net/default.aspx/ntdll.ntqueryinformationprocess).

Von der Verwendung der Methode ist jedoch aus heutiger Sicht abzuraten, da Microsoft im entsprechenden Developer Network Artikel darauf hinweist, dass diese in Zukunft modifiziert oder nicht mehr verfügbar sein wird (vgl. [http://msdn.microsoft.com/en-us/library/windows/desktop/ms684280(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/windows/desktop/ms684280(v=vs.85).aspx)).

Die (von mir) empfohlene Variante ist die Verwendung der **Windows Management Instrumentation** (kurz WMI, vgl. [http://msdn.microsoft.com/en-us/library/aa384642(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/aa384642(v=vs.85).aspx)). Es handelt sich hierbei um eine Implementierung der Web Based Enterprise Management-Spezifikation. Die Spezifikation enthält eine Menge von Funktionen zum Administrieren und zur Fernwartung von Computern, wobei sie Hardware und Betriebssystem unabhängig ist. WMI lässt sich auch lokal auf dem Computer verwenden, das auslesen der PID des Vaterprozesses sieht wie folgt aus:

> var process = FindProcess();  
> var query = string.Format("SELECT ParentProcessId FROM Win32\_Process WHERE ProcessId = {0}", process.Id);  
> using (var mos = new ManagementObjectSearcher(query))  
> {  
>     var enumerator = mos.Get().GetEnumerator();  
>     if(enumerator.MoveNext())  
>     {  
>         int pid = (int)((uint)enumerator.Current    
>            \["ParentProcessId"\]);  
>         Console.WriteLine("parent pid = {0}", pid);  
>     }  
>     else  
>         throw new InvalidOperationException("can't read   
>            parentProcessId by WMI");  
> }
