## Überblick

Das Personal der Bibliothek wird die Website Local Library verwenden, um Informationen über Bücher und Ausleiher zu speichern, während die Bibliotheksbenutzer sie nutzen können, um nach Büchern zu suchen und zu stöbern, herauszufinden, ob Exemplare verfügbar sind, und diese dann zu reservieren oder auszuleihen. Um Informationen effizient zu speichern und abzurufen, werden wir sie in einer _Datenbank_ speichern.

Express-Apps können viele verschiedene Datenbanken verwenden, und es gibt mehrere Ansätze für die Durchführung von **C**reate, **R**ead, **U**pdate und **D**elete (CRUD)-Operationen. Dieses Tutorial gibt einen kurzen Überblick über einige der verfügbaren Optionen und zeigt dann im Detail die ausgewählten Mechanismen.

### Verwendung von Mongoose und MongoDB für die Local Library

Für das Beispiel der _Local Library_ (und den Rest dieses Themas) verwenden wir das [Mongoose ODM](https://www.npmjs.com/package/mongoose), um auf unsere Bibliotheksdaten zuzugreifen. Mongoose fungiert als Schnittstelle für [MongoDB](https://www.mongodb.com/what-is-mongodb), eine Open-Source-[NoSQL](https://en.wikipedia.org/wiki/NoSQL)-Datenbank, die ein dokumentenorientiertes Datenmodell verwendet. Eine "Sammlung" von "Dokumenten" in einer MongoDB-Datenbank [ist analog zu](https://www.mongodb.com/docs/manual/core/databases-and-collections/) einer "Tabelle" von "Zeilen" in einer relationalen Datenbank.

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


![Mongoose Library Model mit richtiger Kardinalität](assets/library_website_-_mongoose_express.png)


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


-----


## Einrichten der MongoDB-Datenbank

Nun, da wir verstehen, was Mongoose tun kann und wie wir unsere Modelle entwerfen möchten, ist es an der Zeit, mit der Entwicklung der _LocalLibrary_-Website zu beginnen. Das allererste, was wir tun möchten, ist die Einrichtung einer MongoDB-Datenbank, in der wir unsere Bibliotheksdaten speichern können.

In diesem Tutorial verwenden wir die cloud-basierte Sandbox-Datenbank [MongoDB Atlas](https://www.mongodb.com/atlas/database). Diese Datenbankstufe gilt nicht als geeignet für Produktionswebsites, da sie keine Redundanz aufweist, ist jedoch ideal für Entwicklung und Prototyping. Wir verwenden sie hier, weil sie kostenlos und einfach einzurichten ist und weil MongoDB Atlas ein beliebter Anbieter von "Datenbank als Service" ist, den


Zunächst müssen Sie [ein Konto](https://www.mongodb.com/cloud/atlas/register) bei MongoDB Atlas erstellen (dies ist kostenlos und erfordert nur die Eingabe grundlegender Kontaktdaten und die Bestätigung der Nutzungsbedingungen).

Nach dem Einloggen gelangen Sie zur [Startseite](https://cloud.mongodb.com/v2):

1. Klicken Sie auf die Schaltfläche **Eine Datenbank erstellen** im Abschnitt _Database Deployments_.
   ![Eine Datenbank auf MongoDB Atlas erstellen.](assets/mongodb_atlas_-_createdatabase.jpg)

2. Dadurch wird der Bildschirm _Deploy a cloud database_ geöffnet. Klicken Sie unter der Option _Shared_ auf die Schaltfläche **Create**.
   ![Eine Bereitstellungsoption auf MongoDB Atlas auswählen.](assets/mongodb_atlas_-_deploy.jpg)

3. Dadurch wird der Bildschirm _Create a Shared Cluster_ geöffnet.
   ![Wählen Sie einen Cloud-Anbieter auf MongoDB Atlas aus.](assets/mongodb_atlas_-_createsharedcluster.jpg)

   - Wählen Sie einen beliebigen Anbieter aus dem Abschnitt _Cloud Provider & Region_. Verschiedene Regionen bieten unterschiedliche Anbieter an.
   - Die _Cluster-Tier_- und _Additional Settings_-Einstellungen müssen nicht geändert werden. Sie können den Namen Ihres Clusters unter _Cluster Name_ ändern. Für dieses Tutorial nennen wir ihn `Cluster0`.
   - Klicken Sie auf die Schaltfläche **Create Cluster** (das Erstellen des Clusters dauert einige Minuten).

4. Dadurch wird der _Security Quickstart_-Bereich geöffnet.
   ![Richten Sie die Zugriffsregeln auf dem Bildschirm Security Quickstart auf MongoDB Atlas ein.](assets/mongodb_atlas_-_securityquickstart.jpg)

   - Geben Sie einen Benutzernamen und ein Passwort ein. Denken Sie daran, die Anmeldeinformationen zu kopieren und sicher aufzubewahren, da wir sie später benötigen werden. Klicken Sie auf die Schaltfläche **Create User**.

     > **Hinweis:** Verwenden Sie keine Sonderzeichen im MongoDB-Benutzerpasswort, da mongoose die Verbindungszeichenfolge möglicherweise nicht richtig analysiert.

   - Geben Sie `0.0.0.0/0` im IP-Adressfeld ein. Dies gibt

 MongoDB an, dass wir den Zugriff von überall erlauben möchten. Klicken Sie auf die Schaltfläche **Add Entry**.

     > **Hinweis:** Es ist eine bewährte Praxis, die IP-Adressen einzuschränken, die auf Ihre Datenbank und andere Ressourcen zugreifen können. Hier erlauben wir den Zugriff von überall, da wir nicht wissen, von welcher Stelle aus die Anfrage nach der Bereitstellung erfolgen wird.

   - Klicken Sie auf die Schaltfläche **Finish and Close**.

5. Dadurch gelangen Sie zu folgendem Bildschirm. Klicken Sie auf die Schaltfläche **Go to Databases**.
   ![Nach der Einrichtung der Zugriffsregeln zu Datenbanken auf MongoDB Atlas gehen.](assets/mongodb_atlas_-_accessrules.jpg)

6. Sie kehren zum Bildschirm _Database Deployments_ zurück. Klicken Sie auf die Schaltfläche **Browse Collections**.
   ![Eine Sammlung auf MongoDB Atlas einrichten.](assets/mongodb_atlas_-_createcollection.jpg)

7. Dadurch wird der Abschnitt _Collections_ geöffnet. Klicken Sie auf die Schaltfläche **Add My Own Data**.
   ![Eine Datenbank auf MongoDB Atlas erstellen.](assets/mongodb_atlas_-_adddata.jpg)

8. Dadurch wird der Bildschirm _Create Database_ geöffnet.

   ![Details während der Erstellung einer Datenbank auf MongoDB Atlas.](assets/mongodb_atlas_-_databasedetails.jpg)

   - Geben Sie den Namen für die neue Datenbank als `local_library` ein.
   - Geben Sie den Namen der Sammlung als `Collection0` ein.
   - Klicken Sie auf die Schaltfläche **Create**, um die Datenbank zu erstellen.

9. Sie kehren zum Bildschirm _Collections_ zurück, und Ihre Datenbank wurde erstellt.
   ![Bestätigung der Erstellung einer Datenbank auf MongoDB Atlas.](assets/mongodb_atlas_-_databasecreated.jpg)

   - Klicken Sie auf den Tab _Overview_, um zur Clusterübersicht zurückzukehren.

10. Klicken Sie auf der Cluster0-Übersicht auf die Schaltfläche **Connect**.
    ![Verbindung einrichten, nachdem ein Cluster auf MongoDB Atlas erstellt wurde.](assets/mongodb_atlas_-_connectbutton.jpg)

11. Dadurch wird der Bildschirm _Connect to Cluster_ geöffnet.
    Wählen Sie die Option **Connect your application** aus.
    ![Eine Verbindungsmethode auf MongoDB Atlas auswählen.](assets/mongodb_atlas_-_chooseaconnectionmethod.jpg)

12. Sie sehen nun den Bildschirm _Connect_.
    ![Die Short SRV-Verbindung auswählen, wenn eine Verbindung auf MongoDB Atlas eingerichtet wird.](assets/mongodb_atlas_-_connectforshortsrv.jpg)

    - Wählen Sie den Node-Treiber und die Version wie gezeigt aus.
    - Klicken Sie auf das Symbol **Copy**, um die Verbindungszeichenfolge zu kopieren.
    - Fügen Sie dies in Ihren lokalen Texteditor ein.
    - Aktualisieren Sie den Benutzernamen und das Passwort mit dem Passwort Ihres Benutzers.
    - Fügen Sie den Datenbanknamen "local_library" vor den Optionen in den Pfad ein (`...mongodb.net/local_library?retryWrites...`).
    - Speichern Sie die Datei mit dieser Zeichenfolge an einem sicheren Ort.

Sie haben jetzt die Datenbank erstellt und eine URL (mit Benutzernamen und Passwort), mit der darauf zugegriffen werden kann.
Diese sieht ungefähr so aus: `mongodb+srv://your_user_name:your_password@cluster0.lz91hw2.mongodb.net/local_library?retryWrites=true&w=majority`