Dieser zweite Artikel in unserem Express Tutorial zeigt, wie Sie ein "Grundgerüst" für ein Website-Projekt erstellen können, das Sie dann mit spezifischen Routen, Templates/Views und Datenbankaufrufen füllen können.

<table>
  <tbody>
    <tr>
      <th scope="row">Voraussetzungen:</th>
      <td>
        <a href="/de/docs/Learn/Server-seitig/Express_Nodejs/development_environment">Richten Sie eine Node-Entwicklungsumgebung ein</a>.
          Lesen Sie das Express Tutorial.
      </td>
    </tr>
    <tr>
      <th scope="row">Ziel:</th>
      <td>
        Die Fähigkeit erlangen, eigene neue Website-Projekte mit dem <em>Express Application Generator</em> zu starten.
      </td>
    </tr>
  </tbody>
</table>

## Überblick

Dieser Artikel zeigt, wie Sie mithilfe des [Express Application Generator](https://expressjs.com/en/starter/generator.html)-Tools eine "Grundstruktur" für eine Website erstellen können, die Sie dann mit spezifischen Routen, Views/Templates und Datenbankaufrufen füllen können. In diesem Fall verwenden wir das Tool, um das Grundgerüst für unsere [Local Library-Website](/de/docs/Learn/Server-seitig/Express_Nodejs/Tutorial_local_library_website) zu erstellen, zu der wir später den restlichen für die Website erforderlichen Code hinzufügen werden. Der Prozess ist äußerst einfach und erfordert lediglich den Aufruf des Generators in der Befehlszeile mit einem neuen Projektnamen, optional auch der Angabe des Template-Engines und CSS-Generators für die Website.

In den folgenden Abschnitten zeigen wir Ihnen, wie Sie den Application Generator aufrufen können, und geben eine kurze Erklärung zu den verschiedenen Ansichten/CSS-Optionen. Außerdem erklären wir, wie die Grundstruktur der Website aufgebaut ist. Am Ende zeigen wir Ihnen, wie Sie die Website ausführen können, um zu überprüfen, ob sie funktioniert.

> **Hinweis:**
>
> - Der _Express Application Generator_ ist nicht der einzige Generator für Express-Anwendungen, und das generierte Projekt ist nicht die einzige mögliche Art und Weise, Ihre Dateien und Verzeichnisse zu strukturieren. Die generierte Website hat jedoch eine modulare Struktur, die einfach erweitert und verstanden werden kann. Weitere Informationen zu einer _minimalen_ Express-Anwendung finden Sie unter [Hello world-Beispiel](https://expressjs.com/en/starter/hello-world.html) (Express-Dokumentation).
> - Der _Express Application Generator_ deklariert die meisten Variablen mit `var`.
>   Wir haben die meisten davon in der Anleitung in `const`-Variablen (und einige

 in `let`-Variablen) geändert, um moderne JavaScript-Praktiken zu demonstrieren.
> - Diese Anleitung verwendet die in der durch den _Express Application Generator_ erstellten **package.json** definierten Version von _Express_ und anderen Abhängigkeiten.
>   Diese sind nicht unbedingt die neueste Version, und Sie möchten sie möglicherweise aktualisieren, wenn Sie eine echte Anwendung in Produktion bereitstellen.

## Verwenden des Application Generators

Sie sollten den Generator bereits im Rahmen der [Einrichtung einer Node-Entwicklungsumgebung](/de/docs/Learn/Server-seitig/Express_Nodejs/development_environment) installiert haben. Als kurze Erinnerung: Sie installieren das Generator-Tool systemweit mit dem npm-Paketmanager, wie im folgenden Beispiel gezeigt:

```bash
npm install express-generator -g
```

Der Generator verfügt über eine Reihe von Optionen, die Sie auf der Befehlszeile mit dem Befehl `--help` (oder `-h`) anzeigen können:

```bash
> express --help

    Verwendung: express [Optionen] [Verzeichnis]

  Optionen:

        --version        Ausgabe der Versionsnummer
    -e, --ejs            EJS-Engine-Unterstützung hinzufügen
        --pug            Pug-Engine-Unterstützung hinzufügen
        --hbs            Handlebars-Engine-Unterstützung hinzufügen
    -H, --hogan          Hogan.js-Engine-Unterstützung hinzufügen
    -v, --view <Engine>  Ansichts-<Engine>-Unterstützung hinzufügen (dust|ejs|hbs|hjs|jade|pug|twig|vash) (Standard: jade)
        --no-view        Statische HTML- statt Ansichts-Engine verwenden
    -c, --css <Engine>   Stylesheet-<Engine>-Unterstützung hinzufügen (less|stylus|compass|sass) (Standard: reines CSS)
        --git            .gitignore hinzufügen
    -f, --force          Erzwingen in einem nicht leeren Verzeichnis
    -h, --help           Anzeige der Hilfeseite
```

Sie können Express angeben, um ein Projekt im aktuellen Verzeichnis mit der _Jade_-Ansichts-Engine und reinem CSS zu erstellen (wenn Sie einen Verzeichnisnamen angeben, wird das Projekt in einem Unterordner mit diesem Namen erstellt).

```bash
express
```

Sie können auch eine Ansichts- (Template-)Engine mit `--view` und/oder eine CSS-Generierungs-Engine mit `--css` auswählen.

> **Hinweis:** Die anderen Optionen zum Auswählen von Template-Engines (z. B. `--hogan`, `--ejs`, `--hbs` usw.) sind veraltet. Verwenden Sie stattdessen `--view` (oder `-v`).

### Welche Ansichts-Engine sollte ich verwenden?

Der _Express Application Generator_ ermöglicht es Ihnen, verschiedene gängige Ansichts-/Template-Engines zu konfigurieren, darunter [EJS](https://www.npmjs.com/package/ejs), [Hbs](https://github.com/pillarjs/hbs), [Pug](https://pugjs.org/api/getting-start

ed.html) (Jade), [Twig](https://www.npmjs.com/package/twig) und [Vash](https://www.npmjs.com/package/vash). Standardmäßig verwendet der Generator Jade, wenn Sie keine Ansichtsoption angeben. Express selbst unterstützt auch eine große Anzahl anderer Template-Sprachen [out of the box](https://github.com/expressjs/express/wiki#template-engines).

> **Hinweis:** Wenn Sie eine Template-Engine verwenden möchten, die vom Generator nicht unterstützt wird, sehen Sie sich [Verwendung von Template-Engines mit Express](https://expressjs.com/de/guide/using-template-engines.html) (Express-Dokumentation) und die Dokumentation für Ihre Ziel-Template-Engine an.

Im Allgemeinen sollten Sie eine Template-Engine auswählen, die Ihnen alle benötigte Funktionalität bietet und es Ihnen ermöglicht, schnell produktiv zu sein - oder mit anderen Worten, so wie Sie jede andere Komponente auswählen würden! Bei der Auswahl einer Template-Engine sollten Sie folgende Punkte berücksichtigen:

- Zeitaufwand bis zur Produktivität: Wenn Ihr Team bereits Erfahrung mit einer bestimmten Template-Sprache hat, werden sie wahrscheinlich schneller produktiv sein. Wenn nicht, sollten Sie den relativen Lernaufwand für die verschiedenen Template-Engines berücksichtigen.
- Beliebtheit und Aktivität: Überprüfen Sie die Beliebtheit der Engine und ob es eine aktive Community gibt. Es ist wichtig, Unterstützung zu erhalten, wenn Probleme während der gesamten Lebensdauer der Website auftreten.
- Stil: Einige Template-Engines verwenden spezifische Markierungen, um eingefügten Inhalt in "gewöhnlichem" HTML zu kennzeichnen, während andere das HTML mithilfe einer anderen Syntax aufbauen (z. B. durch Einrückungen und Blocknamen).
- Leistung/Rendering-Zeit.
- Funktionen: Sie sollten überlegen, ob die von Ihnen betrachteten Engines die folgenden Funktionen bieten:

  - Layout-Vererbung: Ermöglicht es Ihnen, eine Basisschablone zu definieren und dann nur die Teile davon "zu erben", die Sie für eine bestimmte Seite ändern möchten. Dies ist in der Regel besser, als Schablonen zu erstellen, indem Sie eine Reihe von erforderlichen Komponenten einschließen oder eine Schablone jedes Mal von Grund auf neu erstellen.
  - "Include"-Unterstützung: Ermöglicht es Ihnen, Schablonen aufzubauen, indem Sie andere Schablonen einfügen.
  - Kurze Variablen- und Schleifensteuerungssyntax.
  - Möglichkeit zur Filterung von Variablenwerten auf der Template-Ebene, z. B. durch Umwandlung von Variablen in Großbuchstaben oder Formatierung eines Datums.
  - Möglichkeit zur Generierung von Ausgabeformaten, die sich von HTML unterscheiden, wie JSON oder XML.
  - Unterstützung für asynchrone Vorgänge und Streaming.
  - Clientseitige Funktionen. Wenn eine Template-Engine auf dem Client verwendet werden kann, besteht die Möglichkeit, dass ein Großteil des Renderns clientseitig erfolgt.

> **Hinweis:** Im Internet finden Sie viele Ressourcen, die Ihnen helfen, die verschiedenen Optionen zu vergleichen

 und zu entscheiden, welche Engine für Ihre Bedürfnisse am besten geeignet ist. Einige Ressourcen sind [Express-Engine-Vergleich](https://github.com/expressjs/express/wiki#template-engines) (Express-Wiki), [Template Engine Benchmarks](https://github.com/leizongmin/template-benchmark) (GitHub) und [JavaScript Template Engine Performance](https://garann.github.io/template-chooser/) (garann.me).

### Beispiel

Um ein neues Express-Projekt mit dem Namen `local-library` zu erstellen, das die EJS-Template-Engine und CSS verwendet, geben Sie den folgenden Befehl ein:

```bash
express --view=ejs --css=css local-library
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
