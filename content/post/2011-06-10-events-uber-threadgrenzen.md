---
title: "Events über Threadgrenzen"
date: "2011-06-10"
categories: 
  - "c-net"
tags: 
  - "net"
  - "c"
  - "event"
  - "thread"
---

Die meisten Tools/Programme verrichten Ihre Arbeit in Threads um die Oberfläche (UI) nicht zu blockieren. Eine "Kommunikations-Möglichkeit" um die UI beim Eintreten von bestimmten Gegebenheiten zu benachrichtigen sind Events. Ein Event im .Net ruft seine Ziele (InvokationList) aus dem Thread auf, aus dem es "gefeuert" wird. UI Controls, können i.A. nur im Mainthread arbeiten. Einen Zugriff aus einen anderen Thread führt zu einer Exception. Über das Interface ISynchronizedInvoke (implementiert von allen .Net UI Controls):

public interface ISynchronizeInvoke
{
 bool InvokeRequired { get; }
 IAsyncResult BeginInvoke (Delegate method, object\[\] args);
 object EndInvoke(IAsyncResult result);
 object Invoke(Delegate method, object\[\] args);
}

ist es nicht nur möglich nachzufragen ob ein Threadwechsel benötigt wird, er kann damit sogar ausgeführt werden. Damit die Methode zum Senden von Events in anderen Threads wiederverwendbar ist, bietet sich eine statische Hilfemethode an die mit Generics arbeitet und das Senden eines Events mit "normaler" Signatur unterstützt:

/// <summary> /// Method sending event to right Thread /// </summary> /// <typeparam name="T">Event which should arise</typeparam> /// <typeparam name="K">event arguments</typeparam> /// <typeparam name="G"> /// object which is the sender of event /// </typeparam> /// <param name="eventToFire">instance of T</param> /// <param name="args">instance of K</param> /// <param name="instance">instance of G</param> /// <remarks> /// this will arise all events with the signature /// void eventname(object sender, K eventargs) /// </remarks>

private static void RiseEvent<T, K, G\>(T eventToFire, K args, G instance)
  where T : clas
 {
  if(eventToFire != null)
  {
    MethodInfo methodInfo \= typeof (T).GetMethod("GetInvocationList");
    if (methodInfo != null)
    {
      Delegate\[\] delegates \= methodInfo.Invoke(eventToFire, null) as Delegate\[\];
      if (delegates != null)
      {
        foreach (Delegate del in delegates)
        {
          ISynchronizeInvoke syncInvoke \= del.Target as ISynchronizeInvoke;
          if (syncInvoke != null)
 {
            if (syncInvoke.InvokeRequired)
            {
              syncInvoke.Invoke(del, new object\[\] { instance, args });
            } else //no invoke requried so "normal" call delegate
             del.DynamicInvoke(new object\[\] { instance, args });
         } else //target don't implement ISynchronizeInvoke > no thread switch
           del.DynamicInvoke(new object\[\] { instance, args });
        }//end foreach
      } else Debug.Fail("GetInvocationList don't return Delegate\[\]");
    } else Debug.Fail("generic T don't seems to be an event");
  } else Debug.Write("wan't to rise event where nobody is registered for");
 }

Verwendet kann die Methode wie folgt werden:

RiseEvent<DataChanged, DataChangedEventArgs, object>
  (this.DataChanged, new DataChangedEventArgs, this);
