# Angular i18n

## Installation

`ng add @angular/localize`

## Text zur Übersetzung markieren

```html
<!-- HTML Elemente -->
<span i18n>Hello world!</span>

<!-- Ohne HTML Element -->
<ng-container i18n>Hello world!</ng-container>

<!-- HTML Properties -->
<img src="..." i18n-title title="Logo">Hello world! />
```

### Beschreibungen und eindeutige IDs

Um dem Übersetzer mehr Informationen zum Kontext des zu übersetzenden Texts zu geben, können zusätzliche Beschreibungen hinzugefügt werden.

```html
<span i18n="Meaning|Description@@ID">Hello world!</span>
```

Diese Informationen werden beim Extrahieren hinzugefügt. Wird eine ID angegeben, wird keine ID von Angular beim Extrahieren generiert. Dabei muss man darauf achten, dass sich die IDs nicht bei verschiedenen Texten überschneiden.

## Übersetzung bei Pluralization und Selection

```ts
export enum Colors {
    Blue = 'BLUE',
    Red = 'RED'
}

@Component()
export class AppComponent {
    minutes = Math.floor(Math.random() * 10);
    color = Math.random() > 0.5 ? Colors.Blue : Colors.Red;
}
```

```html
<!-- Pluralization -->
<span i18n>Aktualisiert vor
{ minutes, plural,
    =0 {Kurzem}
    =1 {einer Minute}
    other {<b>{{minutes}}</b> Minuten}
}
</span>

<!-- Selection -->
<span i18n>Die Farbe ist: 
{ color, select,
    BLUE {Blau}
    RED {Rot}
}
</span>
```

Im ausgegebenen Text kann man sowohl HTML als auch Interpolation mithilfe von `{{ ... }}` nutzen.

