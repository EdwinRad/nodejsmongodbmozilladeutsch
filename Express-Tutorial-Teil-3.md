Dieser Artikel gibt einen kurzen Überblick über Datenbanken und wie sie mit Node/Express-Apps verwendet werden können. Anschließend wird gezeigt, wie wir [Mongoose](https://mongoosejs.com/) verwenden können, um den Zugriff auf die Datenbank für die Website [LocalLibrary](/de-DE/docs/Learn/Server-side/Express_Nodejs/Tutorial_local_library_website) zu ermöglichen. Es wird erklärt, wie Objektschemata und Modelle deklariert werden, welche Hauptfeldtypen es gibt und welche grundlegende Validierung möglich ist. Außerdem werden kurz einige der wichtigsten Möglichkeiten gezeigt, auf Modelldaten zuzugreifen.

<table>
  <tbody>
    <tr>
      <th scope="row">Voraussetzungen:</th>
      <td>
        <a href="/de-DE/docs/Learn/Server-side/Express_Nodejs/skeleton_website">Express-Tutorial Teil 2: Erstellung einer Grundstruktur für eine Website</a>
      </td>
    </tr>
    <tr>
      <th scope="row">Ziel:</th>
      <td>Eigene Modelle mit Mongoose entwerfen und erstellen können.</td>
    </tr>
  </tbody>
</table>

## Überblick

Das Personal der Bibliothek wird die Website Local Library verwenden, um Informationen über Bücher und Ausleiher zu speichern, während die Bibliotheksbenutzer sie nutzen können, um nach Büchern zu suchen und zu stöbern, herauszufinden, ob Exemplare verfügbar sind, und diese dann zu reservieren oder auszuleihen. Um Informationen effizient zu speichern und abzurufen, werden wir sie in einer _Datenbank_ speichern.

Express-Apps können viele verschiedene Datenbanken verwenden, und es gibt mehrere Ansätze für die Durchführung von **C**reate, **R**ead, **U**pdate und **D**elete (CRUD)-Operationen. Dieses Tutorial gibt einen kurzen Überblick über einige der verfügbaren Optionen und zeigt dann im Detail die ausgewählten Mechanismen.

### Welche Datenbanken kann ich verwenden?

Express-Apps können jede von _Node_ unterstützte Datenbank verwenden (_Express_ selbst definiert keine spezifischen zusätzlichen Verhaltensweisen/Anforderungen für die Datenbankverwaltung). Es gibt [viele beliebte Optionen](https://expressjs.com/de/guide/database-integration.html), darunter PostgreSQL, MySQL, Redis, SQLite und MongoDB.

Bei der Auswahl einer Datenbank sollten Sie Dinge wie Time-to-Productivity/Lernkurve, Leistung, Benutzerfreundlichkeit bei Replikation/Backup, Kosten, Community-Support usw. berücksichtigen. Obwohl es keine einzelne "beste" Datenbank gibt, sollte nahezu jede der beliebten Lösungen für eine kleine bis mittelgroße Website wie unsere Local Library mehr als ausreichend sein.

Weitere Informationen zu den Optionen finden Sie unter [Datenbankintegration](https://expressjs.com/de/guide/database-integration.html) (Express-Dokumentation).

### Was ist der beste Weg zur Interaktion mit einer Datenbank?

Es gibt zwei gängige Ansätze für die Interaktion mit einer Datenbank:

- Verwendung der nativen Abfragesprache der Datenbank, z. B. SQL.
- Verwendung eines Objektdatenmodells ("ODM") oder eines objektorientierten Daten

modells ("ORM"). Ein ODM/ORM stellt die Daten der Website als JavaScript-Objekte dar, die dann auf die zugrunde liegende Datenbank abgebildet werden. Einige ORMs sind an eine bestimmte Datenbank gebunden, während andere einen datenbankunabhängigen Backend bieten.

Die beste _Leistung_ kann erzielt werden, indem SQL oder die von der Datenbank unterstützte Abfragesprache verwendet wird. ODMs sind oft langsamer, da sie Übersetzungscodes verwenden, um zwischen Objekten und dem Datenbankformat zu vermitteln, was möglicherweise nicht die effizientesten Datenbankabfragen verwendet (dies gilt insbesondere, wenn das ODM verschiedene Datenbank-Backends unterstützt und größere Kompromisse hinsichtlich der unterstützten Datenbankfunktionen eingehen muss).

Der Vorteil der Verwendung eines ORM besteht darin, dass Programmierer weiterhin in JavaScript-Objekten statt in Datenbanksemantiken denken können - dies gilt insbesondere, wenn Sie mit verschiedenen Datenbanken arbeiten müssen (auf derselben oder verschiedenen Websites). Sie bieten auch einen offensichtlichen Ort, um die Validierung von Daten durchzuführen.

> **Hinweis:** Die Verwendung von ODM/ORMs führt oft zu niedrigeren Kosten für Entwicklung und Wartung! Wenn Sie nicht sehr vertraut mit der nativen Abfragesprache sind oder wenn die Leistung von entscheidender Bedeutung ist, sollten Sie ernsthaft in Betracht ziehen, ein ODM zu verwenden.

### Welches ORM/ODM sollte ich verwenden?

Auf der npm-Paketmanager-Website gibt es viele ODM/ORM-Lösungen (schauen Sie sich die Tags [odm](https://www.npmjs.com/search?q=keywords:odm) und [orm](https://www.npmjs.com/search?q=keywords:orm) an!).

Einige Lösungen, die zum Zeitpunkt der Erstellung dieses Artikels beliebt waren:

- [Mongoose](https://www.npmjs.com/package/mongoose): Mongoose ist ein objektorientiertes Modellierungswerkzeug für [MongoDB](https://www.mongodb.com/), das in einer asynchronen Umgebung arbeitet.
- [Waterline](https://www.npmjs.com/package/waterline): Ein ODM, das aus dem auf Express basierenden Webframework [Sails](https://sailsjs.com/) extrahiert wurde. Es bietet eine einheitliche API zum Zugriff auf zahlreiche verschiedene Datenbanken, darunter Redis, MySQL, LDAP, MongoDB und Postgres.
- [Bookshelf](https://www.npmjs.com/package/bookshelf): Unterstützt sowohl promise-basierte als auch traditionelle Callback-Schnittstellen, bietet Transaktionsunterstützung, vorzeitiges/eingebettetes Laden von Beziehungen, polymorphe Assoziationen und Unterstützung für Eins-zu-eins-, Eins-zu-viele- und viele-zu-viele-Beziehungen. Funktioniert mit PostgreSQL, MySQL und SQLite3.
- [Objection](https://www.npmjs.com/package/objection): Ermöglicht die Nutzung der vollen Leistungsfähigkeit von SQL und des zugrunde liegenden Datenbank-Motors (unterstützt SQLite3, Postgres und MySQL).
- [Sequelize](https://www.npmjs.com/package/sequelize) ist ein promise-b

asiertes ORM für Node.js und io.js. Es unterstützt die Dialekte PostgreSQL, MySQL, MariaDB, SQLite und MSSQL und bietet solide Transaktionsunterstützung, Beziehungen, Lese-Replikation und mehr.
- [Node ORM2](https://node-orm.readthedocs.io/en/latest/) ist ein Objekt-Relationship-Manager für NodeJS. Es unterstützt MySQL, SQLite und Progress und hilft beim Arbeiten mit der Datenbank mit einem objektorientierten Ansatz.
- [GraphQL](https://graphql.org/): Hauptsächlich eine Abfragesprache für RESTful APIs, ist GraphQL sehr beliebt und bietet Funktionen zum Lesen von Daten aus Datenbanken.

Als Faustregel sollten Sie sowohl die bereitgestellten Funktionen als auch die "Community-Aktivitäten" (Downloads, Beiträge, Fehlerberichte, Qualität der Dokumentation usw.) bei der Auswahl einer Lösung berücksichtigen. Zum Zeitpunkt der Erstellung dieses Artikels war Mongoose mit Abstand das beliebteste ODM und ist eine vernünftige Wahl, wenn Sie MongoDB für Ihre Datenbank verwenden.

### Verwendung von Mongoose und MongoDB für die Local Library

Für das Beispiel der _Local Library_ (und den Rest dieses Themas) verwenden wir das [Mongoose ODM](https://www.npmjs.com/package/mongoose), um auf unsere Bibliotheksdaten zuzugreifen. Mongoose fungiert als Schnittstelle für [MongoDB](https://www.mongodb.com/what-is-mongodb), eine Open-Source-[NoSQL](https://en.wikipedia.org/wiki/NoSQL)-Datenbank, die ein dokumentenorientiertes Datenmodell verwendet. Eine "Sammlung" von "Dokumenten" in einer MongoDB-Datenbank [ist analog zu](https://www.mongodb.com/docs/manual/core/databases-and-collections/) einer "Tabelle" von "Zeilen" in einer relationalen Datenbank.

Diese Kombination aus ODM und Datenbank ist äußerst beliebt in der Node-Community, teilweise weil die Speicherung und Abfrage von Dokumenten sehr ähnlich zu JSON aussieht und JavaScript-Entwicklern daher vertraut ist.

> **Hinweis:** Sie müssen MongoDB nicht kennen, um Mongoose verwenden zu können, obwohl Teile der [Mongoose-Dokumentation](https://mongoosejs.com/docs/guide.html) einfacher zu verwenden und zu verstehen sind, wenn Sie bereits mit MongoDB vertraut sind.

Der Rest dieses Tutorials zeigt, wie das Mongoose-Schema und die Modelle für das Beispiel der [LocalLibrary-Website](/de-DE/docs/Learn/Server-side/Express_Nodejs/Tutorial_local_library_website) definiert und darauf zugegriffen werden können.

## Entwurf der Modelle für die Local Library

Bevor Sie loslegen und mit der Codierung der Modelle beginnen, sollten Sie sich einige Minuten Zeit nehmen, um darüber nachzudenken, welche Daten wir speichern müssen und welche Beziehungen zwischen den verschiedenen Objekten bestehen.

Wir wissen, dass wir Informationen über Bücher (Titel, Zusammenfassung, Autor, Genre, ISBN) speichern müssen und dass wir möglicherweise mehrere Exemplare davon haben (mit global eindeutigen IDs, Verfügbarkeitsstatus usw.). Wir möchten möglicherweise zusätzliche Informationen über den Autor speichern, nicht nur den Namen, und es könnten mehrere Autoren mit dem

selben oder ähnlichen Namen vorhanden sein. Wir möchten Informationen nach Buchtitel, Autor, Genre und Kategorie sortieren können.

Wenn Sie Ihre Modelle entwerfen, macht es Sinn, für jedes "Objekt" (eine Gruppe von zusammengehörigen Informationen) separate Modelle zu haben. In diesem Fall sind Bücher, Buchinstanzen und Autoren offensichtliche Kandidaten für diese Modelle.

Sie möchten möglicherweise auch Modelle verwenden, um Auswahlmöglichkeiten (z. B. eine Dropdown-Liste mit Auswahlmöglichkeiten) darzustellen, anstatt die Auswahlmöglichkeiten direkt in die Website einzubetten. Dies ist empfehlenswert, wenn nicht alle Optionen im Voraus bekannt sind oder sich ändern können. Ein gutes Beispiel ist ein Genre (z. B. Fantasy, Science-Fiction usw.).

Nachdem wir uns für unsere Modelle und Felder entschieden haben, müssen wir über die Beziehungen zwischen ihnen nachdenken.

In diesem Sinne zeigt das UML-Assoziationsdiagramm unten die Modelle, die wir in diesem Fall definieren (als Kästchen). Wie oben diskutiert, haben wir Modelle für das Buch (die allgemeinen Details des Buches), die Buchinstanz (Status bestimmter physischer Exemplare des Buches im System) und den Autor erstellt. Wir haben uns auch entschieden, ein Modell für das Genre zu haben, damit Werte dynamisch erstellt werden können. Wir haben uns dafür entschieden, kein Modell für den `BookInstance:status` zu haben - wir werden die akzeptablen Werte fest kodieren, da wir nicht erwarten, dass sich diese ändern. In jedem der Kästchen sehen Sie den Modellnamen, die Feldnamen und -typen sowie die Methoden und deren Rückgabetypen.

Das Diagramm zeigt auch die Beziehungen zwischen den Modellen, einschließlich ihrer _Multiplizitäten_. Die Multiplizitäten sind die Zahlen auf dem Diagramm, die die Anzahl (maximal und minimal) jedes Modells anzeigen, die in der Beziehung vorhanden sein können. Zum Beispiel zeigt die Verbindungslinie zwischen den Kästchen, dass ein `Book` und ein `Genre` miteinander verknüpft sind. Die Zahlen in der Nähe des `Book`-Modells zeigen, dass ein `Genre` null oder mehr `Book`s haben muss (so viele wie Sie möchten), während die Zahlen am anderen Ende der Linie neben dem `Genre` zeigen, dass ein Buch null oder mehr zugehörige `Genre`s haben kann.

> **Hinweis:** Wie in unserem [Mongoose-Primer](#mongoose_primer) weiter unten diskutiert, ist es oft besser, das Feld, das die Beziehung zwischen den Dokumenten/Modellen definiert, in nur _einem_ Modell zu haben (Sie können die umgekehrte Beziehung immer finden, indem Sie nach der entsprechenden `_id` im anderen Modell suchen). Unten haben wir uns dafür entschieden, die Beziehung zwischen `Book`/`Genre` und `Book`/`Author` im Buch-Schema zu definieren und die Beziehung zwischen `Book`/`BookInstance` im `BookInstance`-Schema zu definieren. Diese Wahl war weitgehend willkürlich - wir hätten genauso gut das Feld im anderen Schema haben können.

![Mongoose Library

 Model mit richtiger Kardinalität](library_website_-_mongoose_express.png)

> **Hinweis:** Der nächste Abschnitt gibt eine grundlegende Einführung in die Definition und Verwendung von Modellen. Während Sie ihn lesen, überlegen Sie, wie Sie jedes der Modelle in dem oben gezeigten Diagramm erstellen würden.

### Datenbank-APIs sind asynchron

Datenbankmethoden zum Erstellen, Suchen, Aktualisieren oder Löschen von Datensätzen sind asynchron.
Das bedeutet, dass die Methoden sofort zurückkehren und der Code zum Umgang mit dem Erfolg oder dem Fehlschlagen der Methode zu einem späteren Zeitpunkt ausgeführt wird, wenn die Operation abgeschlossen ist.
Während der Server auf den Abschluss der Datenbankoperation wartet, kann anderer Code ausgeführt werden, sodass der Server auf andere Anfragen reagieren kann.

JavaScript verfügt über mehrere Mechanismen, um asynchrones Verhalten zu unterstützen.
Historisch gesehen wurde in JavaScript häufig stark auf das Übergeben von [Callback-Funktionen](/de-DE/docs/Learn/JavaScript/Asynchronous/Introducing) an asynchrone Methoden zurückgegriffen, um den Erfolgs- und Fehlerfall zu behandeln.
In modernem JavaScript wurden Callbacks weitgehend durch [Promises](/de-DE/docs/Web/JavaScript/Reference/Global_Objects/Promise) ersetzt. Promises sind Objekte, die (sofort) von einer asynchronen Methode zurückgegeben werden und ihren zukünftigen Zustand repräsentieren.
Wenn die Operation abgeschlossen ist, wird das Promise-Objekt "abgewickelt" und gibt ein Objekt zurück, das das Ergebnis der Operation oder einen Fehler repräsentiert.

Es gibt zwei Hauptwege, wie Sie Promises verwenden können, um Code auszuführen, wenn ein Promise abgewickelt wurde, und wir empfehlen Ihnen dringend, [Wie man Promises verwendet](/de-DE/docs/Learn/JavaScript/Asynchronous/Promises) für eine Übersicht über beide Ansätze zu lesen.
In diesem Tutorial verwenden wir hauptsächlich [`await`](/de-DE/docs/Web/JavaScript/Reference/Operators/await), um auf den Abschluss von Promises innerhalb einer [`async-Funktion`](/de-DE/docs/Web/JavaScript/Reference/Statements/async_function) zu warten, da dies zu einem lesbareren und verständlicheren asynchronen Code führt.

Die Arbeitsweise dieses Ansatzes ist wie folgt: Sie verwenden das Schlüsselwort `async function`, um eine Funktion als asynchron zu markieren, und wenden dann innerhalb dieser Funktion `await` auf jede Methode an, die ein Promise zurückgibt.
Wenn die asynchrone Funktion ausgeführt wird, wird ihre Ausführung an der ersten `await`-Methode angehalten, bis das Promise abgewickelt ist.
Aus Sicht des umgebenden Codes gibt die asynchrone Funktion dann zurück und der Code danach kann ausgeführt werden.
Später, wenn das Promise abgewickelt wird, gibt die `await`-Methode innerhalb der asynchronen Funktion das Ergebnis zurück, oder es wird ein Fehler geworfen, wenn das Promise abgelehnt wurde.
Der Code in der asynchronen Funktion wird dann ausgeführt, bis ein weiteres `await` auftritt, an dem es erneut angehalten wird, oder

 bis der gesamte Code in der Funktion ausgeführt wurde.

Sie können sehen, wie dies im folgenden Beispiel funktioniert.
`myFunction()` ist eine asynchrone Funktion, die in einem [`try...catch`](/de-DE/docs/Web/JavaScript/Reference/Statements/try...catch)-Block aufgerufen wird.
Wenn `myFunction()` ausgeführt wird, wird die Codeausführung an `methodThatReturnsPromise()` angehalten, bis das Promise gelöst ist, und der Code wird dann zu `aFunctionThatReturnsPromise()` weitergeführt und wartet erneut.
Der Code im `catch`-Block wird ausgeführt, wenn in der asynchronen Funktion ein Fehler geworfen wird, und dies geschieht, wenn das Promise, das von einer der Methoden zurückgegeben wird, abgelehnt wird.

```js
async function myFunction {
  // ...
  await someObject.methodThatReturnsPromise();
  // ...
  await aFunctionThatReturnsPromise();
  // ...
}

try {
  // ...
  myFunction();
  // ...
} catch (e) {
 // Fehlerbehandlungscode
}
```

Die oben genannten asynchronen Methoden werden nacheinander ausgeführt.
Wenn die Methoden voneinander unabhängig sind, können Sie sie parallel ausführen und die gesamte Operation schneller abschließen.
Dies erfolgt mit Hilfe der Methode [`Promise.all()`](/de-DE/docs/Web/JavaScript/Reference/Global_Objects/Promise/all), die einen Iterable von Promises als Eingabe entgegennimmt und ein einzelnes `Promise` zurückgibt.
Dieses zurückgegebene Promise wird erfüllt, wenn alle Promises in der Eingabe erfüllt werden, und gibt ein Array mit den Erfüllungswerten zurück.
Es wird abgelehnt, wenn eines der Promises in der Eingabe abgelehnt wird, und gibt den Grund für die erste Ablehnung zurück.

Der folgende Code zeigt, wie dies funktioniert.
Zunächst haben wir zwei Funktionen, die Promises zurückgeben.
Wir verwenden `await`, um auf die Fertigstellung beider Promises mithilfe des von `Promise.all()` zurückgegebenen Promises zu warten.
Sobald beide abgeschlossen sind, gibt `await` zurück, und das Ergebnisarray wird befüllt.
Die Funktion fährt dann mit dem nächsten `await` fort und wartet darauf, dass das Promise, das von `anotherFunctionThatReturnsPromise()` zurückgegeben wird, abgewickelt wird.
Sie würden den Aufruf von `myFunction()` in einem `try...catch`-Block durchführen, um etwaige Fehler abzufangen.

```js
async function myFunction {
  // ...
  const [resultFunction1, resultFunction2] = await Promise.all([
     functionThatReturnsPromise1(),
     functionThatReturnsPromise2()
  ]);
  // ...
  await anotherFunctionThatReturnsPromise(resultFunction1);
}
```

Promises mit `await`/`async` ermöglichen eine flexible und "nachvollziehbare" Steuerung der asynchronen Ausführung!

## Mongoose-Einführung

Dieser Abschnitt gibt einen Überblick darüber, wie Mongoose mit einer MongoDB-Datenbank verbunden wird, wie ein Schema und ein Modell definiert werden und wie grundlegende Abfragen durchgeführt werden.

> **Hinweis:** Diese Einführung basiert stark auf dem [Mongoose-Quickstart](https://www.npm

js.com/package/mongoose) auf _npm_ und der [offiziellen Dokumentation](https://mongoosejs.com/docs/guide.html).

### Installation von Mongoose und MongoDB

Mongoose wird in Ihrem Projekt (**package.json**) wie jede andere Abhängigkeit mit npm installiert.
Verwenden Sie den folgenden Befehl in Ihrem Projektordner, um es zu installieren:

```bash
npm install mongoose
```

Die Installation von _Mongoose_ fügt alle seine Abhängigkeiten hinzu, einschließlich des MongoDB-Datenbanktreibers, installiert jedoch nicht MongoDB selbst. Wenn Sie einen MongoDB-Server installieren möchten, können Sie von [hier aus Installationsprogramme herunterladen](https://www.mongodb.com/try/download/community) für verschiedene Betriebssysteme und es lokal installieren. Sie können auch cloud-basierte MongoDB-Instanzen verwenden.

> **Hinweis:** Für dieses Tutorial verwenden wir die Cloud-basierte _Datenbank als Dienst_ MongoDB Atlas im kostenlosen Tarif, um die Datenbank bereitzustellen. Dies ist für die Entwicklung geeignet und macht für das Tutorial Sinn, da die "Installation" unabhängig vom Betriebssystem ist (Datenbank-als-Dienst ist auch ein Ansatz, den Sie für Ihre Produktionsdatenbank verwenden könnten).

### Verbindung mit MongoDB

Mongoose erfordert eine Verbindung zu einer MongoDB-Datenbank.
Sie können eine Verbindung zu einer lokal gehosteten Datenbank mit `mongoose.connect()` herstellen, wie im folgenden Beispiel gezeigt (für das Tutorial stellen wir stattdessen eine Verbindung zu einer im Internet gehosteten Datenbank her).

```js
// Importieren des Mongoose-Moduls
const mongoose = require("mongoose");

// Setzen von `strictQuery: false`, um global die Filterung von Eigenschaften zu ermöglichen, die nicht im Schema vorhanden sind
// Hinzugefügt, um vorbereitende Warnungen für Mongoose 7 zu entfernen.
// Siehe: https://mongoosejs.com/docs/migrating_to_6.html#strictquery-is-removed-and-replaced-by-strict
mongoose.set("strictQuery", false);

// Definieren der Datenbank-URL, zu der eine Verbindung hergestellt werden soll.
const mongoDB = "mongodb://127.0.0.1/my_database";

// Auf Verbindung zur Datenbank warten und bei Problemen einen Fehler protokollieren
main().catch((err) => console.log(err));
async function main() {
  await mongoose.connect(mongoDB);
}
```

> **Hinweis:** Wie im Abschnitt [Datenbank-APIs sind asynchron](#database_apis_are_asynchronous) erläutert, verwenden wir hier `await`, um auf das von der Methode `connect()` zurückgegebene Promise in einer mit `async function` deklarierten Funktion zu warten (siehe Abschnitt).
> Wir verwenden den `catch()`-Handler des Promises, um etwaige Fehler beim Verbindungsversuch zu behandeln, aber wir könnten auch `main()` in einem `try...catch`-Block aufgerufen haben.

Sie können das standardmäßige `Connection`-Objekt mit `mongoose.connection` abrufen.
Wenn Sie zusätzliche Verbindungen erstellen möchten, können Sie `mongoose.createConnection()` verwenden.
Dies nimmt die gleiche Form einer Datenbank-URI (mit Host, Datenbank, Port, Optionen usw.) wie `connect()` an und gibt ein `Connection`-Objekt zurück).
Beachten Sie, dass `createConnection()` sofort zurückgegeben wird. Wenn Sie auf den Aufbau der Verbindung warten müssen, können Sie es mit `asPromise()` aufrufen, um ein Promise zurückzugeben (`mongoose.createConnection(mongoDB).asPromise()`).

### Definition und Erstellung von Modellen

Modelle werden mithilfe der `Schema`-Schnittstelle aus Schemas _definiert_. Das Schema ermöglicht es Ihnen, die in jedem Dokument gespeicherten Felder zusammen mit ihren Validierungsanforderungen und Standardwerten zu definieren. Darüber hinaus können Sie statische und Instanzhilfsmethoden definieren, um die Arbeit mit Ihren Datentypen zu erleichtern, sowie virtuelle Eigenschaften, die Sie wie jedes andere Feld verwenden können, die aber tatsächlich nicht in der Datenbank gespeichert werden (dazu kommen wir weiter unten noch).

Schemas werden dann mit der Methode `mongoose.model()` in Modelle "kompiliert". Sobald Sie ein Modell haben, können Sie es verwenden, um Objekte des entsprechenden Typs zu suchen, zu erstellen, zu aktualisieren und zu löschen.

> **Hinweis:** Jedes Modell ordnet einer _Sammlung_ von _Dokumenten_ in

 der MongoDB-Datenbank zu. Die Dokumente enthalten die im Modell-`Schema` definierten Felder/Schema-Typen.

#### Schemas definieren

Der folgende Codeausschnitt zeigt, wie Sie ein einfaches Schema definieren könnten. Zuerst verwenden Sie `require()` für Mongoose und verwenden dann den Schema-Konstruktor, um eine neue Schema-Instanz zu erstellen und die verschiedenen Felder darin im Objektparameter des Konstruktors zu definieren.

```js
// Mongoose importieren
const mongoose = require("mongoose");

// Ein Schema definieren
const Schema = mongoose.Schema;

const SomeModelSchema = new Schema({
  a_string: String,
  a_date: Date,
});
```

In obigem Beispiel haben wir nur zwei Felder, einen String und ein Datum. In den nächsten Abschnitten zeigen wir einige weitere Feldtypen, Validierung und andere Methoden.

#### Ein Modell erstellen

Modelle werden aus Schemas mit der Methode `mongoose.model()` erstellt:

```js
// Schema definieren
const Schema = mongoose.Schema;

const SomeModelSchema = new Schema({
  a_string: String,
  a_date: Date,
});

// Modell aus Schema erstellen
const SomeModel = mongoose.model("SomeModel", SomeModelSchema);
```

Das erste Argument ist der singuläre Name der für Ihr Modell erstellten Sammlung (Mongoose erstellt die Datenbanksammlung für das oben genannte Modell _SomeModel_), und das zweite Argument ist das Schema, das Sie bei der Erstellung des Modells verwenden möchten.

> **Hinweis:** Sobald Sie Ihre Modellklassen definiert haben, können Sie sie verwenden, um Datensätze zu erstellen, zu aktualisieren oder zu löschen und Abfragen auszuführen, um alle Datensätze oder bestimmte Teilmengen von Datensätzen zu erhalten. Wir zeigen Ihnen, wie Sie dies im Abschnitt [Verwendung von Modellen](#using_models) tun können, und wenn wir unsere Ansichten erstellen.

#### Schema-Typen (Felder)

Ein Schema kann eine beliebige Anzahl von Feldern haben - jedes repräsentiert ein Feld in den in MongoDB gespeicherten Dokumenten.
Ein Beispiel für ein Schema, das viele der gängigen Feldtypen und deren Deklaration zeigt, wird unten gezeigt.

```js
const schema = new Schema({
  name: String,
  binary: Buffer,
  living: Boolean,
  updated: { type: Date, default: Date.now() },
  age: { type: Number, min: 18, max: 65, required: true },
  mixed: Schema.Types.Mixed,
  _someId: Schema.Types.ObjectId,
  array: [],
  ofString: [String], // Sie können auch ein Array von jedem anderen Typ haben.
  nested: { stuff: { type: String, lowercase: true, trim: true } },
});
```

Die meisten [Schema-Typen](https://mongoosejs.com/docs/schematypes.html) (die Deskriptoren nach "type:" oder nach Feldnamen) sind selbsterklärend. Die Ausnahmen sind:

- `ObjectId`: Repräsentiert bestimmte Instanzen eines Modells in der Datenbank. Zum Beispiel könnte ein Buch dies verwenden, um sein Autor-Objekt darzustellen. Dies enthält tatsächlich die eindeutige ID (`_id`) für das angegebene Objekt. Wir können die Methode `populate()` verwenden, um die z

ugehörigen Informationen bei Bedarf abzurufen.
- [`Mixed`](https://mongoosejs.com/docs/schematypes.html#mixed): Ein beliebiger Schema-Typ.
- `[]`: Ein Array von Elementen. Sie können JavaScript-Arrayoperationen auf diesen Modellen ausführen (push, pop, unshift usw.). Die obigen Beispiele zeigen ein Array von Objekten ohne einen angegebenen Typ und ein Array von `String`-Objekten, aber Sie können ein Array von Objekten beliebigen Typs haben.

Der Code zeigt auch beide Möglichkeiten, ein Feld zu deklarieren:

- Feld _name_ und _type_ als Schlüssel-Wert-Paar (wie bei den Feldern `name`, `binary` und `living`).
- Feld _name_ gefolgt von einem Objekt, das den `type` und andere _Optionen_ für das Feld definiert. Zu den Optionen gehören Dinge wie:

  - Standardwerte.
  - Eingebaute Validatoren (z. B. max/min-Werte) und benutzerdefinierte Validierungsfunktionen.
  - Ob das Feld erforderlich ist.
  - Ob `String`-Felder automatisch in Kleinbuchstaben, Großbuchstaben oder beschnitten (z. B. `{ type: String, lowercase: true, trim: true }`) gesetzt werden sollen.

Weitere Informationen zu den Optionen finden Sie in der Dokumentation zu [Schema-Typen](https://mongoosejs.com/docs/schematypes.html) (Mongoose-Dokumentation).

#### Validierung

Mongoose bietet integrierte und benutzerdefinierte Validatoren sowie synchrone und asynchrone Validatoren. Sie können sowohl den akzeptablen Wertebereich als auch die Fehlermeldung für eine Validierungsfehler in allen Fällen angeben.

Die integrierten Validatoren umfassen:

- Alle [Schema-Typen](https://mongoosejs.com/docs/schematypes.html) haben den integrierten [required](https://mongoosejs.com/docs/api.html#schematype_SchemaType-required)-Validator. Dies wird verwendet, um anzugeben, ob das Feld angegeben werden muss, um ein Dokument zu speichern.
- [Numbers](https://mongoosejs.com/docs/api.html#schema-number-js) haben [min](https://mongoosejs.com/docs/api.html#schema_number_SchemaNumber-min)- und [max](https://mongoosejs.com/docs/api.html#schema_number_SchemaNumber-max)-Validatoren.
- [Strings](https://mongoosejs.com/docs/api.html#schema-string-js) haben:

  - [enum](https://mongoosejs.com/docs/api.html#schema_string_SchemaString-enum): gibt den Satz zulässiger Werte für das Feld an.
  - [match](https://mongoosejs.com/docs/api.html#schema_string_SchemaString-match): gibt einen regulären Ausdruck an, dem der String entsprechen muss.
  - [maxLength](https://mongoosejs.com/docs/api.html#schema_string_SchemaString-maxlength)- und [minLength](https://mongoosejs.com/docs/api.html#schema_string_SchemaString-minlength)-Validatoren für den String.

Das folgende Beispiel (leicht modifiziert aus den Mongoose-Dokumenten) zeigt, wie Sie einige der Validatortypen und Fehlermeldungen angeben können:

```js
const breakfastSchema = new Schema({
  eggs: {
    type: Number,
    min: [6, "

Zu wenige Eier"],
    max: 12,
    required: [true, "Warum keine Eier?"],
  },
  drink: {
    type: String,
    enum: ["Kaffee", "Tee", "Wasser"],
  },
});
```

Für ausführliche Informationen zur Feldvalidierung siehe [Validation](https://mongoosejs.com/docs/validation.html) (Mongoose-Dokumentation).

#### Virtuelle Eigenschaften

Virtuelle Eigenschaften sind Dokumenteigenschaften, auf die Sie zugreifen und diese setzen können, die aber nicht in MongoDB gespeichert werden. Die Getter sind nützlich zum Formatieren oder Kombinieren von Feldern, während die Setter nützlich sind, um einen einzelnen Wert in mehrere Werte für die Speicherung aufzuteilen. Das Beispiel in der Dokumentation erstellt (und zerlegt) eine virtuelle Eigenschaft mit vollem Namen aus einem Vor- und Nachnamen-Feld, was einfacher und sauberer ist als die Konstruktion eines vollen Namens jedes Mal, wenn er in einer Vorlage verwendet wird.

> **Hinweis:** In der Bibliothek verwenden wir eine virtuelle Eigenschaft, um eine eindeutige URL für jeden Modell-Datensatz zu definieren, indem wir einen Pfad und den Wert der `_id` des Datensatzes verwenden.

Weitere Informationen finden Sie unter [Virtuals](https://mongoosejs.com/docs/guide.html#virtuals) (Mongoose-Dokumentation).

#### Methoden und Abfragehilfsprogramme

Ein Schema kann auch [Instanzmethoden](https://mongoosejs.com/docs/guide.html#methods), [statische Methoden](https://mongoosejs.com/docs/guide.html#statics) und [Abfragehilfsprogramme](https://mongoosejs.com/docs/guide.html#query-helpers) haben. Die Instanz- und statischen Methoden sind ähnlich, aber mit dem offensichtlichen Unterschied, dass eine Instanzmethode mit einem bestimmten Datensatz verknüpft ist und Zugriff auf das aktuelle Objekt hat. Abfragehilfsprogramme ermöglichen es Ihnen, die kettenbare Abfragebaustein-API von Mongoose zu erweitern (z. B. um eine Abfrage "byName" hinzuzufügen, zusätzlich zu den Methoden `find()`, `findOne()` und `findById()`).

### Verwendung von Modellen

Sobald Sie ein Schema erstellt haben, können Sie es verwenden, um Modelle zu erstellen. Das Modell repräsentiert eine Sammlung von Dokumenten in der Datenbank, die Sie durchsuchen können, während die Modellinstanzen einzelne Dokumente repräsentieren, die Sie speichern und abrufen können.

Im Folgenden geben wir einen kurzen Überblick. Weitere Informationen finden Sie unter: [Models](https://mongoosejs.com/docs/models.html) (Mongoose-Dokumentation).

> **Hinweis:** Das Erstellen, Aktualisieren, Löschen und Abfragen von Datensätzen sind asynchrone Vorgänge, die ein [Promise](/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) zurückgeben.
> Die unten gezeigten Beispiele zeigen lediglich die Verwendung der relevanten Methoden und von `await` (d. h. den wesentlichen Code für die Verwendung der Methoden).
> Die umgebende `async function` und der `try...catch

`-Block zum Abfangen von Fehlern werden aus Gründen der Klarheit weggelassen.
> Weitere Informationen zur Verwendung von `await/async` finden Sie im Abschnitt [Datenbank-APIs sind asynchron](#database_apis_are_asynchronous) oben.

#### Erstellen und Ändern von Dokumenten

Um einen Datensatz zu erstellen, können Sie eine Instanz des Modells definieren und dann `save()` darauf aufrufen.
Die folgenden Beispiele gehen davon aus, dass `SomeModel` ein Modell ist (mit einem einzelnen Feld `name`), das wir aus unserem Schema erstellt haben.

```js
// Eine Instanz des Modells SomeModel erstellen
const awesome_instance = new SomeModel({ name: "awesome" });

// Das neue Modell-Objekt asynchron speichern
await awesome_instance.save();
```

Sie können auch `create()` verwenden, um die Modellinstanz gleichzeitig mit dem Speichern zu definieren.
Im folgenden Beispiel erstellen wir nur eine Instanz, aber Sie können auch mehrere Instanzen erstellen, indem Sie ein Array von Objekten übergeben.

```js
await SomeModel.create({ name: "also_awesome" });
```

Jedes Modell hat eine zugehörige Verbindung (das wird die Standardverbindung sein, wenn Sie `mongoose.model()` verwenden). Sie können eine neue Verbindung erstellen und `.model()` darauf aufrufen, um die Dokumente in einer anderen Datenbank zu erstellen.

Sie können auf die Felder in diesem neuen Datensatz mit der Punkt-Syntax zugreifen und die Werte ändern. Sie müssen `save()` oder `update()` aufrufen, um geänderte Werte zurück in die Datenbank zu speichern.

```js
// Auf Model-Feldwerte mit der Punkt-Notation zugreifen
console.log(awesome_instance.name); // sollte 'also_awesome' protokollieren

// Datensatz ändern, indem Sie die Felder ändern und dann save() aufrufen.
awesome_instance.name = "Neuer cooler Name";
await awesome_instance.save();
```

#### Suche nach Datensätzen

Sie können nach Datensätzen suchen, indem Sie Abfragemethoden verwenden und die Abfragebedingungen als JSON-Dokument angeben. Der folgende Codeausschnitt zeigt, wie Sie alle Athleten in einer Datenbank finden können, die Tennis spielen, und nur die Felder für den Athleten _name_ und _age_ zurückgeben. Hier geben wir nur ein übereinstimmendes Feld (Sport) an, aber Sie können weitere Kriterien hinzufügen, reguläre Ausdruckskriterien angeben oder die Bedingungen ganz entfernen, um alle Athleten zurückzugeben.

```js
const Athlete = mongoose.model("Athlete", yourSchema);

// Alle Athleten finden, die Tennis spielen, und die Felder 'name' und 'age' auswählen
const tennisPlayers = await Athlete.find(
  { sport: "Tennis" },
  "name age"
).exec();
```
