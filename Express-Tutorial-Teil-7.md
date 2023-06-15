## Templates mit Pug

Eine Vorlage ist eine Textdatei, die die Struktur oder das Layout einer Ausgabedatei definiert, wobei Platzhalter verwendet werden, um darzustellen, wo Daten eingefügt werden, wenn die Vorlage gerendert wird (in Express werden Vorlagen als Ansichten bezeichnet).

Express-Vorlagenauswahl
Express kann mit vielen verschiedenen Vorlagenrendering-Engines verwendet werden. In diesem Tutorial verwenden wir Pug (früher bekannt als Jade) für unsere Vorlagen. Dies ist die beliebteste Vorlagensprache von Node und beschreibt sich selbst als eine "saubere, whitespace-sensible Syntax zum Schreiben von HTML, stark beeinflusst von Haml".

Vorlagenkonfiguration
Die LocalLibrary wurde konfiguriert, um Pug zu verwenden, als wir die Skeleton-Website erstellten. Sie sollten das Pug-Modul als Abhängigkeit in der package.json-Datei der Website sehen und die folgenden Konfigurationseinstellungen in der app.js-Datei. Die Einstellungen zeigen uns, dass wir pug als View-Engine verwenden und dass Express nach Vorlagen im /views-Unterverzeichnis suchen soll.

```javascript
// View engine setup
app.set("views", path.join(__dirname, "views"));
app.set("view engine", "pug");
```

Wenn Sie in das Verzeichnis "views" schauen, sehen Sie die .pug-Dateien für die Standardansichten des Projekts. Dazu gehören die Ansicht für die Startseite (index.pug) und die Basistemplate (layout.pug), die wir durch unseren eigenen Inhalt ersetzen müssen.

```bash
/express-locallibrary-tutorial  //the project root
  /views
    error.pug
    index.pug
    layout.pug
```

### Vorlagensyntax
Die unten stehende Beispieldatei für die Vorlage zeigt viele der nützlichsten Funktionen von Pug.

Das erste, was auffällt, ist, dass die Datei die Struktur einer typischen HTML-Datei abbildet, wobei das erste Wort in (fast) jeder Zeile ein HTML-Element ist und Einrückungen verwendet werden, um verschachtelte Elemente zu kennzeichnen. Also zum Beispiel befindet sich das body-Element in einem html-Element, und Absatz-Elemente (p) befinden sich innerhalb des body-Elements usw. Nicht verschachtelte Elemente (z.B. einzelne Absätze) befinden sich auf separaten Zeilen.

```pug
doctype html
html(lang="en")
  head
    title= title
    script(type='text/javascript').
  body
    h1= title

    p This is a line with #[em some emphasis] and #[strong strong text] markup.
    p This line has un-escaped data: !{'<em> is emphasized</em>'} and escaped data: #{'<em> is not emphasized</em>'}.
      | This line follows on.
    p= 'Evaluated and <em>escaped expression</em>:' + title

    <!-- You can add HTML comments directly -->
    // You can add single line JavaScript comments and they are generated to HTML comments
    //- Introducing a single line JavaScript comment with "//-" ensures the comment isn't rendered to HTML

    p A line with a link
      a(href='/catalog/authors') Some link text
      |  and some extra text.

    #container.col
     

 if title
        p A variable named "title" exists.
      else
        p A variable named "title" does not exist.
      p.
        Pug is a terse and simple template language with a
        strong focus on performance and powerful features.

    h2 Generate a list

    ul
      each val in [1, 2, 3, 4, 5]
        li= val
```

Elementattribute werden in Klammern nach ihrem zugehörigen Element definiert. Innerhalb der Klammern werden die Attribute in Komma- oder Leerzeichen-getrennten Listen der Paare von Attributnamen und Attributwerten definiert, zum Beispiel:

```pug
script(type='text/javascript'), link(rel='stylesheet', href='/stylesheets/style.css')
meta(name='viewport' content='width=device-width initial-scale=1')
```

Die Werte aller Attribute werden escapet (z.B. werden Zeichen wie ">" in ihre HTML-Code-Äquivalente wie "&gt;" umgewandelt), um JavaScript-Injection oder Cross-Site-Scripting-Angriffe zu verhindern.

Wenn ein Tag durch ein Gleichheitszeichen gefolgt wird, wird der folgende Text als JavaScript-Ausdruck behandelt. So wird zum Beispiel in der ersten Zeile unten der Inhalt des h1-Tags die Variable title sein (entweder in der Datei definiert oder in die Vorlage von Express eingegeben). In der zweiten Zeile ist der Absatzinhalt ein Textstring, der mit der title-Variable verkettet ist. In beiden Fällen ist das Standardverhalten, die Zeile zu escapen.

```pug
h1= title
p= 'Evaluated and <em>escaped expression</em>:' + title
```

Wenn es kein Gleichheitszeichen nach dem Tag gibt, wird der Inhalt als Klartext behandelt. Innerhalb des Klartextes können Sie escapte und unescapte Daten mit der Syntax #{} und !{} einfügen, wie unten gezeigt. Sie können auch rohes HTML innerhalb des Klartextes hinzufügen.

