## Überblick

Im [letzten Tutorial-Artikel](/en-US/docs/Learn/Server-side/Express_Nodejs/mongoose) haben wir _Mongoose_-Modelle definiert, um mit der Datenbank zu interagieren und haben ein (eigenständiges) Skript verwendet, um einige erste Bibliotheksdatensätze zu erstellen. Jetzt können wir den Code schreiben, um diese Informationen den Benutzern zu präsentieren. Das Erste, was wir tun müssen, ist zu bestimmen, welche Informationen wir auf unseren Seiten anzeigen möchten, und dann geeignete URLs für die Rückgabe dieser Ressourcen zu definieren. Danach müssen wir die Routen (URL-Handler) und Ansichten (Templates) erstellen, um diese Seiten anzuzeigen.

Die untenstehende Grafik dient als Erinnerung an den Hauptablauf von Daten und den Dingen, die bei der Behandlung einer HTTP-Anforderung/Antwort implementiert werden müssen. Zusätzlich zu den Ansichten und Routen zeigt das Diagramm "Controller" - Funktionen, die den Code zur Routen der Anfragen von dem Code trennen, der tatsächlich die Anfragen verarbeitet.

Da wir die Modelle bereits erstellt haben, sind die Hauptpunkte, die wir erstellen müssen:

- "Routen", um die unterstützten Anfragen (und alle in Anforderungs-URLs codierten Informationen) an die entsprechenden Controller-Funktionen weiterzuleiten.
- Controller-Funktionen, um die angeforderten Daten von den Modellen zu erhalten, eine HTML-Seite erstellen, die die Daten anzeigt, und diese an den Benutzer zurückgeben, um sie im Browser anzusehen.
- Ansichten (Templates), die von den Controllern zur Darstellung der Daten verwendet werden.

![Hauptdatenflussdiagramm eines MVC Express-Servers: 'Routen' empfangen die HTTP-Anfragen, die an den Express-Server gesendet werden und leiten sie an die entsprechende 'Controller'-Funktion weiter. Der Controller liest und schreibt Daten von den Modellen. Modelle sind mit der Datenbank verbunden, um dem Server den Datenzugriff zu ermöglichen. Controller verwenden 'Ansichten', auch Templates genannt, um die Daten darzustellen. Der Controller sendet die HTML HTTP Antwort zurück an den Client als HTTP Antwort.](mvc_express.png)

Letztendlich könnten wir Seiten haben, die Listen und Detailinformationen für Bücher, Genres, Autoren und Buchinstanzen anzeigen, sowie Seiten zum Erstellen, Aktualisieren und Löschen von Datensätzen. Das ist viel, um es in einem Artikel zu dokumentieren. Daher wird der größte Teil dieses Artikels sich darauf konzentrieren, unsere Routen und Controller einzurichten, um "Dummy"-Inhalte zurückzugeben. Wir werden die Controller-Methoden in unseren nachfolgenden Artikeln erweitern, um mit den Modell-Daten zu arbeiten.

