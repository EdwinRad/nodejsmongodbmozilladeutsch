## Mongoose installieren

Öffnen Sie ein Befehlsfenster und navigieren Sie zum Verzeichnis, in dem Sie Ihre [Grundstruktur der Local Library-Website](/de-DE/docs/Learn/Server-side/Express_Nodejs/skeleton_website) erstellt haben.
Geben Sie den folgenden Befehl ein, um Mongoose (und seine Abhängigkeiten) zu installieren und es zur **package.json**-Datei hinzuzufügen, falls Sie dies nicht bereits getan haben, als Sie den [Mongoose-Primer](#mongoose_und_mongodb_installieren) oben gelesen haben.

```bash
npm install mongoose
```

## Verbindung zu MongoDB herstellen

Öffnen Sie **/app.js** (im Stammverzeichnis Ihres Projekts) und kopieren Sie den folgenden Text unterhalb der Stelle, an der Sie das _Express Application Object_ deklarieren (nach der Zeile `const app = express();`). Ersetzen Sie den Datenbank-URL-String ('_insert_your_database_url_here_') durch die URL des Standorts, der Ihre eigene Datenbank repräsentiert (d. h. unter Verwendung der Informationen von _MongoDB Atlas_).

```js
// Mongoose-Verbindung einrichten
const mongoose = require("mongoose");
mongoose.set("strictQuery", false);
const mongoDB = "insert_your_database_url_here";

main().catch((err) => console.log(err));
async function main() {
  await mongoose.connect(mongoDB);
}
```

Dieser Code die Standardverbindung zur Datenbank und meldet etwaige Fehler in der Konsole.

## Definition des LocalLibrary-Schemas

Erstellen Sie zunächst einen Ordner für unsere Modelle im Projektstammverzeichnis (**/models**) und erstellen Sie dann separate Dateien für jedes der Modelle:

```plain
/express-locallibrary-tutorial  // Das Projektstammverzeichnis
  /models
    author.js
    book.js
    bookinstance.js
    genre.js
```

### Autorenmodell

Kopieren Sie den untenstehenden Code des `Author`-Schemas und fügen Sie ihn in Ihre Datei **./models/author.js** ein.
Das Schema definiert einen Autor mit `String`-SchemaTypes für den Vornamen und Nachnamen (erforderlich, mit einer maximalen Länge von 100 Zeichen) und `Date`-Feldern für das Geburts- und Sterbedatum.

```js
const mongoose = require("mongoose");

const Schema = mongoose.Schema;

const AuthorSchema = new Schema({
  first_name: { type: String, required: true, maxLength: 100 },
  family_name: { type: String, required: true, maxLength: 100 },
  date_of_birth: { type: Date },
  date_of_death: { type: Date },
});

// Virtuelles Attribut für den vollständigen Namen des Autors
AuthorSchema.virtual("name").get(function () {
  // Um Fehler zu vermeiden, wenn ein Autor keinen Nachnamen oder Vornamen hat,
  // stellen wir sicher, dass wir die Ausnahme behandeln, indem wir für diesen Fall einen leeren String zurückgeben
  let fullname = "";
  if (

this.first_name && this.family_name) {
    fullname = `${this.family_name}, ${this.first_name}`;
  }

  return fullname;
});

// Virtuelles Attribut für die URL des Autors
AuthorSchema.virtual("url").get(function () {
  // Wir verwenden keine Pfeilfunktion, da wir das this-Objekt benötigen
  return `/catalog/author/${this._id}`;
});

// Modell exportieren
module.exports = mongoose.model("Author", AuthorSchema);
```

Wir haben auch ein [virtuelles Attribut](#virtuelle_eigenschaften) für das AuthorSchema mit dem Namen "url" deklariert, das die absolute URL zurückgibt, die benötigt wird, um eine bestimmte Instanz des Modells zu erhalten. Wir werden dieses Attribut in unseren Vorlagen verwenden, wenn wir einen Link zu einem bestimmten Autor benötigen.

> **Hinweis:** Durch das Deklarieren der URLs als virtuelles Attribut im Schema wird sichergestellt, dass die URL für ein Element nur an einer Stelle geändert werden muss.
> Zu diesem Zeitpunkt würde ein Link mit dieser URL nicht funktionieren, da wir noch keine Routen für die Behandlung einzelner Modellinstanzen haben.
> Diese werden wir in einem späteren Artikel einrichten!

Am Ende des Moduls exportieren wir das Modell.

### Buchmodell

Kopieren Sie den untenstehenden Code des `Book`-Schemas und fügen Sie ihn in Ihre Datei **./models/book.js** ein.
Der Großteil davon ist ähnlich wie beim Autorenmodell - wir haben ein Schema mit mehreren String-Feldern und ein virtuelles Attribut zum Abrufen der URL bestimmter Buchaufzeichnungen definiert, und wir haben das Modell exportiert.

```js
const mongoose = require("mongoose");

const Schema = mongoose.Schema;

const BookSchema = new Schema({
  title: { type: String, required: true },
  author: { type: Schema.Types.ObjectId, ref: "Author", required: true },
  summary: { type: String, required: true },
  isbn: { type: String, required: true },
  genre: [{ type: Schema.Types.ObjectId, ref: "Genre" }],
});

// Virtuelles Attribut für die URL des Buches
BookSchema.virtual("url").get(function () {
  // Wir verwenden keine Pfeilfunktion, da wir das this-Objekt benötigen
  return `/catalog/book/${this._id}`;
});

// Modell exportieren
module.exports = mongoose.model("Book", BookSchema);
```

Der Hauptunterschied hier besteht darin, dass wir zwei Verweise auf andere Modelle erstellt haben:

- `author` ist ein Verweis auf ein einzelnes `Author`-Modellobjekt und ist erforderlich.
- `genre` ist ein Verweis auf ein Array von `Genre`-Modellobjekten. Wir haben dieses Objekt noch nicht deklariert!

### Buchinstanzmodell

Kopieren Sie schließlich den untenstehenden Code des `BookInstance`-Schemas und fügen Sie ihn in Ihre Datei **./models/bookinstance.js** ein.
Die `BookInstance` repräsentiert eine bestimmte Kopie eines Buches, die jemand ausleihen könnte, und enthält Informationen darüber, ob die Kopie verfügbar ist, wann sie zurück erwartet wird und Einzelheiten zur "Imprint" (oder Version).

```js
const mongoose = require("mongoose");

const Schema = mongoose.Schema;

const BookInstanceSchema

 = new Schema({
  book: { type: Schema.Types.ObjectId, ref: "Book", required: true }, // Verweis auf das zugehörige Buch
  imprint: { type: String, required: true },
  status: {
    type: String,
    required: true,
    enum: ["Available", "Maintenance", "Loaned", "Reserved"],
    default: "Maintenance",
  },
  due_back: { type: Date, default: Date.now },
});

// Virtuelles Attribut für die URL der Buchinstanz
BookInstanceSchema.virtual("url").get(function () {
  // Wir verwenden keine Pfeilfunktion, da wir das this-Objekt benötigen
  return `/catalog/bookinstance/${this._id}`;
});

// Modell exportieren
module.exports = mongoose.model("BookInstance", BookInstanceSchema);
```

Das Neue hier sind die Feldoptionen:

- `enum`: Dies ermöglicht es uns, die erlaubten Werte eines Strings festzulegen. In diesem Fall verwenden wir es, um den Verfügbarkeitsstatus unserer Bücher festzulegen (die Verwendung eines Enums bedeutet, dass wir Tippfehler und beliebige Werte für unseren Status verhindern können).
- `default`: Wir verwenden default, um den Standardstatus für neu erstellte Buchinstanzen auf "Maintenance" und das Standard-Datum `due_back` auf `now` festzulegen (beachten Sie, wie Sie die Date-Funktion aufrufen können, um das Datum festzulegen!).

Ansonsten sollte Ihnen alles aus unserem vorherigen Schema bekannt vorkommen.

### Genre-Modell - Herausforderung!

Öffnen Sie Ihre Datei **./models/genre.js** und erstellen Sie ein Schema zum Speichern von Genres (der Kategorie des Buches, z. B. ob es sich um Fiktion oder Sachbuch, Romanze oder Militärgeschichte handelt usw.).

Die Definition wird der der anderen Modelle sehr ähnlich sein:

- Das Modell sollte einen `String`-SchemaType namens `name` haben, um das Genre zu beschreiben.
- Dieser Name sollte erforderlich sein und zwischen 3 und 100 Zeichen haben.
- Deklarieren Sie ein [virtuelles Attribut](#virtuelle_eigenschaften) für die URL des Genres mit dem Namen `url`.
- Exportieren Sie das Modell.

## Testen - Erstellen Sie einige Objekte

Das ist es. Jetzt haben wir alle Modelle für die Website festgelegt!

Um die Modelle zu testen (und einige Beispielbücher und andere Objekte zu erstellen, die wir in unseren nächsten Artikeln verwenden können), führen wir nun ein eigenständiges Skript aus, um Objekte jeder Art zu erstellen:

1. Laden Sie die Datei [populatedb.js](https://raw.githubusercontent.com/mdn/express-locallibrary-tutorial/main/populatedb.js) herunter (oder erstellen Sie sie anderweitig) und speichern Sie sie in Ihrem _express-locallibrary-tutorial_-Verzeichnis (auf der gleichen Ebene wie `package.json`).

   > **Hinweis:** Sie müssen nicht wissen, wie `populatedb.js` funktioniert; es fügt einfach Beispieldaten in die Datenbank ein.

2. Führen Sie das Skript in Ihrem Befehlsfenster mit Node aus und geben Sie die URL Ihrer _MongoDB_-Datenbank an (die gleiche, mit der Sie den Platzhalter _

insert_your_database_url_here_ in `app.js` zuvor ersetzt haben):

   ```bash
   node populatedb <Ihre MongoDB-URL>
   ```
   > **Hinweis:** Sagt bescheid wenns hier garnicht klappt. Ich hatte hier viele Probleme

   > **Hinweis:** Unter Windows müssen Sie die Datenbank-URL in doppelte Anführungszeichen (") setzen.
   > In anderen Betriebssystemen benötigen Sie möglicherweise einfache (') Anführungszeichen.

1. Das Skript sollte bis zum Ende durchlaufen und dabei Objekte im Terminal anzeigen.

> **Hinweis:** Gehen Sie zu Ihrer Datenbank auf MongoDB Atlas (im Abschnitt _Collections_).
> Sie sollten nun in der Lage sein, in einzelne Sammlungen von Büchern, Autoren, Genres und Buchinstanzen zu navigieren und sich einzelne Dokumente anzusehen.

## Zusammenfassung

In diesem Artikel haben wir etwas über Datenbanken und ORMs in Node/Express gelernt und viel darüber erfahren, wie Mongoose-Schemata und -Modelle definiert werden. Wir haben dann dieses Wissen verwendet, um die Modelle `Book`, `BookInstance`, `Author` und `Genre` für die LocalLibrary-Website zu entwerfen und zu implementieren.

Zu guter Letzt haben wir unsere Modelle getestet, indem wir eine Reihe von Instanzen erstellt haben (mithilfe eines eigenständigen Skripts). Im nächsten Artikel werden wir einige Seiten erstellen, um diese Objekte anzuzeigen.