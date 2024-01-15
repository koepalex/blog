---
title: "Tracing/Logging und die Config-Datei"
date: "2011-06-08"
categories: 
  - "c-net"
tags: 
  - "net"
  - "c"
  - "config"
  - "trace"
---

Logging von Informationen ist so gut wie in jedem Programm notwendig. Nicht alle Entwickler können oder wollen ein professionelles Log-Tool wie z.B. [log4net](http://logging.apache.org/log4net/index.html) oder [nlog](http://nlog-project.org/) verwenden. In vielen Fällen reichen auch Debug.WriteLine oder Trace.WriteLine (letztes schreibt auch wenn die Software im Modus „Release“ kompiliert wurde).

Über eine Config-Datei lassen sich u.a. verschiedene TraceListener auswählen:

1. ConsoleTraceListener
2. DefaultTraceListener
3. DelimitedListTraceListener
4. EventLogTraceListener
5. EventSchemaTraceListener
6. TextWriterTraceListener
7. XmlWriterTraceListener
8. eigene Implementierungen welche sich von TraceListener ableiten

Damit kann man ohne neu kompilieren des Assemblies einfach den Ausgabeort des Trace-Informationen verändern.

Eine Config-Datei ist eine XML-Datei mit dem Namen des Assemblies und der Extension .config z.B. MeinBeispielApp.exe.config für das Programm MeinBeispielApp.exe. Visual Studio erkennt die Datei-Extension und bietet eine entsprechende Intellisense an. In einem Beispiel sollen die zwischen Ergebnisse einer Berechnung geloggt werden, der DefaultTraceListener schreibt diese Daten in den Debug-Stream des Betriebssystems.

Class Program
{
	static void Main(string\[\] args)
	{
		Stopwatch sw = new Stopwatch();
		sw.Start();
		Console.WriteLine("Result:{0}", Worker.Calculate(1, 100000));
		sw.Stop();
		Console.WriteLine("ElapsedTime:{0}", sw.ElapsedTicks);
		//output:
		//Result:21081692,7461519
		//ElapsedTime:82659575
	}
}
class Worker
{
	internal static double Calculate(int min, int max)
	{
		double result = 0.0;
		for(int i = min; i < max; i++)
		{
			result += Math.Sqrt((double)i);
			Trace.WriteLine(String.Format("Interimresult:\\t{0}",
                          result));
		}
		return	result;
	}
}

Wenn das Programm auf einem anderen Rechner ausgeführt wird (z.B. beim Kunden), währe ist es einfacher den Trace in einer Datei zu haben anstatt Remote zu Debuggen. Die Datei kann anschließend dem Entwickler zugesand werden kann. Um dies zu erreichen, dazu muss man in der Config-Datei nur etwas eintragen:

<?xmlversion="1.0"encoding="utf-8"?>
<configuration>
  <system.diagnostics>
    <trace>
      <listeners>
        <add name="otherListener" initializeData="D:\\tmp\\trace.txt"
             type="System.Diagnostics.TextWriterTraceListener" />
      </listeners>
    </trace>
  </system.diagnostics>
</configuration>

Die Config-Datei muss sich im selben Ordner wie das Assembly befinden. Tracing im kostet viel Zeit, die Methode Calculate benötigt ~ 80000000 Ticks zur Berechnung des Ergebnis mit aktiven Trace, aber nur ~215000 Ticks ohne. Daher sollte man darüber nachdenken ob man (wenn man die Software freigibt) nicht eine Config-Datei beilegt, welche alle TraceListener ausschaltet und diese nur in Problemfällen über die Config-Datei einschalten lässt. Eine Config-Datei welche alle Listener entfernt (auch welche Programmatisch z.B. via Trace.Listeners.Add(newXmlWriterTraceListener(@"C:\\log.xml")); hinzugefügt wurden) sieht z.B. so aus:

<?xmlversion="1.0"encoding="utf-8"?>
  <configuration>
    <system.diagnostics>
      <trace>
        <listeners>
          <clear/> <!-- Disable all TraceListener -->
        </listeners>
      </trace>
  </system.diagnostics>
</configuration>

Wer eine schöne Ausgabe sehen möchte, die Logs eventuell auf einen Webserver ablegen möchte oder verschiedene Log-Level (Info, Warning, Error) benutzen dem seien die oben erwähnten professionellen Log Tools empfohlen. Über die Config-Datei lassen sich noch viel mehr Einstellungen vornehmen, es lohnt sich damit zu Beschäftigen. Die erste Anlaufstelle für Informationen ist die [MSDN](http://msdn.microsoft.com/en-us/library/1fk1t1t0(v=VS.90).aspx).
