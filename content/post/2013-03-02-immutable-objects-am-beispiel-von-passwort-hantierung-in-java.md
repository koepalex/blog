---
title: "Immutable Objects am Beispiel von Passwort Hantierung in Java"
date: "2013-03-02"
categories: 
  - "miscellaneous"
tags: 
  - "immutable"
  - "java"
  - "passwort"
---

In Java sind Strings „Immutable Objects“ (unveränderliche Objekte),  
dass bedeutet sie werden zur Laufzeit nicht mehr geändert.  Immutables Objects haben viele Vorteile:

- im Allgemeinen ist es einfach möglich zu parallelisieren
- Implementierung von Undo- und Redo-Funktionalitäten sind normalerweise einfach (z.B. mittels Memento-Pattern)
- James Gosling (einer der Erfinder von Java), gibt zusätzlich an, dass bei Immutable Objects es grundsätzlich möglich ist, Ergebnisse zu Cachen und das die Sicherheit wird erhöht. (vgl. [https://www.artima.com/intv/gosling313.html](https://www.artima.com/intv/gosling313.html) )

Sicherheit ist ein gutes Stichwort, denn Immutable Objects können auch Sicherheitsprobleme mit sich bringen. Schauen wir uns einmal Passwörter bei Java an. Diese sollten niemals in einem String gespeichert werden, da sie solange der String im Speicher existiert, im Klartext im Speicher stehen. Die allgemeine Empfehlung in verschiedenen Security Guides lautet daher Passwörter nur als Char-Array zu hantieren und sobald man das Passwort nicht mehr benötigt, den Inhalt dieses Array zu überschreiben. Überschreiben funktioniert mit Immutable Objects nicht, was ein Passwort in einem String zu einem Sicherheitsrisiko werden lässt.

Die Java-Entwickler haben dies natürlich erkannt und u.a. die Methode getText() des JPasswordFields (mit ihren zwei Überladungen) wurden vor geraumer Zeit auf "deprecated" (obsolet) gesetzt und sollten nicht mehr Verwendet werden. Als Ersatz wurde die Methode getPassword(), welche ein Char-Array zurückliefert, hinzugefügt.So weit, so gut, doch wenn man in die Verlegenheit kommt, in einem JPasswordField ein Passwort zu setzen findet man nur die von JTextComponent geerbte Methode setText(). Diese Methode bekommt als Übergabeparameter einen String, was auch verständlich ist, da JTextComponent die Vaterklasse für alle Text-Controls ist. Die Methode sollte somit nicht verwendet werden. Als Lösung kann man so tun, als ob der Benutzer etwas in das Tastenfeld eingibt und die Zeichen einzeln an das JPasswordField senden:

```java
char[] password = getPasswordInternal();

for (int i = 0; i< password.length; i++) { 
    passwordField.dispatchEvent(
      new KeyEvent(
        passwordField,
        KeyEvent._KEY_TYPED_,
        System._currentTimeMillis_(),
        0,
        KeyEvent._VK_UNDEFINED_,
        password[i]));
}  
Arrays._fill_(password, '0');
```

Ich habe öfter die Frage gehört, warum man ein Passwort automatisch in ein Passwort-Control setzen sollte, es dient ja dazu da um Passworteingaben, von einem Benutzer zu bekommen. So finde ich die Antwort, weil man einen Betriebssystemunabhängigen PasswordManager bauen will oder weil es einfach den Anforderungen entspricht, logisch.

PasswordManager bieten meist auch die Funktionalität, das Passwort in die Zwischenablage zu kopieren und diese automatisch nach ein paar Sekunden, wieder zu säubern. In Java funktioniert der Zugriff auf die Zwischenablage über die Klasse java.awt.Toolkit.

```java
Toolkit
  ._getDefaultToolkit_()
  .getSystemClipboard()
  .setContents(
    new StringSelection(
      new String(password)), null);
```

Doch diese Implementierung ist natürlich nicht akzeptabel, da wieder ein String verwendet wird. Jeder Versuch die StringSelection Klasse mit einer eigenen Transferable Implementierung (siehe [http://docs.oracle.com/javase/1.4.2/docs/api/java/awt/datatransfer/Transferable.html](http://docs.oracle.com/javase/1.4.2/docs/api/java/awt/datatransfer/Transferable.html) ), die auf Basis eines Char-Arrays arbeitet, zu ersetzen schlugen bei mir fehl. Als Erläuterung, durch die Vaterklasse Transferable erbt man u.a. die Methode getTransferDataFlavors, bei welcher man ein Array von Unterstützen DataFlavors zurückliefern muss. Jedes Mal, wenn ich einen DataFlavor der sich auf Text oder UnicodeText bezieht, zurück geliefert habe, wurde im Verlauf der setContents Methode, der Rückgabewert der Methode getTransferData auf String gecastet. Was bei einem Char-Array nicht funktioniert. Meine Lösung für dieses Problem ist aufwendig. Und zwar dass via Java Native Access (JNA), für jedes Betriebssystem (Windows, Linux, MacOS, Solaris ) auf die nativen Funktionen zugegriffen wird. Für Windows könnte eine Implementierung wie folgt aussehen:

```java
interface Kernel32Lib  extends  StdCallLibrary {

    Pointer GlobalLock(Pointer hResult);
    Pointer GlobalUnlock(Pointer hResult);
    Pointer GlobalAlloc(int uFlags, int dwBytes);
    Pointer GlobalFree(Pointer hResult);
}

interface User32Lib extends StdCallLibrary {
    boolean OpenClipboard(Pointer hInstance);
    boolean EmptyClipboard();
    Pointer SetClipboardData(int uFormat, Pointer hMem);
    boolean CloseClipboard();
}

class Kernel32Wrapper {
    private Kernel32Lib _Instance;
    private Pointer _Memory;
    private Pointer _LockedMemory;

    private final static int _GMEM_MOVEABLE_ = 0x2;
    private final static int _GMEM _ZEROINIT_ = 0x40;

    Kernel32Wrapper() {
      _Instance = (Kernel32Lib) Native._loadLibrary_("Kernel32", Kernel32Lib.class);
    }

    public Pointer allocate(int size) {
        _Memory = _Instance.GlobalAlloc(_GMEM_MOVEABLE_ | _GMEM_ZEROINIT_, size);

        if (_Memory == null) {
            int errno = Native._getLastError_();
            System._out_.println(errno);
        }
        return _Memory;
    }
      
    public boolean free() {
        boolean result = false;

        if (_Memory !=  null) {
            result =  _Instance.GlobalFree( _Memory) !=  null;
           _Memory = null ;
        }
        return  result;
    }
      
    public Pointer lock() {
        _LockedMemory =  _Instance.GlobalLock( _Memory);
        if (_LockedMemory == null) {
            int errno = Native._getLastError_();
            System._out_.println(errno);
        }
         return  _LockedMemory;
    }
      
    public void unlock() {
        if (_LockedMemory != null) {
            _Instance.GlobalUnlock( _LockedMemory);
            _LockedMemory =  null ;
        }  
    }
}
  
class Win32ClipboardService {
   private final static int _CF_UNICODETEXT_ = 13;
   private User32Lib _Instance;
   private Kernel32Wrapper _Kernel32;

    Win32ClipboardService () {
     _Instance = (User32Lib) Native._loadLibrary_("User32",  User32Lib.class );
     _Instance.OpenClipboard( null );
     _Kernel32 = new Kernel32Wrapper();
    }
  
   public void setClipboard(char [] password) {
        byte[] buff = _convertCharArrayToByteArray_(password);
        int size = buff.length + 2;
        Pointer hResult =  _Kernel32.allocate(size);
  
        if (hResult != null) {
            Pointer globalMem = _Kernel32.lock();
            if (globalMem != null) {
                ByteBuffer bBuff = globalMem.getByteBuffer(0, buff.length);
  
                bBuff.put(buff);
                _Kernel32.unlock();
  
                globalMem = null;
  
                _Instance.SetClipboardData(_CF _UNICODETEXT_, hResult);
            }
        }
      Arrays._fill_(buff, 0, buff.length, ( byte )0);
    }
  
    public void clearClipboard() {
        _Instance.EmptyClipboard();
      }

    public void close() {
     _Instance.CloseClipboard();
     _Kernel32.free();
    }
  
    private static byte [] convertCharArrayToByteArray(char [] password) {

      byte [] result = new byte[password.length * 2];
  
        for(int i = 0; i < password.length; i++) {
            result[i * 2] = (byte)password[i];
            result[i * 2 + 1]=(byte)(password[i] >> 8);
          }
  
        return  result;
  
    }
}
```