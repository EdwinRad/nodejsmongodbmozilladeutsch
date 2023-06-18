## Überblick

Dieser Artikel zeigt, wie Sie mithilfe des [Express Application Generator](https://expressjs.com/en/starter/generator.html)-Tools eine "Grundstruktur" für eine Website erstellen können, die Sie dann mit spezifischen Routen, Views/Templates und Datenbankaufrufen füllen können. In diesem Fall verwenden wir das Tool, um das Grundgerüst für unsere [Local Library-Website](/de/docs/Learn/Server-seitig/Express_Nodejs/Tutorial_local_library_website) zu erstellen, zu der wir später den restlichen für die Website erforderlichen Code hinzufügen werden. Der Prozess ist äußerst einfach und erfordert lediglich den Aufruf des Generators in der Befehlszeile mit einem neuen Projektnamen, optional auch der Angabe des Template-Engines und CSS-Generators für die Website.

In den folgenden Abschnitten zeigen wir Ihnen, wie Sie den Application Generator aufrufen können, und geben eine kurze Erklärung zu den verschiedenen Ansichten/CSS-Optionen. Außerdem erklären wir, wie die Grundstruktur der Website aufgebaut ist. Am Ende zeigen wir Ihnen, wie Sie die Website ausführen können, um zu überprüfen, ob sie funktioniert.


## Verwenden des Application Generators

Sie installieren das Generator-Tool systemweit mit dem npm-Paketmanager, wie im folgenden Beispiel gezeigt:

```bash
npm install express-generator -g
```

### Beispiel

Um ein neues Express-Projekt mit dem Namen `local-library` zu erstellen, das die pugs-Template-Engine und CSS verwendet, geben Sie den folgenden Befehl ein:

```bash
express --view=pugs --css=css local-library
```

Nachdem der Generator erfolgreich abgeschlossen ist, sollten Sie eine ähnliche Ausgabe sehen:

```bash
   create : local-library/
   create : local-library/public/
   create : local-library/public/javascripts/
   create : local-library/public/images/
   create : local-library/public/stylesheets/
   create : local-library/public/stylesheets/style.css
   create : local-library/routes/
   create : local-library/routes/index.js
   create : local-library/routes/users.js
   create : local-library/views/
   create : local-library/views/error.ejs
   create : local-library/views/index.ejs
   create : local-library/views/layout.ejs
   create : local-library/app.js
   create : local-library/package.json
   create : local-library/bin/
   create : local-library/bin/www

   change directory:
     $ cd local-library

   install dependencies:
     $ npm install

   run the app:
     $ DEBUG=local-library:* npm start
```

In diesem Beispiel wird ein neues Verzeichnis namens `local-library` erstellt, das das grundlegende Skelett einer Express-Anwendung enthält.

## Struktur der generierten Website

Wenn Sie in das neu erstellte Verzeichnis `local-library` wechseln, finden Sie eine Reihe von Dateien und Verzeichnissen. Hier ist eine Übersicht über die Struktur:

- **`app.js`**: Die zentrale Datei der Anwendung, die den Express-Server konfiguriert und startet.
- **`bin/www`**: Dieses Skript wird verwendet, um den Server zu starten. Es erstellt einen HTTP-Server auf dem angegebenen Port (oder dem Standardport 3000) und stellt den Express-Server bereit.
- **`package.json`**: Die Datei `package.json` enthält Metadaten über die Anwendung und die Abhängigkeiten, die von der Anwendung verwendet werden.
- **`public/`**: Dieses Verzeichnis enthält statische Ressourcen wie Bilder, Stylesheets und Client-seitige JavaScript-Dateien.
- **`routes/`**: Dieses Verzeichnis enthält Moduldateien, die für die Routen der Anwendung zuständig sind.
- **`views/`**: Dieses Verzeichnis enthält die Ansichtsvorlagen (Templates) der Anwendung. In diesem Fall enthält es auch eine Teilvorlage (`layout.ejs`), die von anderen Vorlagen erweitert werden kann.

### Ressourcen

Die Anwendung generiert auch einige grundlegende Ressourcen für Sie:

- `public/stylesheets/style.css`: Eine CSS-Datei, in der Sie Ihre eigenen Styles definieren können.
- `routes/index.js` und `routes/users.js`: Diese Dateien enthalten den grundlegenden Code für die Routen der Anwendung

. Sie zeigen, wie Sie Routen definieren und Funktionen zum Bearbeiten von Anfragen erstellen können.
- `views/index.ejs` und `views/error.ejs`: Diese Vorlagen werden verwendet, um die Startseite und Fehlerseiten der Anwendung anzuzeigen. Sie können diese Vorlagen an Ihre Bedürfnisse anpassen.

## Installieren der Abhängigkeiten

Nachdem das Projekt generiert wurde, navigieren Sie in das Projektverzeichnis (`local-library`) und installieren die Abhängigkeiten mit dem Befehl `npm install`. Dieser Befehl liest die in der `package.json`-Datei definierten Abhängigkeiten und installiert sie im `node_modules`-Verzeichnis.

```bash
cd local-library
npm install
```

## Starten der Anwendung

Nachdem die Abhängigkeiten installiert wurden, können Sie die Anwendung mit dem Befehl `npm start` starten:

```bash
npm start
```

Dieser Befehl startet den Express-Server und gibt die folgende Ausgabe aus:

```bash
> local-library@0.0.0 start /path/to/local-library
> node ./bin/www

  local-library:server Listening on port 3000 +0ms
```

Sie können die Anwendung nun im Browser öffnen, indem Sie `http://localhost:3000` eingeben.

> **Hinweis:** Standardmäßig startet die Anwendung auf Port 3000. Sie können den Port ändern, indem Sie die Umgebungsvariable `PORT` setzen, z.B. `PORT=5000 npm start`.

Wenn alles erfolgreich ist, sollten Sie die Startseite der Anwendung sehen.

## Zusammenfassung

In diesem Tutorial haben wir den Express Application Generator verwendet, um eine neue Express-Anwendung zu generieren. Wir haben die verschiedenen Optionen des Generators untersucht und gezeigt, wie Sie eine bestimmte Ansichts-Engine und CSS-Engine auswählen können. Nachdem das Projekt generiert wurde, haben wir die grundlegende Struktur der generierten Website überprüft und gezeigt, wie Sie die Anwendung starten können.

Der Express Application Generator ist ein nützliches Tool, um schnell ein grundlegendes Skelett für Ihre Express-Anwendung zu generieren und Sie auf den Weg zu bringen. Von dort aus können Sie die generierte Struktur an Ihre spezifischen Anforderungen anpassen und Ihre Anwendung weiterentwickeln.

Wir hoffen, dass dieses Tutorial Ihnen einen guten Einstieg in den Express Application Generator gegeben hat und Ihnen dabei hilft, Ihre Express-Projekte effizienter zu starten!
