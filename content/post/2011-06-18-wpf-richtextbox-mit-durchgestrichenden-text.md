---
title: "WPF RichTextBox mit Durchgestrichenden Text"
date: "2011-06-18"
categories: 
  - "dotnet"
tags: 
  - "dotnet"
  - "c#"
  - "durchgestrichen"
  - "richtextbox"
  - "strikethrough"
  - "textdecorator"
  - "wpf"
---

Um bei einer WPF RichTextBox einen Text **Fett**, _Kursiv_ oder Unterstrichen darzustellen ist nicht viel notwenig. Eigentlich muss man im XAML nur einen (Toggle)Button definieren, der ein Kommando (EditingCommands) an die RichTextBox sendet:

```xml
<ToggleButton
  Command="EditingCommands.ToggleItalic"
  CommandTarget="{Binding ElementName=myRichTextBox}"
  Content="Italic"/>
<RichTextBox/>
```

Genauso einfach verhält es sich einen Text Hochgestellt oder Tiefgestellt darzustellen, mann muss nur zusätzlich noch die Schrift-Familie des Hinter der RichTextBox liegenden Dokumentes verändern:

```xml
<ToggleButton
  Command="EditingCommands.ToggleSubscript"
  CommandTarget="{Binding ElementName=myRichTextBox}"
  Content="Subscript"/>
<RichTextBox/>
  <!--on default fontfamily subscript and superscript dont work!!-->
  <FlowDocument FontFamily="Palatino Linotype" FontSize="14"/>
</RichTextBox>
```

Die Veränderungen an dem Text funktionieren in zwei Modi:

1. Ein Text wurde selektiert und der entsprechende Button geklickt, dies bewirkt das nur der selektierte Text beeinflusst wird.
2. Der Button wird geklickt ohne das Text selektiert ist, dies bewirkt das der danach neu geschriebene Text beeinflusst wird.

Einen Text Durchgestrichen (Strikethrough) anzuzeigen ist etwas Komplizierter, da die Klasse EditingCommands keine Eigenschaft ToggleStrikethrough besitzt. Im Internet finde man viele Webseiten die einem Zeigen wie man den Modi 1 Implmentiert:

```xml
<ToggleButton
  Click="myStrikethroughButtonClicked"
  Content="Strikethrough"/> 
```

```csharp
private void myStrikethroughButtonClicked(object sender, RoutedEventArgs e)
{
  if(myRichTextBox == null) return;
  if(!myRichTextBox.Selection.IsEmpty)
    HandleStrikethrouphSelection();
}
private void HandleStrikethrouphSelection()
{
  TextDecorationCollection decoratorCollection =
  myRichTextBox.Selection.GetPropertyValue(Inline.TextDecorationsProperty) as
  TextDecorationCollection;
  if(decoratorCollection == null) return;

  TextRange range = new TextRange(myRichTextBox.Selection.Start,
  myRichTextBox.Selection.End);

  if(!decoratorCollection.Equals(TextDecorations.Strikethrough))
    decoratorCollection = TextDecorations.Strikethrough;
  else
    decoratorCollection = new TextDecorationCollection();

  range.ApplyPropertyValue(Inline.TextDecorationsProperty, decoratorCollection);
}
```

Eine Lösung um den Modi 2 umzusetzen fand ich bei der Internet-Recherce nicht. Daher habe ich so lange Programmiert bis ich eine gefunden habe. Der Trick ist sich im Button-Click-Eventhandler ein Flag zu setzen (bzw. Rücksetzen). Im RichTextBox-TextChanged-Eventhandler wird dieses Flag gelesen um die TextDecorationCollection entweder neu zu erstellen oder auf TextDecorations.Strikethrough zusetzen.

```xml
<ToggleButton
  Click="myStrikethroughButtonClicked"
  Content="Strikethrough"/>
<RichTextBox TextChanged="myRichTextBoxTextChanged">
  <!-- on default fontfamily subscript and superscript dont work!!-->
  <FlowDocument FontFamily="Palatino Linotype" FontSize="14"/>
</RichTextBox>
```

```csharp
[Flags]
private enum RtfFlags
{
  NONE=0,
  STRIKETHROUGH=1,
  COLORING=2,
}
private RtfFlags m_ActiveFlags = RtfFlags.NONE;
private void UpdateActiveFlags(RtfFlags expected)
{
  if((m_ActiveFlags & expected) == expected)
    m_ActiveFlags &= ~expected; //unset flag
  else
    m_ActiveFlags |= expected; //set flag
}

private bool IsRtfFlagActive(RtfFlags expected)
{
  return (m_ActiveFlags & expected) == expected;
}

private void ApplyPropertyValueToText(int begin, int end, 
  DependencyProperty involvedProperty, object value)

{
  if(begin < 0 || end < 0)
    throw new ArgumentException("negative text position isn't supported");
  if(involvedProperty == null)
    throw new ArgumentNullException("involvedProperty");
  if(value == null) throw new ArgumentNullException("value");

  var contentStart = myRichTextBox.Document.ContentStart;
  var beginPos = contentStart.GetPositionAtOffset(begin);
  var endPos = contentStart.GetPositionAtOffset(end);
  var text = new TextRange(beginPos, endPos);
  text.ApplyPropertyValue(involvedProperty, value);
}
private void HandleStrikethroughGeneral(int begin, int end)
{
  TextDecorationCollection decoratorCollection = myRichTextBox.Selection.GetPropertyValue
    (Inline.TextDecorationsProperty) as TextDecorationCollection;
  if(decoratorCollection == null) return;

  decoratorCollection = IsRtfFlagActive(RtfFlags.STRIKETHROUGH)
   ? TextDecorations.Strikethrough
   : new TextDecorationCollection();

  ApplyPropertyValueToText(begin, end, Inline.TextDecorationsProperty,
    decoratorCollection);
}
private void myStrikethroughButtonClicked(object sender, RoutedEventArgs e)
{
  if(myRichTextBox == null) return;

  if(!myRichTextBox.Selection.IsEmpty)
    HandleStrikethrouphSelection();
  else
    UpdateActiveFlags(RtfFlags.STRIKETHROUGH);

  myRichTextBox.Focus();
}

private void myRichTextBoxTextChanged(object sender, TextChangedEventArgs e)
{
  foreach(var change in e.Changes)
  {
    if(change.RemovedLength == 0)//only work whether something added
      HandleStrikethroughGeneral(change.Offset,
        change.Offset + change.AddedLength);
  }
}
```
