## Erstellen Sie das Katalog-Routen-Modul

Als nächstes erstellen wir _Routen_ für alle URLs, [die für die LocalLibrary-Website benötigt werden](#routes_needed_for_the_locallibrary), die die Controller-Funktionen aufrufen, die wir in den vorherigen Abschnitten definiert haben.

Das Skelett hat bereits einen **./routes** Ordner, der Routen für _index_ und _users_ enthält.
Erstellen Sie eine weitere Routendatei - **catalog.js** - in diesem Ordner, wie unten gezeigt.

```plain
/express-locallibrary-tutorial //das Projektverzeichnis
  /routes
    index.js
    users.js
    catalog.js
```

Öffnen Sie **/routes/catalog.js** und kopieren Sie den unten stehenden Code hinein:

```js
const express = require("express");
const router = express.Router();

// Require controller modules.
const book_controller = require("../controllers/bookController");
const author_controller = require("../controllers/authorController");
const genre_controller = require("../controllers/genreController");
const book_instance_controller = require("../controllers/bookinstanceController");

/// BOOK ROUTES ///

// GET catalog home page.
router.get("/", book_controller.index);

// GET request for creating a Book. NOTE This must come before routes that display Book (uses id).
router.get("/book/create", book_controller.book_create_get);

// POST request for creating Book.
router.post("/book/create", book_controller.book_create_post);

// GET request to delete Book.
router.get("/book/:id/delete", book_controller.book_delete_get);

// POST request to delete Book.
router.post("/book/:id/delete", book_controller.book_delete_post);

// GET request to update Book.
router.get("/book/:id/update", book_controller.book_update_get);

// POST request to update Book.
router.post("/book/:id/update", book_controller.book_update_post);

// GET request for one Book.
router.get("/book/:id", book_controller.book_detail);

// GET request for list of all Book items.
router.get("/books", book_controller.book_list);

/// AUTHOR ROUTES ///

// GET request for creating Author. NOTE This must come before route for id (i.e. display author).
router.get("/author/create", author_controller.author_create_get);

// POST request for creating Author.
router.post("/author/create", author_controller.author_create_post);

// GET request to delete Author.
router.get("/author/:id/delete", author_controller.author_delete_get);

// POST request to delete Author.
router.post("/author/:id/delete", author_controller.author_delete_post);

// GET request to update Author.
router.get("/author/:id/update", author_controller.author_update_get);

// POST request to update Author.
router.post("/author/:id/update", author_controller.author_update_post);

// GET request for one Author.
router.get("/author/:id", author_controller.author_detail);

// GET request for list of all Authors.
router.get("/authors", author_controller.author_list);

/// GENRE ROUTES ///

// GET request for creating a Genre. NOTE This must come before route that displays Genre (uses id).
router.get("/genre/create", genre_controller.genre_create_get);

//POST request for creating Genre.
router.post("/genre/create", genre_controller.genre_create_post);

// GET request to delete Genre.
router.get("/genre/:id/delete", genre_controller.genre_delete_get);

// POST request to delete Genre.
router.post("/genre/:id/delete", genre_controller.genre_delete_post);

// GET request to update Genre.
router.get("/genre/:id/update", genre_controller.genre_update_get);

// POST request to update Genre.
router.post("/genre/:id/update", genre_controller.genre_update_post);

// GET request for one Genre.
router.get("/genre/:id", genre_controller.genre_detail);

// GET request for list of all Genre.
router.get("/genres", genre_controller.genre_list);

/// BOOKINSTANCE ROUTES ///

// GET request for creating a BookInstance. NOTE This must come before route that displays BookInstance (uses id).
router.get(
  "/bookinstance/create",
  book_instance_controller.bookinstance_create_get
);

// POST request for creating BookInstance.
router.post(
  "/bookinstance/create",
  book_instance_controller.bookinstance_create_post
);

// GET request to delete BookInstance.
router.get(
  "/bookinstance/:id/delete",
  book_instance_controller.bookinstance_delete_get
);

// POST request to delete BookInstance.
router.post(
  "/bookinstance/:id/delete",
  book_instance_controller.bookinstance_delete_post
);

// GET request to update BookInstance.
router.get(
  "/bookinstance/:id/update",
  book_instance_controller.bookinstance_update_get
);

// POST request to update BookInstance.
router.post(
  "/bookinstance/:id/update",
  book_instance_controller.bookinstance_update_post
);

// GET request for one BookInstance.
router.get("/bookinstance/:id", book_instance_controller.bookinstance_detail);

// GET request for list of all BookInstance.
router.get("/bookinstances", book_instance_controller.bookinstance_list);

module.exports = router;
```

### Aktualisieren Sie das Index-Routen-Modul
Wir haben alle unsere neuen Routen eingerichtet, aber wir haben immer noch eine Route zur ursprünglichen Seite. Lassen Sie uns diese stattdessen auf die neue Index-Seite umleiten, die wir unter dem Pfad '/catalog' erstellt haben.

Öffnen Sie /routes/index.js und ersetzen Sie die vorhandene Route durch die untenstehende Funktion.


```js
// GET home page.
router.get("/", function (req, res) {
  res.redirect("/catalog");
});
```


### Aktualisieren Sie app.js

Der letzte Schritt besteht darin, die Routen zur Middleware-Kette hinzuzufügen.
Dies tun wir in `app.js`.

Öffnen Sie **app.js** und fordern Sie die Katalogroute unter den anderen Routen an (fügen Sie die dritte unten gezeigte Zeile hinzu, unter den anderen beiden, die bereits in der Datei vorhanden sein sollten):

```js
var indexRouter = require("./routes/index");
var usersRouter = require("./routes/users");
const catalogRouter = require("./routes/catalog"); //Import routes for "catalog" area of site
```

Als nächstes fügen Sie die Katalogroute zur Middleware-Stack unter den anderen Routen hinzu (fügen Sie die dritte unten gezeigte Zeile hinzu, unter den anderen beiden, die bereits in der Datei vorhanden sein sollten):

```js
app.use("/", indexRouter);
app.use("/users", usersRouter);
app.use("/catalog", catalogRouter); // Add catalog routes to middleware chain.
```

> **Hinweis:** Wir haben unser Katalogmodul unter dem Pfad `'/catalog'` hinzugefügt. Dies wird allen in dem Katalogmodul definierten Pfaden vorangestellt. Um also beispielsweise eine Liste von Büchern abzurufen, lautet die URL: `/catalog/books/`.

Das war's. Wir sollten jetzt Routen und Skelettfunktionen für alle URLs aktiviert haben, die wir schließlich auf der LocalLibrary-Website unterstützen werden.

### Testen der Routen

Um die Routen zu testen, starten Sie zuerst die Website auf die übliche Weise.

- Die Standardmethode

  ```bash
  // Windows
  SET DEBUG=express-locallibrary-tutorial:* & npm start

  // macOS or Linux
  DEBUG=express-locallibrary-tutorial:* npm start
  ```


Wenn Sie zuvor nodemon eingerichtet haben, können Sie stattdessen dies verwenden:

  ```bash
  npm run serverstart
  ```

Navigieren Sie dann zu einer Reihe von LocalLibrary-URLs und stellen Sie sicher, dass Sie keine Fehlerseite (HTTP 404) erhalten. Eine kleine Auswahl an URLs finden Sie unten für Ihre Bequemlichkeit:


- `http://localhost:3000/`
- `http://localhost:3000/catalog`
- `http://localhost:3000/catalog/books`
- `http://localhost:3000/catalog/bookinstances/`
- `http://localhost:3000/catalog/authors/`
- `http://localhost:3000/catalog/genres/`
- `http://localhost:3000/catalog/book/5846437593935e2f8c2aa226`
- `http://localhost:3000/catalog/book/create`

### Zusammenfassung
Wir haben nun alle Routen für unsere Website erstellt, zusammen mit Dummy-Controller-Funktionen, die wir in späteren Artikeln mit einer vollständigen Implementierung füllen können. Dabei haben wir viel Grundlegendes über Express-Routen, das Handling von Ausnahmen und einige Ansätze zur Strukturierung unserer Routen und Controller gelernt.

Im nächsten Artikel erstellen wir eine richtige Willkommensseite für die Website, mit Hilfe von Ansichten (Templates) und Informationen, die in unseren Modellen gespeichert sind.