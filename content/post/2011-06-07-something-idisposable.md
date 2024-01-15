---
title: "Einiges über IDisposable"
date: "2011-06-07"
categories: 
  - "c-net"
tags: 
  - "net"
  - "bitmap"
  - "c"
  - "idisposable"
---

Das Standard Interface IDisposable welches  zur "Freigabe" von Ressourcen in .Net dient ist recht Einfach:

public interface IDisposable
{
	void Dispose();
}

Nur dieses Interface zu Implementieren reicht in vielen Fällen nicht aus. Es gibt z.B. einen FxCop Fehler [Implement IDisposable correctly](http://msdn.microsoft.com/en-us/library/ms244737(v=vs.80).aspx), dieser erscheint u.a. bei non-sealed Klassen welche keinen Medthode mit der Signatur protected virtual Dispose(bool) besitzen.  IDisposable zu Implementieren wird nötig wenn man:

1. unmanged (native) Ressourcen lädt um diese wieder freizugeben
2. managed Felder besitzt, welche wiederum IDisposable implementieren

Im folgenden Beispiel zeigt eine (nach FxCop komplette Implementierung) einer Basis-Klasse  und einer Kind-Klasse:

public class ManagedBasisClassI : IDisposable
{

	~ManagedBasisClassI()
	{
		Dispose(false);
	}

	public void Dispose()
	{
		Dispose(true);
	}

	protected virtual void Dispose(bool disposing)
	{
		if (disposing)
		{
			//clean managed ressources
			GC.SuppressFinalize(this);
		}
		//clean unmanaged ressources
	}
}

public class ManagedChildClass : ManagedBasisClassI
{
	protected override void Dispose(bool disposing)
	{
		try
		{
			if (disposing)
			{
				//clean managed ressources
			}
			//clean unmanaged ressources
		}
		finally
		{
			base.Dispose(disposing);
		}
	}
}

Man darf jetzt nicht auf die Idee kommen, dass IDisposable im .Net Framework dazu dient den Lebenszyklus von Objekten zu steuern. Objekte existieren mindestens so lange, wie eine Referenz auf sie besteht (Ausnahme: WeakReference). Bitte das "mindestens" beachten, denn wann ein Objekt aus dem Speicher verschwindet, auf das keine Referenz mehr existiert entscheidet nur der Garbage Collector. Zugriffe auf Objekte die noch existieren, aber bei denen Dispose bereits aufgerufen wurde, enden oft in Exceptions. Die Klasse Bitmap wirft in solchen Fällen eine  ArgumentException “Ungültiger Parameter“.  Eigene Klassen sollte man daher eine Weitere public Property erhalten, durch welche man Abfragen kann ob das Objekt bereits Disposed wurde oder gerade Disposed wird. In solchen Fällen, darf man auf das Objekt nicht mehr Zugreifen, sondern muss es Neuanlegen.

public class ManagedBasisClassII : IDisposable
{
	~ManagedBasisClassII()
	{
		Dispose(false);
	}

	public void Dispose()
	{
		Dispose(true);
	}

	protected virtual void Dispose(bool disposing)
	{
		**if (Disposing) throw new ObjectDisposedException(this.ToString());
		Disposing = true;**
		if (disposing)
		{
			//clean managed ressources
			GC.SuppressFinalize(this);
		}
		//clean unmanaged ressources
	}

	**public bool Disposing { get; private set; }**
}

Diese Lösung trägt für eigene Objekte sehr gut. Problematisch wird es bei Objekten von Klassen des .Net Framework selbst. Der nächste Quelltext stellt eine kleines Dummy Programm dar, indem 2 Clients Zugriff auf ein und das selbe [Bitmap](http://msdn.microsoft.com/en-us/library/system.drawing.image.aspx) Objekt möchten. Client1 ruft Dispose am Bitmap auf nach Verwendung (was auch sehr gut ist). Client2 bekommt beim Versuch das Bitmap aus einem Pool zu laden, die Referenz auf das alte Objekt, beim Zugriff auf eine der Methoden des Objektes kommt es zum Absturz des Programmes.

class Program
{
	static void Main(string\[\] args)
	{
		PicturePool pool = new PicturePool();
		ClientI client1 = new ClientI(pool);
		client1.Run();
		ClientII client2 = new ClientII(pool);
		client2.Run();
	}
}

public class ClientI
{
	private PicturePool m\_Pool;
	public ClientI(PicturePool pool)
	{
		if(pool == null) throw new ArgumentNullException("pool");
		m\_Pool = pool;
	}

	public void Run()
	{
		Bitmap pic = m\_Pool\[@"D:\\tmp\\bild.jpg"\];
		Console.WriteLine(String.Format("Height:{0}", pic.Height));
		Console.WriteLine(String.Format("Width:{0}", pic.Width));
		Console.WriteLine(String.Format("Pixelformat:{0}", pic.PixelFormat));
		pic.Dispose();
	}
}

public class ClientII
{
	private PicturePool m\_Pool;
	public ClientII(PicturePool pool)
	{
		if(pool == null) throw new ArgumentNullException("pool");
		m\_Pool = pool;
	}
	public void Run()
	{
		Bitmap pic = m\_Pool\[@"D:\\tmp\\bild.jpg"\];
		**_//ArgumentException"UngültigerParameter"_**
		Image thumbnail = pic.GetThumbnailImage(200, 200, null, IntPtr.Zero);
		thumbnail.Save(@"D:\\tmp\\bild\_thumbnail.jpg");                 pic.Dispose();
	}
}

public class PicturePool
{
	private Dictionary<string, WeakReference> m\_Pool;

	public PicturePool()
	{
		m\_Pool = new Dictionary<string, WeakReference>();
	}

	public Bitmap this\[string path\]
	{
		get
		{
			if(String.IsNullOrEmpty(path)) throw new ArgumentNullException("path");
			return GetPicture(path);
		}
	}

	private Bitmap GetPicture(string path)
	{
		if(!m\_Pool.ContainsKey(path) || !m\_Pool\[path\].IsAlive || m\_Pool\[path\].Target == null)
		{
			m\_Pool\[path\] = new WeakReference(new Bitmap(path));
		}
		**_//WeakReference.IsAlvie kann true sein aber das Target bereits Disposed
		//was kann man tun?_**
		Return m\_Pool\[path\].Target as Bitmap;
	}
}

Da es nicht Möglich ist zu ermitteln ob das Bitmap-Objekt bereits Disposed ist kommt es zu dem oberen Absturz. Eine einfache Lösung  ist ein Wrapper um das Bitmap Objekt (eine Ableitung kann es nicht sein, da die Klasse Bitmap Sealed ist). Der PicturePool gibt nur noch Referenzen auf einen BitmapWrapper herraus und fragt am BitmapWrapper ob er Disposed ist, wenn ja muss der Wrapper für dieses Bitmap neu geladen werden.

class Program
{
	static void Main(string\[\] args)
	{
		PicturePool2 pool = new PicturePool2();
		ClientIa client1 = new ClientIa(pool);
		client1.Run();
		ClientIIa client2 = new ClientIIa(pool);
		client2.Run();
	}
}

public class ClientIa
{
	private PicturePool2 m\_Pool;
	public ClientIa(PicturePool2 pool)
	{
		if(pool == null) throw new ArgumentNullException("pool");
		m\_Pool = pool;
	}

	public void Run()
	{
		**BitmapWrapper wrapper** = m\_Pool\[@"D:\\tmp\\bild.jpg"\];
		Console.WriteLine(String.Format("Height:{0}", **wrapper.Bitmap**.Height));
		Console.WriteLine(String.Format("Width:{0}", **wrapper.Bitmap**.Width));
		Console.WriteLine(String.Format("Pixelformat:{0}", **wrapper.Bitmap**.PixelFormat));
		**wrapper.Dispose();**
	}
}

public class ClientIIa
{
	private PicturePool2 m\_Pool;
	public ClientIIa(PicturePool2 pool)
	{
		if(pool == null) throw new ArgumentNullException("pool");
		m\_Pool = pool;
	}
	public void Run()
	{
		**BitmapWrapper wrapper** = m\_Pool\[@"D:\\tmp\\bild.jpg"\];
		Image thumbnail = **wrapper.Bitmap**.GetThumbnailImage(200, 200, null, IntPtr.Zero);
		thumbnail.Save(@"D:\\tmp\\bild\_thumbnail.jpg");
                **wrapper.Dispose();**
	}
}

public class PicturePool2
{
	private Dictionary<string, WeakReference> m\_Pool;

	public PicturePool2()
	{
		m\_Pool = new Dictionary<string, WeakReference>();
	}

	public **BitmapWrapper** this\[string path\]
	{
		get
		{
			if(String.IsNullOrEmpty(path)) throw new ArgumentNullException("path");
			return GetPicture(path);
		}
	}

	private **BitmapWrapper** GetPicture(string path)
	{
		if(!m\_Pool.ContainsKey(path) || !m\_Pool\[path\].IsAlive || m\_Pool\[path\].Target == null)
		{
			m\_Pool\[path\] = new WeakReference(new BitmapWrapper(path));
		}
		else
		{
			**BitmapWrapper pic = m\_Pool\[path\].Target as BitmapWrapper;
			if(pic.Disposed)
			{
				m\_Pool\[path\] = new WeakReference(new BitmapWrapper(path));
			}**
		}
		return m\_Pool\[path\].Target as **BitmapWrapper**;
	}
}

**public sealed class BitmapWrapper : IDisposable
{
	public BitmapWrapper(string path)
	{
		if(String.IsNullOrEmpty("path")) throw new ArgumentNullException("path");
		Bitmap = new Bitmap(path);
	}

	public bool Disposed { get; private set; }
	public Bitmap Bitmap { get; private set; }

	public void Dispose()
	{
		Disposed = true;
		Bitmap.Dispose();
	}
}**

Dies ist kein All-Heil-Mittel, sollte jemand wrapper.Bitmap.Dispose() aufrufen, kommt es zum alten Problem.  Doch für viele Szenarien, ist diese Variante absolut ausreichend. Die Königs-Lösung ist ein komplett Wrapper für das Bitmap dieser

- Leitet sich von System.Drawing.Image ab und ist sealed
- Beinhaltet einen privaten Member vom Typ System.Drawing.Bitmap
- Implementiert alle public Methoden von Bitmap und Image und leitet diese an den privaten Member weiter (notfalls mit new Überschreiben)
- Implementiert IDisposable und die public Property Disposed

Diesen Komplett Wrapper zu schreiben ist etwas aufwendig, lohnt sich irgendwann (kommt auf die Größe und die Laufzeit des Projektes an).

Ich hoffe das dieser Blog Eintrag etwas weiterhelfen konnte. :-D