Der erste Abschnitt unten bietet eine kurze "Einführung" darüber, wie man die Express [Router](https://expressjs.com/en/4x/api.html#router) Middleware verwendet. Wir werden dieses Wissen in den folgenden Abschnitten verwenden, wenn wir die LocalLibrary-Routen einrichten.

## Einführung in Routen

Eine Route ist ein Abschnitt des Express-Codes, der ein HTTP-Verb (`GET`, `POST`, `PUT`, `DELETE`, usw.), einen

 URL-Pfad/Muster und eine Funktion, die aufgerufen wird, um die Anforderung zu bearbeiten, definiert. Im Code unten ist `app` eine Instanz von Express und wir binden die Route an sie.

```javascript
app.get('/', function (req, res) {
    res.send('Hello World!')
})
```

In diesem Fall ruft die Route eine anonyme Funktion auf (einen Callback), die zwei Objekte als Parameter hat: ein `Request`-Objekt (das die HTTP-Anforderung repräsentiert) und ein `Response`-Objekt (das verwendet wird, um die HTTP-Antwort an den Client zu erstellen und zu senden). In diesem Fall sendet der Callback einfach die Zeichenkette "Hello World!" an den Client.

Routes können benannt werden und optional mit einer oder mehreren Callback-Funktionen verknüpft werden. Die Callbacks sind wie Middleware, in dem Sinne, dass sie in der Reihenfolge ihrer Bindung aufgerufen werden, und jeder Callback hat die Möglichkeit, den Request-Verarbeitungs-Stack entlangzuschieben (zu "routen") zur nächsten Route oder Middleware.

Wenn eine Route nur einen Callback hat, kann sie direkt bereitgestellt werden, wie im obigen Beispiel. Wenn mehrere Callbacks bereitgestellt werden müssen, können sie in einem Array bereitgestellt werden, oder sie können direkt als Argumente zur Routenmethode bereitgestellt werden. Das untenstehende Beispiel zeigt eine Route mit drei Callback-Funktionen — die ersten beiden Callbacks rufen die `next()`-Middleware in der Kette auf und die letzte beendet den Request-Verarbeitungszyklus, indem sie die HTTP-Antwort sendet.

```javascript
app.get('/example/b', function (req, res, next) {
    console.log('the response will be sent by the next function ...')
    next()
}, function (req, res) {
    res.send('Hello from B!')
})
```

Ein häufiges Muster ist es, eine Serie von Middleware-Funktionen in einem Array zu definieren und sie dann an die Route zu binden. Das untenstehende Beispiel zeigt, wie man dies macht.

```javascript
var cb0 = function (req, res, next) {
    console.log('CB0')
    next()
}

var cb1 = function (req, res, next) {
    console.log('CB1')
    next()
}

var cb2 = function (req, res) {
    res.send('Hello from C!')
}

app.get('/example/c', [cb0, cb1, cb2])
```

## Express-Router

Die Express-Router-Klasse ist eine Erweiterung des Express-Servers: Sie ermöglicht es Ihnen, Routenverarbeitungs-Funktionen zu organisieren und sie an bestimmte URLs zu binden. Sie können beispielsweise einen Router erstellen und alle Anfragen für eine bestimmte URL (z.B. '/catalog') an den Router weiterleiten. Der Router könnte dann alle eingehenden Anfragen entsprechend dem relativen Pfad (der Teil der URL, der auf den Pfad folgt, an den der Router gebunden ist) an verschiedene Routen weiterleiten.

Die Express-Router-API ist sehr mächtig. Es ermöglicht es Ihnen, Routen zu definieren und sie an Controller zu binden, Parameter in den URLs zu spezifizieren und zu verwenden, die auf die Parameterwerte reagieren, und es ermöglicht Ihnen, spezielle Middleware an den Router zu binden, um Fehler zu behandeln.

Einige Beispiele für Routen, die Sie in einem Router erstellen können, finden Sie unten. Beachten Sie, dass diese alle Routen relativ zu dem Pfad sind, an den der Router gebunden ist (z.B. wenn der Router an '/catalog' gebunden ist, dann wird auf die Route für '/' als '/catalog/' zugegriffen, usw.).

```javascript
// GET request for the Book instance.
router.get('/book/:id', book_controller.book_detail);

// GET request for list of all Book items.
router.get('/books', book_controller.book_list);

// GET request for creating a Book. NOTE This must come before route for id (i.e. display book).
router.get('/book/create', book_controller.book_create_get);

// POST request for creating Book.
router.post('/book/create', book_controller.book_create_post);

// GET request to delete Book.
router.get('/book/:id/delete', book_controller.book_delete_get);

// POST request to delete Book.
router.post('/book/:id/delete', book_controller.book_delete_post);

// GET request to update Book.
router.get('/book/:id/update', book_controller.book_update_get);

// POST request to update Book.
router.post('/book/:id/update', book_controller.book_update_post);
```


## Benötigte Routen für die Lokale Bibliothek

Die URLs, die wir letztendlich für unsere Seiten benötigen, sind unten aufgelistet. Dabei wird _object_ durch den Namen jedes unserer Modelle (Buch, Buchinstanz, Genre, Autor) ersetzt, _objects_ ist der Plural von _object_, und _id_ ist das eindeutige Instanzfeld (`_id`), das jeder Mongoose-Modellinstanz standardmäßig zugewiesen wird.

- `catalog/` — Die Start-/Indexseite.
- `catalog/<objects>/` — Die Liste aller Bücher, Buchinstanzen, Genres oder Autoren (z.B. /`catalog/books/`, /`catalog/genres/`, etc.)
- `catalog/<object>/<id>` — Die Detailseite für ein bestimmtes Buch, eine Buchinstanz, ein Genre oder einen Autor mit dem gegebenen `_id` Feldwert (z.B. `/catalog/book/584493c1f4887f06c0e67d37)`.
- `catalog/<object>/create` — Das Formular zur Erstellung eines neuen Buches, einer Buchinstanz, eines Genres oder eines Autors (z.B. `/catalog/book/create)`.
- `catalog/<object>/<id>/update` — Das Formular zur Aktualisierung eines bestimmten Buches, einer Buchinstanz, eines Genres oder eines Autors mit dem gegebenen `_id` Feldwert (z.B. `/catalog/book/584493c1f4887f06c0e67d37/update)`.
- `catalog/<object>/<id>/delete` — Das Formular zur Löschung eines bestimmten Buches, einer Buchinstanz, eines Genres oder Autors mit dem gegebenen `_id` Feldwert (z.B. `/catalog/book/584493c1f4887f06c0e67d37/delete)`.

Die erste Startseite und die Listen-Seiten codieren keine zusätzlichen Informationen. Obwohl die zurückgegebenen Ergebnisse von dem Modelltyp und dem Inhalt in der Datenbank abhängen werden, sind die Anfragen, um die Informationen zu erhalten, immer gleich (ähnlich verhält es sich mit der Codeausführung für die Erstellung von Objekten).

Im Gegensatz dazu werden die anderen URLs verwendet, um auf ein spezifisches Dokument/Modellinstanz zu wirken - diese codieren die Identität des Elements in der URL (wie oben als `<id>` dargestellt). Wir werden Pfadparameter verwenden, um die codierten Informationen zu extrahieren und sie an den Routenhandler zu übergeben (und in einem späteren Artikel werden wir dies verwenden, um dynamisch zu bestimmen, welche Informationen wir aus der Datenbank abrufen sollen). Durch die Codierung der Informationen in unserer URL benötigen wir nur eine Route für jede Ressource eines bestimmten Typs (z.B. eine Route zur Anzeige jedes einzelnen Buchartikels).

> **Hinweis:** Express erlaubt Ihnen, Ihre URLs nach Belieben zu gestalten - Sie können Informationen im Körper der URL codieren, wie oben gezeigt, oder URL `GET` Parameter verwenden (z.B. `/book/?id=6`). Welchen Ansatz Sie auch wählen, die URLs sollten sauber, logisch und lesbar gehalten werden ([schauen Sie sich hier den Rat der W3C an](https://www.w3.org/Provider/Style/URI)).

Als Nächstes erst

ellen wir unsere Routenhandler-Callback-Funktionen und den Routencode für alle oben genannten URLs.

## Erstellen Sie die Routen-Handler-Callback-Funktionen

Bevor wir unsere Routen definieren, erstellen wir zuerst alle Dummy/Skelett-Callback-Funktionen, die sie aufrufen werden. Die Callbacks werden in separaten "Controller"-Modulen für `Book`, `BookInstance`, `Genre`, und `Author` gespeichert (Sie können jede Datei/Modulstruktur verwenden, aber dies scheint eine angemessene Granularität für dieses Projekt zu sein).

Beginnen Sie mit der Erstellung eines Ordners für unsere Controller im Projekt-Root (**/controllers**) und erstellen Sie dann separate Controller-Dateien/Module für die Handhabung jedes der Modelle:

```plain
/express-locallibrary-tutorial  //der Projekt-Root
  /controllers
    authorController.js
    bookController.js
    bookinstanceController.js
    genreController.js
```

Die Controller verwenden das `express-async-handler` Modul, daher sollten Sie es vor dem Fortfahren in die Bibliothek mit `npm` installieren:

```bash
npm install express-async-handler
```

### Autor-Controller

Öffnen Sie die Datei **/controllers/authorController.js** und geben Sie den folgenden Code ein:

```js
const Author = require("../models/author");
const asyncHandler = require("express-async-handler");

// Anzeigen der Liste aller Autoren.
exports.author_list = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Autorenliste");
});

// Detailseite für einen bestimmten Autor anzeigen.
exports.author_detail = asyncHandler(async (req, res, next) => {
  res.send(`NICHT IMPLEMENTIERT: Autorendetails: ${req.params.id}`);
});

// Anzeigen des Autoren-Erstellungsformulars bei GET.
exports.author_create_get = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Autor erstellen GET");
});

// Behandlung des Autoren-Erstellens bei POST.
exports.author_create_post = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Autor erstellen POST");
});

// Anzeigen des Autoren-Löschformulars bei GET.
exports.author_delete_get = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Autor löschen GET");
});

// Behandlung des Autoren-Löschens bei POST.
exports.author_delete_post = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Autor löschen POST");
});

// Anzeigen des Autoren-Aktualisierungsformulars bei GET.
exports.author_update_get = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Autor aktualisieren GET");
});

// Behandlung des Autoren-Aktualisierens bei POST.
exports.author_update_post = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Autor aktualisieren POST");
});
```

Das Modul benötigt zunächst das `Author` Modell, das wir später verwenden werden, um auf unsere Daten zuzugreifen und sie zu aktualisieren, und den `asyncHandler` Wrapper, den wir verwenden werden, um alle Ausnahmen zu fangen, die in unseren Routen

handler-Funktionen geworfen werden.
Es exportiert dann Funktionen für jede der URLs, die wir behandeln möchten.
Beachten Sie, dass die Erstellung, Aktualisierung und Löschung über Formulare erfolgen und daher zusätzliche Methoden zur Behandlung von Formular-Post-Anfragen haben — wir werden diese Methoden im "Formularartikel" später besprechen.

Die Funktionen verwenden alle die oben beschriebene Wrapper-Funktion in [Behandlung von Ausnahmen in Routenfunktionen](#handling_exceptions_in_route_functions), mit Argumenten für die Anfrage, die Antwort und den nächsten Schritt.
Die Funktionen antworten mit einer Zeichenkette, die darauf hinweist, dass die zugehörige Seite noch nicht erstellt wurde.
Wenn eine Controller-Funktion erwartet, dass sie Pfadparameter empfängt, werden diese in der Nachrichtenzeichenkette ausgegeben (siehe `req.params.id` oben).

Beachten Sie, dass einige Routenfunktionen nach der Implementierung möglicherweise keinen Code enthalten, der Ausnahmen auslösen kann.
Wir können diese zurück zu "normalen" Routenhandler-Funktionen ändern, wenn wir dazu kommen.

#### BookInstance-Controller

Öffnen Sie die Datei **/controllers/bookinstanceController.js** und kopieren Sie den folgenden Code hinein (dies folgt dem identischen Muster wie das `Author` Controller-Modul):

```js
const BookInstance = require("../models/bookinstance");
const asyncHandler = require("express-async-handler");

// Anzeige der Liste aller Buchinstanzen.
exports.bookinstance_list = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Liste der Buchinstanzen");
});

// Detailseite für eine bestimmte Buchinstanz anzeigen.
exports.bookinstance_detail = asyncHandler(async (req, res, next) => {
  res.send(`NICHT IMPLEMENTIERT: Detail der Buchinstanz: ${req.params.id}`);
});

// Anzeigen des Buchinstanzen-Erstellungsformulars bei GET.
exports.bookinstance_create_get = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Buchinstanz erstellen GET");
});

// Behandlung des Buchinstanzen-Erstellens bei POST.
exports.bookinstance_create_post = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Buchinstanz erstellen POST");
});

// Anzeigen des Buchinstanzen-Löschformulars bei GET.
exports.bookinstance_delete_get = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Buchinstanz löschen GET");
});

// Behandlung des Buchinstanzen-Löschens bei POST.
exports.bookinstance_delete_post = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Buchinstanz löschen POST");
});

// Anzeigen des Buchinstanzen-Aktualisierungsformulars bei GET.
exports.bookinstance_update_get = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Buchinstanz aktualisieren GET");
});

// Behandlung der Buchinstanzen-Aktualisierung bei POST.
exports.bookinstance_update_post = asyncHandler(async (req, res, next) => {
  res.send("NICHT IMPLEMENTIERT: Buchinstanz

 aktualisieren POST");
});
```
#### Genre-Controller

Öffnen Sie die Datei **/controllers/genreController.js** und kopieren Sie den folgenden Text hinein (dies folgt dem identischen Muster wie die Dateien `Author` und `BookInstance`):

```js
const Genre = require("../models/genre");
const asyncHandler = require("express-async-handler");

// Display list of all Genre.
exports.genre_list = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Genre list");
});

// Display detail page for a specific Genre.
exports.genre_detail = asyncHandler(async (req, res, next) => {
  res.send(`NOT IMPLEMENTED: Genre detail: ${req.params.id}`);
});

// Display Genre create form on GET.
exports.genre_create_get = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Genre create GET");
});

// Handle Genre create on POST.
exports.genre_create_post = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Genre create POST");
});

// Display Genre delete form on GET.
exports.genre_delete_get = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Genre delete GET");
});

// Handle Genre delete on POST.
exports.genre_delete_post = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Genre delete POST");
});

// Display Genre update form on GET.
exports.genre_update_get = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Genre update GET");
});

// Handle Genre update on POST.
exports.genre_update_post = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Genre update POST");
});
```

#### Buch-Controller

Öffnen Sie die Datei **/controllers/bookController.js** und kopieren Sie den folgenden Code hinein.
Dies folgt dem gleichen Muster wie die anderen Controller-Module, hat aber zusätzlich eine `index()`-Funktion zur Anzeige der Willkommensseite der Website:

```js
const Book = require("../models/book");
const asyncHandler = require("express-async-handler");

exports.index = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Site Home Page");
});

// Display list of all books.
exports.book_list = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Book list");
});

// Display detail page for a specific book.
exports.book_detail = asyncHandler(async (req, res, next) => {
  res.send(`NOT IMPLEMENTED: Book detail: ${req.params.id}`);
});

// Display book create form on GET.
exports.book_create_get = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Book create GET");
});

// Handle book create on POST.
exports.book_create_post = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Book create POST");
});

// Display book delete form on GET.
exports.book_delete_get = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Book delete GET");
});

// Handle book delete on POST.
exports.book_delete_post = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Book delete POST");
});

// Display book update form on GET.
exports.book_update_get = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Book update GET");
});

// Handle book update on POST.
exports.book_update_post = asyncHandler(async (req, res, next) => {
  res.send("NOT IMPLEMENTED: Book update POST");
});
```