Das Format von `plural` und `select` entspricht dem [Standard der ICU](http://userguide.icu-project.org/formatparse/messages)

## Extrahieren und übersetzen der Texte

Ausführen: `ng xi18n`

Es wird eine `messages.xlf` generiert, die alle zu übersetzenden Texte enthält.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
  <file source-language="en" datatype="plaintext" original="ng2.template">
    <body>
      <trans-unit id="b6c7be1cfe7c9b8b946383c40c5b7f0e3d0c73d4" datatype="html">
        <source>Hello world!</source>
        <context-group purpose="location">
          <context context-type="sourcefile">src/app/app.component.html</context>
          <context context-type="linenumber">2</context>
        </context-group>
        <note priority="1" from="description">Description</note>
        <note priority="1" from="meaning">Meaning</note>
      </trans-unit>
    </body>
  </file>
</xliff>
```

Zur Übersetzung kopiert man diese Datei und benennt sie in z.B. `messages.de.xlf` um. Anschließend übersetzt man die Texte.

```xml
<trans-unit id="b6c7be1cfe7c9b8b946383c40c5b7f0e3d0c73d4" datatype="html">
    <source>Hello world!</source>
    <target>Hallo Welt!</target>
</trans-unit>
```

Zu dem `<source>` kommt ein `<target>` hinzu, welches den übersetzten Text in der Zielsprache enthält.

## Mehrsprachiger Build

In der `angular.json` der App eine `i18n` Konfiguration hinzufügen:

```json
{
    "projects": {
        "app": {
            "i18n": {
                "sourceLocale": "en", 
                "locales": {
                	"de": "locale/messages.de.xlf" 
                }
            }
        }
    }
}

```

`sourceLocale` ist die Sprache, die während der Entwicklung im Template genutzt wird.
`locales` sind die Zielsprachen und ihre Übersetzungsdateien. Die Bezeichnet der Zielsprachen werden später im Build als Ordner für die jeweilige Sprache genutzt.

Um einen mehrsprachigen Build für alle angegebenen Sprachen durchzuführen, folgenden Befehl ausführen:

```sh
ng build --localize
```

## ng serve mit anderer Sprache

Um beim `ng serve` eine andere Sprache zu nutzen, als die angegebene `sourceLocale` wird die `angular.json` angepasst.

```json
{
    "build": {
        "configurations": {
            "de": {
                "localize": ["de"]
            }
        }
    },
    "serve": {
        "configurations": {
            "de": {
                "browserTarget": "app:build:de"
            }
        }
    }
}
```

Danach kann man folgenden Befehl nutzen:

    ng serve --configuration=de

## XLF Dateien automatisch mergen

Um die XLF Dateien automisch zu mergen, wenn neue Übersetzungstexte hinzu kommen, ohne die vorhanden zu ändern, wird das Tool `xliffmerge` verwendet.

Zunächst die Dependency installieren:

    npm i ngx-i18nsupport --save-dev

Das Extrahieren und mergen der XLF Dateien kann anschließend in einem Schritt ausgeführt werden:

    ng xi18n --output-path locale && xliffmerge --profile xliffmerge.json en de

`xliffmerge.json`

```json
{
  "xliffmergeOptions": {
    "srcDir": "locale",
    "genDir": "locale"
  }
}
```

Das bewirkt, dass neue Übersetzungen in der XLF Datei mit dem Status `new` gekennzeichnet werden:

```xml
<source>Hello world!</source>
<target state="new">Hello world!</target>
```

Vorhandene Übersetzungen, die sich nicht verändert haben, werden mit dem Status `final` gekennzeichnet.

```xml
<source>Hello world!</source>
<target state="final">Hallo Welt!</target>
```

So lässt sich erkennen, welche Texte noch nicht und welche bereits übersetzt wurden.

## Text im Code zur Übersetzung markieren (HACK)

Standardmäßig können nur Texte im Template zur Übersetzung markiert werden. In Angular 9 lässt sich dies aber auch im Code tun. 

Es existiert eine globale `$localize` Funktion, die genutzt werden kann, um zu übersetzenden Text zu markieren.

```ts
@Component({
  template: '{{ text }}'
})
export class HomeComponent {
  text = $localize`Hello world!`;
}
```

Es handelt sich um einen Tagged Template String, bei dem die Template-String Methode `$localize` verwendet wird.

Texte, die auf diese Methode übersetzt werden sollen, werden **NICHT** automatisiert von Angular per `ng xi18n` gefunden.

Erst beim Build gibt der Compiler eine Warnung aus (oder einen Fehler, wenn in der `angular.json` die Option `i18nMissingTranslation` auf `error` gesetzt wird), dass ein übersetzter Text fehlt:

    No translation found for "2463265708606399074" ("Translate me").

Mit dieser ID muss dann **manuell** ein Eintrag in die Übersetzungdatei `messages.xlf` hinzugefügt werden.

```xml
<trans-unit id="2463265708606399074">
    <source>Translate me</source>
</trans-unit>
```

Anschließend wird mithilfe des `xliffmerge` Tools dieser Eintrag in die übrigen Übersetzungsdateien überführt:

    xliffmerge --profile xliffmerge.json en de

Dort wird er dann mit dem `state="new"` gekennzeichnet.

### Variablen im Template String

Wenn Variablen im Template String verwendet werden, sieht die Fehlermeldung etwas anders aus:

    No translation found for "3671200097505039568" ("Translate me {$PH}").

Um diese Variablen in der Übersetzungsdatei korrekt zu übernehmen, wird ein `<x>` Element mit der angezeigten ID `PH` an der Stelle hinzugefügt.

```xml
<trans-unit id="2463265708606399074">
    <source>Translate me <x id="PH"/></source>
</trans-unit>
```

### Beschreibungen und eindeutige IDs

Auch bei der Verwendung der `$localize` Funktion können Beschreibungen und IDs verwendet werden:

```ts
const text = $localize`:Description@@ID:Translate me ${this.value}:placeholderId:`
```

Das würde zu folgender Warnung beim Build führen:

    No translation found for "ID" ("Translate me {$placeholderId}" - "Short Description").

Mithilfe dieser Informationen kann dann ein Eintrag in die Übersetzungsdatei erfolgen:

```xml
<trans-unit id="ID">
    <source>Translate me <x id="placeholderId" /></source>
    <note priority="1" from="description">Short Description</note>
</trans-unit>
```