```pug
p This is a line with #[em some emphasis] and #[strong strong text] markup.
p This line has an un-escaped string: !{'<em> is emphasized</em>'}, an escaped string: #{'<em> is not emphasized</em>'}, and escaped variables: #{title}.
```

Hinweis: Sie werden fast immer Daten von Benutzern escapen wollen (über die Syntax #{}). Daten, denen vertraut werden kann (z.B. generierte Zählungen von Datensätzen usw.), können ohne das Escapen der Werte angezeigt werden.

Sie können das Pipe-Zeichen ('|') am Anfang einer Zeile verwenden, um "Klartext" anzuzeigen. Zum Beispiel wird der zusätzliche Text unten auf derselben Zeile wie der vorherige Anker angezeigt, wird aber nicht verlinkt.

```pug
a(href='http://someurl/') Link text
| Plain text
```

Pug ermöglicht Ihnen die Durchführung von bedingten Operationen mit if, else, else if und unless - zum Beispiel:

```pug
if title
  p A variable named "title" exists
else
  p A variable named "title" does not exist
```

Sie können auch Loop-/Iterationsoperationen mit der Syntax each-in oder while durchführen. Im unten stehenden Codefragment haben

 wir durch ein Array iteriert, um eine Liste von Variablen anzuzeigen (beachten Sie die Verwendung von 'li=' zur Bewertung von "val" als Variable unten. Der Wert, über den Sie iterieren, kann auch als Variable in die Vorlage eingegeben werden!

```pug
ul
  each val in [1, 2, 3, 4, 5]
    li= val
```

Die Syntax unterstützt auch Kommentare (die nach Wunsch im Ausdruck gerendert werden können oder nicht), Mixins zur Erstellung wiederverwendbarer Codeblöcke, Case-Anweisungen und viele andere Funktionen. Für detailliertere Informationen siehe Die Pug-Docs.

## Template für die Local_Library

Dieses Webseiten Template wird eine Seitenleiste mit Links zu den Seiten haben, die wir hoffen, im Laufe der Tutorial-Artikel zu erstellen (z.B. zum Anzeigen und Erstellen von Büchern, Genres, Autoren, etc.) und einen Haupt-Inhaltsbereich, den wir in jeder unserer einzelnen Seiten überschreiben werden.

Öffnen Sie **/views/layout.pug** und ersetzen Sie den Inhalt durch den untenstehenden Code.

```pug
doctype html
html(lang='de')
  head
    title= title
    meta(charset='utf-8')
    meta(name='viewport', content='width=device-width, initial-scale=1')
    link(rel="stylesheet", href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/css/bootstrap.min.css", integrity="sha384-xOolHFLEh07PJGoPkLv1IbcEPTNtaed2xpHsD9ESMhqIYd0nLMwNLD69Npy4HI+N", crossorigin="anonymous")
    script(src="https://code.jquery.com/jquery-3.5.1.slim.min.js", integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj", crossorigin="anonymous")
    script(src="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/js/bootstrap.min.js", integrity="sha384-+sLIOodYLS7CIrQpBjl+C7nPvqq+FbNUBDunl/OZv93DB7Ln/533i8e/mZXLi/P+", crossorigin="anonymous")
    link(rel='stylesheet', href='/stylesheets/style.css')
  body
    div(class='container-fluid')
      div(class='row')
        div(class='col-sm-2')
          block sidebar
            ul(class='sidebar-nav')
              li
                a(href='/catalog') Startseite
              li
                a(href='/catalog/books') Alle Bücher
              li
                a(href='/catalog/authors') Alle Autoren
              li
                a(href='/catalog/genres') Alle Genres
              li
                a(href='/catalog/bookinstances') Alle Buchexemplare
              li
                hr
              li
                a(href='/catalog/author/create') Neuen Autor erstellen
              li
                a(href='/catalog/genre/create') Neues Genre erstellen
              li
                a(href='/catalog/book/create') Neues Buch erstellen
              li
                a(href='/catalog/bookinstance/create') Neues Buchexemplar erstellen (Kopie)

        div(class='col-sm-10')
          block content
```

Die Vorlage verwendet (und enthält) JavaScript und CSS von [Bootstrap](https://getbootstrap.com/) um das Layout und die Präsentation der HTML-Seite zu verbessern. Die Verwendung von Bootstrap oder einem anderen clientseitigen Web-Framework ist eine schnelle Möglichkeit, eine attraktive Seite zu erstellen, die gut auf verschiedene Browsergrößen skaliert. Außerdem ermöglicht es uns, uns um die Seitendarstellung zu kümmern, ohne in die Details gehen zu müssen - wir wollen uns hier auf den serverseitigen Code konzentrieren!

Die Basistemplate verweist auch auf eine lokale CSS-Datei (**style.css**), die ein wenig zusätzliche Formatierung bietet. Öffnen Sie **/public/stylesheets/style.css** und ersetzen Sie dessen Inhalt durch den folgenden CSS-Code:

```css
.sidebar-nav {
  margin-top: 20px;
  padding: 0;
  list-style: none;
}
```

Jetzt haben wir eine Basistemplate für das Erstellen von Seiten mit einer Seitenleiste. In den nächsten Abschnitten werden wir es verwenden, um die einzelnen Seiten zu definieren.