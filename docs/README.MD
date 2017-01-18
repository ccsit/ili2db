================ ili2db-Anleitung ================

Überblick
=========

Ili2pg bzw. ili2gpkg ist ein in Java erstelltes Programm, das eine Interlis-Transferdatei (itf oder xtf) einem Interlis-Modell entsprechend (ili) mittels 1:1-Transfer in eine Datenbank (PostgreSQL/Postgis bzw. GeoPackage) schreibt oder aus der Datenbank mittels einem 1:1-Transfer eine solche Transferdatei erstellt. Folgende Funktionen sind möglich:

-   1:1-Umwandlung einer Modelldatei in ein Datenbankschema.

-   1:1-Import einer beliebigen Transferdatei mit dazugehöriger Modelldatei in eine Datenbank.

-   1:1-Export von Datenbanktabellen in eine Interlis-Transferdatei.

-   1:1-Export von Datenbanktabellen in eine GML-Transferdatei \[1\]\_.

1:1-Import in die Datenbank
---------------------------

Der 1:1-Import schreibt alle Objekte (im Sinne der eigentlichen Daten) der Interlis-Transferdatei in die Datenbank. Falls die Tabellen in der Datenbank resp. im Schema noch nicht existieren, werden die Tabellen und falls nötig das Schema beim Import angelegt.

Es besteht die Möglichkeit ein Schema mit leeren Tabellen aus dem Interlis-Modell in der Datenbank zu erstellen (nur PostGIS).

Area- und Surface-Geometrien können bei Interlis 1 optional polygoniert werden.

Kreisbögen werden als Kreisbögen importiert und somit nicht segmentiert oder können optional auch segmentiert werden.

Attribute vom Interlis-Datentyp „Enumeration“ können wahlweise auch zusätzlich als Text importiert werden (z.B. BB-Art 0 = „Gebaeude“).

Den Geometrien kann mittels Parameter ein EPSG-Code zugewiesen werden. Die Geometrie-Attribute können optional indexiert werden.

1:1-Export aus der Datenbank
----------------------------

Der 1:1-Export schreibt alle Tabellen eines Interlis-Modells in eine Interlis-Transferdatei.

Geometrien vom Typ Area und Surface werden bei Interlis 1 während dem Export in Linien umgewandelt.

Laufzeitanforderungen
---------------------

Das Programm setzt Java 1.6 voraus.

**PostGIS:** Als Datenbank muss mindestens PostgreSQL 8.3 und PostGIS 1.5 vorhanden sein. Falls das Interlis Datenmodell INTERLIS.UUIDOID als OID verwendet, wird die Funktion uuid\_generate\_v4() verwendet. Dazu muss die PostgreSQL-Erweiterung uuid-ossp konfiguriert sein (`CREATE EXTENSION "uuid-ossp";`).

Lizenz
------

GNU Lesser General Public License

Funktionsweise
==============

In den folgenden Abschnitten wird die Funktionsweise anhand einzelner Anwendungsfälle beispielhaft beschrieben. Die detaillierte Beschreibung einzelner Funktionen ist im Kapitel „Referenz“ zu finden.

Import-Funktionen
-----------------

Fall 1 <sub><sub>~</sub></sub>~

Die Tabellen existieren nicht und sollen in der Datenbank angelegt werden.

**PostGIS:** `java -jar ili2pg.jar --schemaimport --dbdatabase mogis --dbusr julia --dbpwd romeo path/to/dm01av.ili`

**GeoPackage:** `java -jar ili2gpkg.jar --schemaimport --dbfile mogis.gpkg path/to/dm01av.ili`

Es werden keine Daten importiert, sondern nur die leeren Tabellen angelegt.

**PostGIS:** Die leeren Tabellen werden im Default-Schema des Benutzers julia angelegt. Die Geometrie-Spalten werden in der Tabelle public.geometry\_columns registriert.

Als Host wird der lokale Rechner angenommen und für die Verbindung zur Datenbank der Standard-Port.

**GeoPackage:** Die Geometrie-Spalten werden in den Tabellen gpkg\_contents und gpkg\_geometry\_columns registriert.

Falls die Datei mogis.gpkg noch nicht existiert, wird sie erzeugt und mit den für GeoPackage nötigen Metatabellen initialisiert.

Fall 2 (nur PostGIS) <sub><sub><sub><sub>~</sub></sub></sub><sub><sub>~</sub></sub></sub>~<sub>~</sub>

Das gewünschte Schema und die Tabellen existieren nicht und es soll das DB-Schema und -Datenmodell angelegt werden:

**PostGIS:** `java -jar ili2pg.jar --schemaimport --dbdatabase mogis --dbschema dm01av --dbusr julia --dbpwd romeo path/to/dm01av.ili`

Es werden keine Daten importiert, sondern nur das Schema dm01av und die leeren Tabellen angelegt. Die Geometrie-Spalten werden in der Tabelle public.geometry\_columns registriert.

Fall 3 <sub><sub>~</sub></sub>~

Die Tabellen existieren nicht und sollen in der Datenbank angelegt werden und die Daten sollen importiert werden:

**PostGIS:** `java -jar ili2pg.jar --import --dbhost ofaioi4531 --dbport 5432 --dbdatabase mogis --dbusr julia --dbpwd romeo --log path/to/logfile path/to/260100.itf`

**GeoPackage:** `java -jar ili2gpkg.jar --import --dbfile mogis.gpkg --log path/to/logfile path/to/260100.itf`

Alle Tabellen werden in der Datenbank erstellt und das Itf 260100.itf importiert. Die Geometrie-Spalten werden registriert. Als Primary-Key wird ein zusätzliches Attribut erstellt (t\_id). Zusätzlich wir ein t\_basket Attribut erstellt. Dieses zeigt als Fremdschlüssel auf eine Meta-Hilfstabelle (Importdatum, Benutzer, Modellname, Pfad der Itf-Datei).

Die Aufzähltypen werden in Lookup-Tables abgebildet.

Es wird ein Logfile angelegt. Dieses enthält Zeitpunkt des Imports, Name des Benutzers, Datenbankparameter (ohne Passwort), Name (ganzer Pfade) der Ili- und Itf-Datei, sämtliche Namen der importierten Tabellen inkl. Anzahl der importierten Elemente pro Tabelle. Allfällige Fehlermeldungen (bei Importabbruch) werden auch in die Logdatei geschrieben.

Fall 4 <sub><sub>~</sub></sub>~

Die Tabellen existieren bereits und der Inhalt der Tabellen soll erweitert werden:

**PostGIS:** `java -jar ili2pg.jar --import --dbdatabase mogis --dbusr julia --dbpwd romeo path/to/260100.itf`

**GeoPackage:** `java -jar ili2gpkg.jar --import --dbfile mogis.gpkg path/to/260100.itf`

Das Itf 260100.itf wird importiert und die Daten den bereits vorhanden Tabellen hinzugefügt. Die Tabellen können zusätzliche Attribute enthalten (z.B. bfsnr, datum etc.), welche beim Import leer bleiben.

Fall 5 <sub><sub>~</sub></sub>~

Die Tabellen existieren bereits und der Inhalt der Tabellen soll durch den Inhalt des itf ersetzt werden:

**PostGIS:** `java -jar ili2pg.jar --import --deleteData --dbdatabase mogis --dbusr julia --dbpwd romeo path/to/260100.itf`

**GeoPackage:** `java -jar ili2gpkg.jar --import --deleteData --dbfile mogis.gpkg path/to/260100.itf`

Das Itf 260100.itf wird importiert und die bestehenden Daten in den bereits vorhanden Tabellen gelöscht. Die Tabellen können zusätzliche Attribute enthalten (z.B. bfsnr, datum etc.), welche beim Import leer bleiben.

Fall 6 <sub><sub>~</sub></sub>~

Enumerations werden zusätzlich als Textattribut hinzugefügt:

**PostGIS:** `java -jar ili2pg.jar --import --createEnumTxtCol --dbdatabase mogis --dbusr julia --dbpwd romeo path/to/260100.itf`

**GeoPackage:** `java -jar ili2gpkg.jar --import --createEnumTxtCol --dbfile mogis.gpkg path/to/260100.itf`

Das Itf wird in die Datenbank importiert. Zusätzlich werden die Attribute vom Typ Enumeration in ihrer Textrepräsentation (Attribut „art“ = 0 ⇒ „art\_txt“ = „Gebaeude“) hinzugefügt.

Fall 7 <sub><sub>~</sub></sub>~

Den Geometrien wird ein spezieller SRS (Spatial Reference System) Identifikator hinzugefügt:

**PostGIS:** `java -jar ili2pg.jar --import --defaultSrsAuth epsg --defaultSrsCode 2056 --dbdatabase mogis --dbusr julia --dbpwd romeo path/to/260100.itf`

**GeoPackage:** `java -jar ili2gpkg.jar --import --defaultSrsAuth epsg --defaultSrsCode 2056 --dbfile mogis.gpkg path/to/260100.itf`

Das Itf wird in die Datenbank importiert. Zusätzlich wird jeder Geometrie eine SRS-ID

(EPSG-Code 2056) hinzugefügt. Ebenfalls wird derselbe Identifikator für die Registrierung der Geometriespalten in den Metatabellen der Datenbank benutzt.

Fall 8 <sub><sub>~</sub></sub>~

Geometrien werden indexiert:

**PostGIS:** `java -jar ili2pg.jar --import --createGeomIdx --dbdatabase mogis --dbusr julia --dbpwd romeo path/to/260100.itf`

**GeoPackage:** `java -jar ili2gpkg.jar --import --createGeomIdx --dbfile mogis.gpkg path/to/260100.itf`

Das Itf wird in die Datenbank importiert. Die Geometrien werden indexiert.

Fall 9 <sub><sub>~</sub></sub>~

Tauchen beim Import des Itf Fehler auf (z. B. mangelnde Modellkonformität oder verletzte Constraints in der DB), bricht der Import ab und keine Daten werden importiert. D.h. der Import in die Datenbank ist ein einzelner Commit.

Export-Funktionen
-----------------

Fall 1 <sub><sub>~</sub></sub>~

Die Tabellen werden aus der Datenbank in eine Interlis 1-Transfer-Datei geschrieben:

**PostGIS:** `java -jar ili2pg.jar --export --models DM01AV --dbhost ofaioi4531 --dbport 5432 --dbdatabase mogis --dbusr julia --dbpwd romeo path/to/output.itf`

**GeoPackage:** `java -jar ili2gpkg.jar --export --models DM01AV --dbfile mogis.gpkg path/to/output.itf`

Die Tabellen werden dem Interlis-Modell DM01AV entsprechend in die Interlis 1-Transferdatei output.itf geschrieben. Fehlende Tabellen in der Datenbank werden dementsprechend als leere Tabellen oder gar nicht (gemäss Definition im Datenmodell) in die Datei geschrieben. Fehlende Attribute in einer Datenbanktabelle werden mit einem „@“ substituiert.

Anhand des Parameters --models wird definiert, welche Daten exportiert werden. Alternativ kann auch der Parameter --topics oder --baskets verwendet werden, um die zu exportierenden Daten auszuwählen. Einer dieser Parameter muss also zwingend beim Export angegeben werden.

Fall 2 <sub><sub>~</sub></sub>~

Die Tabellen werden aus der Datenbank in eine Interlis 2-Transfer-Datei geschrieben:

**PostGIS:** `java -jar ili2pg.jar --export --models DM01AV --dbhost ofaioi4531 --dbport 5432 --dbdatabase mogis --dbusr julia --dbpwd romeo path/to/output.xtf`

**GeoPackage:** `java -jar ili2gpkg.jar --export --models DM01AV --dbfile mogis.gpkg path/to/output.xtf`

Die Tabellen werden dem Interlis-Modell DM01AV entsprechend in das die Interlis 2-Transferdatei output.xtf geschrieben. Fehlende Tabellen und Attribute in der Datenbank werden gar nicht in die Datei geschrieben.

Anhand des Parameters --models wird definiert, welche Daten exportiert werden. Alternativ kann auch der Parameter --topics oder --baskets verwendet werden, um die zu exportierenden Daten auszuwählen. Einer dieser Parameter muss also zwingend beim Export angegeben werden.

Referenz
========

In den folgenden Abschnitten werden einzelne Aspekte detailliert, aber isoliert, beschrieben. Die Funktionsweise als Ganzes wird anhand einzelner Anwendungsfälle beispielhaft im Kapitel „Funktionsweise“ (weiter oben) beschrieben.

Aufruf-Syntax
-------------

**PostGIS:** `java -jar ili2pg.jar [Options] [file]`

**GeoPackage:** `java -jar ili2gpkg.jar [Options] [file]`

Optionen:

<table>
<colgroup>
<col width="5%" />
<col width="94%" />
</colgroup>
<thead>
<tr class="header">
<th>Option</th>
<th>Beschreibung</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>--import</p></td>
<td><p>Importiert Daten aus einer Transferdatei in die Datenbank.</p>
<p>Die Tabellen werden implizit auch angelegt, falls sie noch nicht vorhanden sind (siehe Kapitel Abbildungsregeln). Falls die Tabellen in der Datenbank schon vorhanden sind, können sie zusätzliche Spalten enthalten (z.B. bfsnr, datum etc.), welche beim Import leer bleiben.</p>
<p>Falls beim Import ein Datensatz-Identifikator (--dataset) definiert wird, darf dieser Datensatz-Identifikator in der Datenbank noch nicht vorhanden sein. Um die bestehenden Daten zu ersetzen, kann die Option --replace verwendet werden.</p>
<p>TODO Die Tabellen sind schon vorhanden (und entsprechen (nicht) der ili-Klasse)</p></td>
</tr>
<tr class="even">
<td><p>--update</p></td>
<td><p>Aktualisiert die Daten in der Datenbank anhand einer Transferdatei, d.h. neue Objekte werden eingefügt, bestehende Objekte werden aktualisiert und in der Transferdatei nicht mehr vorhandene Objekte werden gelöscht. Diese Funktion bedingt, dass das Datenbankschema mit der Option --createBasketCol erstellt wurde, und dass die Klassen und Topics eine stabile OID haben.</p></td>
</tr>
<tr class="odd">
<td><p>--replace</p></td>
<td><p>Ersetzt die Daten in der Datenbank anhand eines Datensatz-Identifikators (--dataset) mit den Daten aus einer Transferdatei. Diese Funktion bedingt, dass das Datenbankschema mit der Option --createBasketCol erstellt wurde.</p></td>
</tr>
<tr class="even">
<td><p>--delete</p></td>
<td><p>Löscht die Daten in der Datenbank anhand eines Datensatz-Identifikators (--dataset). Diese Funktion bedingt, dass das Datenbankschema mit der Option --createBasketCol erstellt wurde.</p></td>
</tr>
<tr class="odd">
<td><p>--export</p></td>
<td><p>Exportiert Daten aus der Datenbank in eine Transferdatei.</p>
<p>Mit dem Parameter --models, --topics, --baskets oder --dataset wird definiert, welche Daten exportiert werden.</p>
<p>Ob die Daten im Interlis 1-, Interlis 2- oder GML-Format geschrieben werden, ergibt sich aus der Dateinamenserweiterung der Ausgabedatei. Für eine Interlis 1-Transferdatei muss die Erweiterung .itf verwendet werden. Für eine GML-Transferdatei muss die Erweiterung .gml verwendet werden.</p>
<p>Die Optionen --topics und --baskets bedingen, dass das Datenbankschema mit der Option --createBasketCol erstellt wurde.</p></td>
</tr>
<tr class="even">
<td><p>--schemaimport</p></td>
<td><p>Erstellt die Tabellenstruktur in der Datenbank (siehe Kapitel Abbildungsregeln).</p></td>
</tr>
<tr class="odd">
<td><p>--validConfig filename</p></td>
<td><p>Name der Konfigurationsdatei, die für die Validierung verwendet werden soll.</p></td>
</tr>
<tr class="even">
<td><p>--disableValidation</p></td>
<td><p>Schaltet die Validierung der Daten aus.</p></td>
</tr>
<tr class="odd">
<td><p>--dbhost host</p></td>
<td><p><strong>PostGIS:</strong> Der hostname der Datenbank. Default ist localhost.</p></td>
</tr>
<tr class="even">
<td><p>--dbport port</p></td>
<td><p><strong>PostGIS:</strong> Die Port-Nummer, unter der die Datenbank angesprochen warden kann. Default ist 5432.</p></td>
</tr>
<tr class="odd">
<td><p>--dbdatabase database</p></td>
<td><p><strong>PostGIS:</strong> Der Name der Datenbank.</p></td>
</tr>
<tr class="even">
<td><p>--dbusr username</p></td>
<td><p><strong>PostGIS:</strong> Der Benutzername für den Datenbankzugang und Einträge in Metatabellen.</p>
<p><strong>GeoPackage:</strong> Der Benutzername für Einträge in Metatabellen.</p></td>
</tr>
<tr class="odd">
<td><p>--dbpwd password</p></td>
<td><p><strong>PostGIS:</strong> Das Passwort für den Datenbankzugriff.</p></td>
</tr>
<tr class="even">
<td><p>--dbschema schema</p></td>
<td><p><strong>PostGIS:</strong> Definiert den Namen des Datenbank-Schemas. Default ist kein Wert, d.h. das aktuelle Schema des Benutzers der mit –user definiert wird.</p></td>
</tr>
<tr class="odd">
<td><p>--dbfile filename</p></td>
<td><p><strong>GeoPackage:</strong> Name der GeoPackage-Datei.</p></td>
</tr>
<tr class="even">
<td><p>--deleteData</p></td>
<td><p>bei einem Datenimport (--import) werden alle Daten in den existierenden/benutzten Tabellen gelöscht (Mit DELETE, die Tabellenstruktur bleibt unverändert).</p></td>
</tr>
<tr class="odd">
<td><p>--defaultSrsAuth auth</p></td>
<td><p>SRS Authority für Geometriespalten, wo sich dieser Wert nicht ermitteln lässt (für ili1 und ili2.3 immer der Fall). Default ist EPSG</p></td>
</tr>
<tr class="even">
<td><p>--defaultSrsCode code</p></td>
<td><p>SRS Code für Geometriespalten, wo sich dieser Wert nicht ermitteln lässt (für ili1 und ili2.3 immer der Fall). Default ist 21781</p></td>
</tr>
<tr class="odd">
<td><p>--modeldir path</p></td>
<td><p>Dateipfade, die Modell-Dateien (ili-Dateien) enthalten. Mehrere Pfade können durch Semikolon ‚;‘ getrennt werden. Es sind auch URLs von Modell-Repositories möglich. Default ist</p>
<p>%ILI_FROM_DB;%XTF_DIR;http://models.interlis.ch/;%JAR_DIR</p>
<p>%ILI_FROM_DB ist ein Platzhalter für die in der Datenbank vorhandenen Modelle (in der Tabelle t_ili2db_model).</p>
<p>%XTF_DIR ist ein Platzhalter für das Verzeichnis mit der Transferdatei.</p>
<p>%JAR_DIR ist ein Platzhalter für das Verzeichnis des ili2db Programms (ili2pg.jar bzw. ili2gpkg.jar Datei).</p>
<p>Der erste Modellname (Hauptmodell), zu dem ili2db die ili-Datei sucht, ist nicht von der INTERLIS-Sprachversion abhängig. Es wird in folgender Reihenfolge nach einer ili-Datei gesucht: zuerst INTERLIS 2.3, dann 1.0 und zuletzt 2.2.</p>
<p>Beim Auflösen eines IMPORTs wird die INTERLIS Sprachversion des Hauptmodells berücksichtigt, so dass also z.B. das Modell Units für ili2.2 oder ili2.3 unterschieden wird.</p></td>
</tr>
<tr class="even">
<td><p>--models modelname</p></td>
<td><p>Namen des Modells (nicht zwingend identisch mit dem Dateinamen!), für das die Tabellenstruktur in der Datenbank erstellt werden soll. Mehrere Modellnamen können durch Semikolon ‚;‘ getrennt werden. Normalerweise muss der Namen nicht angegeben werden, und das Programm ermittelt den Wert automatisch aus den Daten. Wird beim --schemaimport nur eine ili-Datei als file angegeben, wird der Name des letzten Modells aus dieser ili-Datei als modelname genommen.</p></td>
</tr>
<tr class="odd">
<td><p>--dataset name</p></td>
<td><p>Name/Identifikator des Datensatzes (Kurzform für mehrere BIDs). Kann z.B. eine BFSNr oder ein Kantonskürzel sein. Bedingt die Option --createBasketCol.</p></td>
</tr>
<tr class="even">
<td><p>--baskets BID</p></td>
<td><p>BID der Baskets, die importiert oder exportiert werden sollen. Mehrere BIDs können durch Semikolon ‚;‘ getrennt werden.</p></td>
</tr>
<tr class="odd">
<td><p>--topics topicname</p></td>
<td><p>Topic-Namen der Baskets, die importiert oder exportiert werden sollen. Mehrere Namen können durch Semikolon ‚;‘ getrennt werden. Falls der Topic-Name in verschiedenen Modellen vorkommt, muss der qualifizierte Topic-Name verwendet werden.</p></td>
</tr>
<tr class="even">
<td><p>--createscript filename</p></td>
<td><p>Erstellt zusätzlich zur Tabellenstruktur in der Datenbank ein SQL-Skript um die Tabellenstruktur unabhängig vom Programm erstellen zu können. Das Skript wird zusätzlich zu den Tabellen in der Datenbank erzeugt, d.h. es ist nicht möglich, nur das Skript zu erstellen (ohne Datenbank).</p></td>
</tr>
<tr class="odd">
<td><p>--dropscript filename</p></td>
<td><p>Erstellt ein SQL-Skript um die Tabellenstruktur unabhängig vom Programm löschen zu können.</p></td>
</tr>
<tr class="even">
<td><p>--noSmartMapping</p></td>
<td><p>Alle strukturellen Abbildungsoptimierungen werden ausgeschaltet. (s.a. --smart1Inheritance, --coalesceCatalogueRef, --coalesceMultiSurface, --expandMultilingual)</p></td>
</tr>
<tr class="odd">
<td><p>--smart1Inheritance</p></td>
<td><p>Bildet die Vererbungshierarchie mit einer dymamischen Strategie ab. Für Klassen, die referenziert werden und deren Basisklassen nicht mit einer NewClass-Strategie abgebildet werden, wird die NewClass-Strategie verwendet. Abstrakte Klassen werden mit einer SubClass-Strategie abgebildet. Konkrete Klassen, ohne Basisklasse oder deren direkte Basisklassen mit einer SubClass-Strategie abgebildet werden, werden mit einer NewClass-Strategie abgebildet. Alle anderen Klassen werden mit einer SuperClass-Strategie abgebildet.</p></td>
</tr>
<tr class="even">
<td><p>--smart2Inheritance</p></td>
<td><p>Bildet die Vererbungshierarchie mit einer dymamischen Strategie ab. Abstrakte Klassen werden mit einer SubClass-Strategie abgebildet. Konkrete Klassen werden mit einer NewAndSubClass-Strategie abgebildet.</p></td>
</tr>
<tr class="odd">
<td><p>--coalesceCatalogueRef</p></td>
<td><p>Strukturattribute deren maximale Kardinalität 1 ist, deren Basistyp CHBase:CatalogueReference oder CHBase:MandatoryCatalogueReference ist und die ausser „Reference“ keine weiteren Attribute haben, werden direkt mit einem Fremdschlüssel auf die Ziel-Tabelle (die die konkrete CHBase:Item Klasse realisiert) abgebildet, d.h. kein Record in der Tabelle für die Struktur mit dem „Reference“ Attribut.</p></td>
</tr>
<tr class="even">
<td><p>--coalesceMultiSurface</p></td>
<td><p>Strukturattribute deren maximale Kardinalität 1 ist, deren Basistyp CHBase:MultiSurface ist und die ausser „Surfaces“ keine weiteren Attribute haben, werden direkt als Spalte mit dem Typ MULTISURFACE (oder MULTIPOLYGON, falls --strokeArcs) abgebildet.</p></td>
</tr>
<tr class="odd">
<td><p>--expandMultilingual</p></td>
<td><p>Strukturattribute deren maximale Kardinalität 1 ist, deren Basistyp LocalisationCH_V1.MultilingualText oder LocalisationCH_V1.MultilingualMText ist und die ausser „LocalisedText“ keine weiteren Attribute haben, werden direkt als Spalten in der Tabelle des Strukturattributes abgebildet, d.h. keine Records in den Tabellen für die Multilingual-Strukturen.</p></td>
</tr>
<tr class="even">
<td><p>--createGeomIdx</p></td>
<td><p>Erstellt für jede Geometriespalte in der Datenbank einen räumlichen Index. (siehe Kapitel Abbildungsregeln/Geometrieattribute)</p></td>
</tr>
<tr class="odd">
<td><p>--createEnumColAsItfCode</p></td>
<td><p>Bildet bei Aufzählungsattributen den Aufzählungswert als ITF-Code ab. Diese Option ist nur zulässig, wenn im Modell keine Erweiterungen von Aufzählungen vorkommen. Ohne diese Option wird der XTF-Code als Aufzählwert in der Datenbank verwendet. (siehe Kapitel Abbildungsregeln/Aufzählungen)</p></td>
</tr>
<tr class="even">
<td><p>--createEnumTxtCol</p></td>
<td><p>Erstellt für Aufzählungsattribute eine zusätzliche Spalte mit dem Namen des Aufzählwertes. (siehe Kapitel Abbildungsregeln/Aufzählungen)</p></td>
</tr>
<tr class="odd">
<td><p>--createEnumTabs</p></td>
<td><p>Erstellt pro Aufzählungsdefinition eine Tabelle mit den einzelnen Aufzählwerten. (siehe Kapitel Abbildungsregeln/Aufzählungen)</p></td>
</tr>
<tr class="even">
<td><p>--createSingleEnumTab</p></td>
<td><p>Erstellt eine einzige Tablle mit allen Aufzählwerten aller Aufzählungsdefinitionen. (siehe Kapitel Abbildungsregeln/Aufzählungen)</p></td>
</tr>
<tr class="odd">
<td><p>--beautifyEnumDispName</p></td>
<td><p>Verschönert den Anzeigetext für das Aufzählelement. Beim Import wird die Spalte mit dem XTF-Code ohne Untersstriche befüllt (&quot;Strasse befestigt&quot; statt &quot;Strasse_befestigt&quot;) (siehe Kapitel Abbildungsregeln/Aufzählungen)</p></td>
</tr>
<tr class="even">
<td><p>--createStdCols</p></td>
<td><p>Erstellt in jeder Tabelle zusätzliche Metadatenspalten T_User, T_CreateDate, T_LastChange. (siehe Kapitel Abbildungsregeln/Tabellen)</p></td>
</tr>
<tr class="odd">
<td><p>--t_id_Name name</p></td>
<td><p>Definiert den Namen für die interne technische Schlüsselspalte in jeder Tabelle (nicht zu verwechseln mit dem externen Transferidentifikator). Default ist T_Id. (siehe Kapitel Abbildungsregeln/Tabellen)</p></td>
</tr>
<tr class="even">
<td><p>--idSeqMin zahl</p></td>
<td><p><strong>PostGIS:</strong> Definiert den Minimalwert für den Generator der internen technischen Schlüssel</p></td>
</tr>
<tr class="odd">
<td><p>--idSeqMax zahl</p></td>
<td><p><strong>PostGIS:</strong> Definiert den Maximalwert für den Generator der internen technischen Schlüssel</p></td>
</tr>
<tr class="even">
<td><p>--createTypeDiscriminator</p></td>
<td><p>Erstellt für jede Tabelle (auch wenn das Modell keine Vererbung benutzt) eine Spalte für den Typdiskriminator. Für Klassen mit Vererbung wird die Spalte immer erstellt. (siehe Kapitel Abbildungsregeln/Tabellen)</p></td>
</tr>
<tr class="odd">
<td><p>--structWithGenericRef</p></td>
<td><p>Erstellt generische Spalten für den Fremdschlüssel bei Tabellen die Interlis-Strukturen abbilden. Ohne diese Option wird pro Strukturattribut eine Spalte erstellt (in der Tabelle, die die Struktur abbildet). (siehe Kapitel Abbildungsregeln/Strukturen)</p></td>
</tr>
<tr class="even">
<td><p>--disableNameOptimization</p></td>
<td><p>Schaltet die Nutzung von unqualifizierten Klassennamen aus. Für alle Tabellennamen werden qualifizierte Interlis-Klassennamen (Model.Topic.Class) verwendet (und in einen gültigen Tabellennamen abgebildet). (siehe Kapitel Abbildungsregeln/Namenskonventionen)</p></td>
</tr>
<tr class="odd">
<td><p>--nameByTopic</p></td>
<td><p>Für alle Tabellennamen werden teilweise qualifizierte Interlis-Klassennamen (Topic.Class) verwendet (und in einen gültigen Tabellennamen abgebildet). (siehe Kapitel Abbildungsregeln/Namenskonventionen)</p></td>
</tr>
<tr class="even">
<td><p>--maxNameLength length</p></td>
<td><p>Definiert die maximale Länge der Namen für Datenbankelemente (Tabellennamen, Spaltennamen , usw.) Default ist 60. Ist der Interlis-Name länger, wird er gekürzt. (siehe Kapitel Abbildungsregeln/Namenskonventionen)</p></td>
</tr>
<tr class="odd">
<td><p>--sqlEnableNull</p></td>
<td><p>Erstellt keine NOT NULL Anweisungen bei Spalten die Interlis-Attribute abbilden. (siehe Kapitel Abbildungsregeln/Attribute)</p></td>
</tr>
<tr class="even">
<td><p>--strokeArcs</p></td>
<td><p>Segmentiert Kreisbogen beim Datenimport. Der Radius geht somit verloren. Die Kreisbogen werden so segmentiert, dass die Abweichung der erzeugten Geraden kleiner als die Koordinatengenauigkeit der Stützpunkte ist.</p></td>
</tr>
<tr class="odd">
<td><p>--oneGeomPerTable</p></td>
<td><p><strong>PostGIS:</strong> Erzeugt Hilfstabellen, falls in einer Klasse/Tabelle mehr als ein Geometrie-Attribut ist, so dass pro Tabelle in der Datenbank nur eine Geometriespalte ist.</p></td>
</tr>
<tr class="even">
<td><p>--skipPolygonBuilding</p></td>
<td><p>Bei ITF-Dateien werden die Linientabellen gelesen, so wie sie in der ITF-Datei sind, d.h. es werden keine Polygon gebildet.</p></td>
</tr>
<tr class="odd">
<td><p>--skipPolygonBuildingErrors</p></td>
<td><p>Bei ITF-Dateien werden aus den Linientabellen Polygone gebildet, aber Fehler werden ignoriert (aber trotzdem rapportiert).</p></td>
</tr>
<tr class="even">
<td><p>--keepAreaRef</p></td>
<td><p>Bei ITF-Dateien wird für AREA Attribute der Gebietsreferenzpunkt als zusätzliche Spalte in der Tabelle eingefügt.</p></td>
</tr>
<tr class="odd">
<td><p>--importTid</p></td>
<td><p>Liest die Transferidentifikation (aus der Transferdatei) in eine zusätzliche Spalte T_Ili_Tid. (siehe Kapitel Abbildungsregeln/Tabellen)</p></td>
</tr>
<tr class="even">
<td><p>--createBasketCol</p></td>
<td><p>Erstellt in jeder Tabelle eine zusätzlich Spalte T_basket um den Behälter identifizieren zu können. (siehe Kapitel Abbildungsregeln/Metadaten)</p></td>
</tr>
<tr class="odd">
<td><p>--createFk</p></td>
<td><p>Erzeugt eine Fremdschlüsselbedingung bei Spalten die Records in anderen Tabellen referenzieren.</p></td>
</tr>
<tr class="even">
<td><p>--createFkIdx</p></td>
<td><p>Erstellt für jede Fremdschlüsselpalte in der Datenbank einen Index. Kann auch ohne die Option --createFk benutzt werden.</p></td>
</tr>
<tr class="odd">
<td><p>--createUnique</p></td>
<td><p>Erstellt für INTERLIS-UNIQUE-Constraints in der Datenbank UNIQUE Bedingungen (sofern abbildbar).</p></td>
</tr>
<tr class="even">
<td><p>--createNumChecks</p></td>
<td><p>Erstellt für numerische Datentypen CHECK-Constraints in der Datenbank.</p></td>
</tr>
<tr class="odd">
<td><p>--ILIGML20</p></td>
<td><p>Verwendet beim Export eCH-0118-2.0 als Transferformat.</p></td>
</tr>
<tr class="even">
<td><p>--log filename</p></td>
<td><p>Schreibt die log-Meldungen in eine Datei.</p></td>
</tr>
<tr class="odd">
<td><p>--gui</p></td>
<td><p>Startet ein einfaches GUI.</p></td>
</tr>
<tr class="even">
<td><p>--trace</p></td>
<td><p>Erzeugt zusätzliche Log-Meldungen (wichtig für Programm-Fehleranalysen)</p></td>
</tr>
<tr class="odd">
<td><p>--help</p></td>
<td><p>Zeigt einen kurzen Hilfetext an.</p></td>
</tr>
<tr class="even">
<td><p>--version</p></td>
<td><p>Zeigt die Version des Programmes an.</p></td>
</tr>
</tbody>
</table>

Abbildungsregeln
----------------

Klassen/Strukturen <sub><sub>~</sub><sub><sub><sub><sub>~</sub></sub></sub></sub></sub>~~

Je nach Programmoption, werden Klassen unterschiedlich abgebildet. Die Abbildungsregeln für den Tabellennamen sind im Abschnitt Namenskonventionen beschrieben.

<table>
<colgroup>
<col width="3%" />
<col width="5%" />
<col width="8%" />
<col width="82%" />
</colgroup>
<thead>
<tr class="header">
<th>Nummer</th>
<th>Beispiel INTERLIS</th>
<th>Beispiel SQL</th>
<th>Kommentare</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>1</p></td>
<td><p>::</p>
<p>CLASS A= END A;</p></td>
<td><p>::</p>
<p>CREATE TABLE A ( T_Id integer PRIMARY KEY );</p></td>
<td><p>Für jede Klasse wird eine Tabelle erstellt.</p>
<p>Jede Tabelle hat mindestens eine Spalte T_Id. Diese Spalte ist der Datenbank interne Primärschlüssel (und nicht die TID aus der Transferdatei).</p></td>
</tr>
<tr class="even">
<td><p>2</p></td>
<td><p>::</p>
<p>CLASS A = END A;</p></td>
<td><p>::</p>
<p>CREATE TABLE A ( T_Id integer PRIMARY KEY, T_Type varchar(60) NOT NULL );</p></td>
<td><p>Mit der Option --createTypeDiscriminator erhält jede Tabelle (die eine Klasse oder Struktur repräsentiert, die keine Basisklasse hat) eine zusätzliche Spalte T_Type. Diese Spalte enthält den konkreten Klassenname (der SQL-Name des qualifizierten INTERLIS-Klassennamens [2]_) des Objektes jedes einzelnen Records.</p>
<p>Tabellen für Klassen die eine Basisklasse haben, erhalten diese Spalte nicht.</p></td>
</tr>
<tr class="odd">
<td><p>3</p></td>
<td><p>::</p>
<p>CLASS A = END A;</p></td>
<td><p>::</p>
<p>CREATE TABLE A ( T_Id integer PRIMARY KEY, T_LastChange timestamp NOT NULL, T_CreateDate timestamp NOT NULL, T_User varchar(40) NOT NULL );</p></td>
<td><p>Mit der Option --createStdCols erhalten alle Tabellen drei zusätzliche Spalten für den Zeitpunkt der letzten Änderung, den Zeitpunkt der Erstellung und den Benutzer, der die letzte Änderung durchgeführt hat. Diese Spalten müssen durch die Applikation nachgeführt werden, und werden typischerweise für die Implementierung eines optimistischen Lockings benötigt/benutzt.</p></td>
</tr>
<tr class="even">
<td><p>4</p></td>
<td><p>::</p>
<p>CLASS A = END A;</p></td>
<td><p>::</p>
<p>CREATE TABLE A ( T_Id integer PRIMARY KEY, T_Ili_Tid varchar(200) NULL );</p></td>
<td><p>Mit der Option --importTid erhält jedes Tabelle (die eine Klasse repräsentiert, die keine Basisklasse hat) eine zusätzliche Spalte T_Ili_Tid. Diese Spalte enthält die TID aus der Transferdatei.</p>
<p>Diese Spalte ist NICHT der Datenbank interne Primärschlüssel.</p></td>
</tr>
<tr class="odd">
<td><p>5</p></td>
<td><p>::</p>
<p>CLASS A = END A;</p></td>
<td><p>::</p>
<p>CREATE TABLE A ( oidname integer PRIMARY KEY );</p></td>
<td><p>Mit der Option --t_id_Name oidname wird der Namen der Spalte für den Datenbank internen Primärschlüssel (nicht die Spalte für die TID aus der Transferdatei) festgelegt.</p></td>
</tr>
<tr class="even">
<td><p>6</p></td>
<td><p>::</p>
<p>STRUCTURE C = END C;</p></td>
<td><p>::</p>
<p>CREATE TABLE C ( T_Id integer PRIMARY KEY, T_seq integer NOT NULL );</p></td>
<td><p>Strukturen werden im Allgemeinen abgebildet wie Klassen.</p>
<p>Die Strukturtabelle enthält zusätzlich eine Spalte T_seq, die die Reihenfolge der Strukturelement festlegt.</p>
<p>Da Strukturelemente keine TID haben, erhalten sie auch mit der Option --importTid kein Spalte T_Ili_Tid.</p></td>
</tr>
</tbody>
</table>

Vererbung <sub>~<sub>~</sub></sub>~

Im allgemeinen lässt sich Vererbung nach drei unterschidlichen Strategien abbilden:

NewClass Diese Strategie ist für jede Klasse möglich. Bei dieser Strategie wird für eine Klasse eine neue Tabelle angelegt, ein Interlis-Objekt verteilt sich somit auf Records in mehreren Tabellen.

SuperClass Diese Strategie ist nur für Klassen mit einer Super-Klasse möglich. Bei dieser Strategie wird für die Klasse keine neue Tabelle angelegt, d.h. die Attribute der Klasse werden als weitere Spalten in der Tabelle der Super-Klasse ergänzt.

SubClass Diese Strategie ist nur für Klassen mit mindestens einer Sub-Klasse möglich. Bei dieser Strategie wird für eine Klasse keine neue Tabelle angelegt, d.h. die Attribute der Klasse werden als weitere Spalten in den Tabellen der Sub-Klassen ergänzt.

ili2db bildet die Vererbung nach einer je nach Klasse unterschiedlichen Strategie (--smart1Inheritance oder --smart2Inheritance) oder für alle Klassen einheitlich nach der NewClass-Strategie (--noSmartMapping) ab.

Bei --smart1Inheritance wird wie folgt abgebildet: Fuer Klassen, die referenziert werden und deren Basisklassen nicht mit einer NewClass-Strategie abgebildet werden, wird die NewClass-Strategie verwendet. Abstrakte Klassen werden mit einer SubClass-Strategie abgebildet. Konkrete Klassen, ohne Basisklasse oder deren direkte Basisklassen mit einer SubClass-Strategie abgebildet werden, werden mit einer NewClass-Strategie abgebildet. Alle anderen Klassen werden mit einer SuperClass-Strategie abgebildet.

Bei --smart2Inheritance wird wie folgt abgebildet: Abstrakte Klassen werden mit einer SubClass-Strategie abgebildet. Konkrete Klassen werden mit einer NewAndSubClass-Strategie abgebildet.

<table>
<colgroup>
<col width="8%" />
<col width="15%" />
<col width="18%" />
<col width="57%" />
</colgroup>
<thead>
<tr class="header">
<th>Nummer</th>
<th>Beispiel INTERLIS</th>
<th>Beispiel SQL</th>
<th>Kommentare</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>1</p></td>
<td><p>::</p>
<p>CLASS A = Attribut_1 : TEXT*20; END A;</p>
<p>CLASS B EXTENDS A = Attribut_2 : TEST*20; END B;</p></td>
<td><p>::</p>
<p>CREATE TABLE A ( T_Id integer PRIMARY KEY, T_Type varchar(60) NOT NULL, Attribut_1 varchar(20) );</p>
<p>CREATE TABLE B ( T_Id integer PRIMARY KEY, Attribut_2 varchar(20) );</p></td>
<td><p>Bei --noSmartMapping wird für jede Klasse eine Tabelle erstellt. Ein Objekt A ergibt ein Record in Tabellen A. Ein Objekt B ergibt je ein Record in Tabellen A und B. Die T_Id ist bei beiden Records identisch.</p></td>
</tr>
<tr class="even">
<td><p>2</p></td>
<td><p>::</p>
<p>CLASS A (ABSTRACT) = Attribut_1 : TEXT*20; END A;</p>
<p>CLASS B EXTENDS A = Attribut_2 : TEST*20; END B;</p>
<p>CLASS C EXTENDS B = Attribut_3 : TEST*20; END C;</p></td>
<td><p>::</p>
<p>CREATE TABLE B ( T_Id integer PRIMARY KEY, T_Type varchar(60) NOT NULL, Attribut_1 varchar(20), Attribut_2 varchar(20), Attribut_3 varchar(20) );</p></td>
<td><p>Bei --smart1Inheritance wird für abstrakte Klassen (A) keine Tabelle erstellt (ausser sie wird referenziert). Für die allgemeinste konkrete Klasse (B) wird eine Tabelle erstellt. Für erweiterte konkrete Klassen (C), die eine konkrete Klasse erweitern, wird keine eigene Tabelle erstellt.</p></td>
</tr>
<tr class="odd">
<td><p>3</p></td>
<td><p>::</p>
<p>CLASS A (ABSTRACT) = Attribut_1 : TEXT*20; END A;</p>
<p>CLASS B EXTENDS A = Attribut_2 : TEST*20; END B;</p>
<p>CLASS C EXTENDS B = Attribut_3 : TEST*20; END B;</p></td>
<td><p>::</p>
<p>CREATE TABLE B ( T_Id integer PRIMARY KEY, T_Type varchar(60) NOT NULL, Attribut_1 varchar(20), Attribut_2 varchar(20) );</p>
<p>CREATE TABLE C ( T_Id integer PRIMARY KEY, T_Type varchar(60) NOT NULL, Attribut_1 varchar(20), Attribut_2 varchar(20), Attribut_3 varchar(20) );</p></td>
<td><p>Bei --smart2Inheritance wird für abstrakte Klassen (A) keine Tabelle erstellt (auch nicht, wenn sie referenziert wird). Für konkrete Klassen (B und C) wird je eine vollständige Tabelle erstellt (inkl. geerbte Attribute).</p></td>
</tr>
</tbody>
</table>

EXTENDED Attribute ergeben keine Spalte, nur die Basis-Definition des Attributs ergibt eine Spalte.

Attribute (allgemein) <sub><sub><sub><sub><sub>~</sub></sub></sub></sub><sub>~</sub>~</sub>~~

<table>
<colgroup>
<col width="7%" />
<col width="31%" />
<col width="19%" />
<col width="41%" />
</colgroup>
<thead>
<tr class="header">
<th>Nummer</th>
<th>Beispiel INTERLIS</th>
<th>Beispiel SQL</th>
<th>Kommentare</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>1</p></td>
<td><p>::</p>
<p>textLimited : TEXT<em>10; textUnlimited : TEXT; mtextLimited : MTEXT</em>10; mtextUnlimited : MTEXT;</p></td>
<td><p>::</p>
<p>textLimited varchar(10) NULL textUnlimited text NULL mtextLimited varchar(10) NULL mtextUnlimited text NULL</p></td>
<td></td>
</tr>
<tr class="even">
<td><p>2</p></td>
<td><p>::</p>
<p>aufzaehlung : (null, eins, zwei, drei, mehr ( vier, fuenf, sechs, sieben, acht ,neun, zehn) );</p></td>
<td><p>::</p>
<p>aufzaehlung varchar(255) NULL</p></td>
<td><p>Je nach Option, sind andere Abbildungen möglich. Siehe Kapitel Aufzählungen.</p></td>
</tr>
<tr class="odd">
<td><p>3</p></td>
<td><p>::</p>
<p>horizAlignment : HALIGNMENT; vertAlignment : VALIGNMENT;</p></td>
<td><p>::</p>
<p>horizAlignment varchar(255) NULL vertAlignment varchar(255) NULL</p></td>
<td></td>
</tr>
<tr class="even">
<td><p>4</p></td>
<td><p>::</p>
<p>aBoolean : BOOLEAN;</p></td>
<td><p>::</p>
<p>aBoolean boolean NULL</p></td>
<td></td>
</tr>
<tr class="odd">
<td><p>5</p></td>
<td><p>::</p>
<p>numericInt : 0 .. 10; numericDec : 0.0 .. 10.0;</p></td>
<td><p>::</p>
<p>numericInt integer NULL numericDec decimal(4,1) NULL</p></td>
<td></td>
</tr>
<tr class="even">
<td><p>6</p></td>
<td><p>::</p>
<p>aTime : INTERLIS.XMLTime; aDate : INTERLIS.XMLDate; aDateTime : INTERLIS.XMLDateTime;</p></td>
<td><p>::</p>
<p>aTime time NULL aDate date NULL aDateTime timestamp NULL</p></td>
<td></td>
</tr>
<tr class="odd">
<td><p>7</p></td>
<td><p>::</p>
<p>aOid : OID TEXT*30; aUuid : INTERLIS.UUIDOID;</p></td>
<td><p>::</p>
<p>aOid varchar(255) NULL aUuid uuid NULL</p></td>
<td></td>
</tr>
<tr class="even">
<td><p>8</p></td>
<td><p>::</p>
<p>aClass : CLASS;</p></td>
<td><p>::</p>
<p>aClass varchar(255) NULL</p></td>
<td></td>
</tr>
</tbody>
</table>

Beziehungen/Referenzattribute <sub><sub><sub><sub><sub><sub><sub><sub><sub><sub><sub><sub><sub>~</sub></sub></sub></sub></sub></sub></sub></sub></sub></sub></sub></sub></sub>~~

TODO

Geometrieattribute (allgemein) <sub><sub><sub><sub><sub><sub><sub><sub><sub><sub><sub><sub><sub><sub>~</sub></sub></sub></sub></sub></sub></sub></sub></sub></sub></sub></sub></sub></sub>~

TODO

SURFACE/AREA/ITF/XTF <sub><sub><sub><sub>~</sub></sub></sub><sub><sub>~</sub></sub></sub>~<sub>~</sub>

TODO

Strukturattribute <sub>~<sub><sub><sub><sub><sub>~</sub></sub></sub></sub></sub></sub>~

Strukturen werden im Allgemeinen abgebildet wie Klassen (siehe Kapitel zu der Abbildung von Klassen). Strukturattribute (also wenn eine Struktur als Attributstyp verwendet wird, z.B. bei BAG OF oder LIST OF) werden unabhängig von der Kardinalität durch einen Fremdschlüssel bei der Tabelle der Struktur abgebildet. Bei gewissen Strukturen wird bei Smart-Mapping eine alternative Abbildung verwendet.

<table>
<colgroup>
<col width="4%" />
<col width="8%" />
<col width="12%" />
<col width="75%" />
</colgroup>
<thead>
<tr class="header">
<th>Nummer</th>
<th>Beispiel INTERLIS</th>
<th>Beispiel SQL</th>
<th>Kommentare</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>1</p></td>
<td><p>::</p>
<p>STRUCTURE C = END C;</p>
<p>CLASS D = attr1 : LIST OF C; attr2 : LIST OF C; END D;</p></td>
<td><p>::</p>
<p>CREATE TABLE C ( T_Id integer PRIMARY KEY, T_seq integer NOT NULL, D_attr1 integer, D_attr2 integer );</p>
<p>CREATE TABLE D ( T_Id integer PRIMARY KEY );</p></td>
<td><p>Für jedes Strukturattribut wird in der Tabelle der Struktur eine Spalte für den Fremdschlüssel erstellt. Der Name der Spalte ist der qualifizierte INTERLIS-Attributnamen [3]_.</p>
<p>Die Strukturtabelle enthält zusätzlich eine Spalte T_seq die die Reihenfolge der Strukturelement festlegt.</p></td>
</tr>
<tr class="even">
<td><p>2</p></td>
<td><p>::</p>
<p>STRUCTURE C = END C;</p>
<p>CLASS D = attr1 : LIST OF C; END D;</p></td>
<td><p>::</p>
<p>CREATE TABLE C ( T_Id integer PRIMARY KEY, T_seq integer NOT NULL, T_ParentId integer NOT NULL T_ParentType varchar(60) NOT NULL T_ParentAttr varchar(60) NOT NULL );</p>
<p>CREATE TABLE D ( T_Id integer PRIMARY KEY );</p></td>
<td><p>Mit der Option --structWithGenericRef werden statt für jedes Strukturattribut eine Spalte nur drei Standardspalten T_ParentId, T_ParentType, T_ParentAttr angelegt. Diese drei Spalten bilden zusammen einen generischen Fremdschlüssel.</p>
<p>T_ParentId ist die t_id des Objektes, das das Strukturelement enthält.</p>
<p>T_ParentType ist die konkrete Klasse (der SQL-Name des qualifizierten INTERLIS-Klassennamens [4]_) des Objektes, das das Strukturelement enthält.</p>
<p>T_ParentAttr ist der Strukturattributname (der SQL-Name des unqualifizierten INTERLIS-Attributnamens) in der Klasse des Objektes, das das Strukturelement enthält.</p></td>
</tr>
</tbody>
</table>

Beispiel XML::

|&lt;BspTable.TopicA.D TID="2"&gt; | <attr1>
| &lt;BspTable.TopicA.C&gt; | &lt;/BspTable.TopicA.C&gt; | &lt;BspTable.TopicA.C&gt; | &lt;/BspTable.TopicA.C&gt; | </attr1> | <attr2> | &lt;BspTable.TopicA.C&gt; | &lt;/BspTable.TopicA.C&gt; | </attr2> |&lt;/BspTable.TopicA.D&gt;

Beispiel für Abbildungsvariante 1:

<table style="width:71%;">
<colgroup>
<col width="19%" />
<col width="15%" />
<col width="18%" />
<col width="18%" />
</colgroup>
<thead>
<tr class="header">
<th>Tabelle C</th>
<th></th>
<th></th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>t_id</p></td>
<td><p>t_seq</p></td>
<td><p>D_attr1</p></td>
<td><p>D_attr2</p></td>
</tr>
<tr class="even">
<td><p>7</p></td>
<td><p>0</p></td>
<td><p>6</p></td>
<td></td>
</tr>
<tr class="odd">
<td><p>8</p></td>
<td><p>1</p></td>
<td><p>6</p></td>
<td></td>
</tr>
<tr class="even">
<td><p>9</p></td>
<td><p>0</p></td>
<td></td>
<td><p>6</p></td>
</tr>
</tbody>
</table>

<table style="width:42%;">
<colgroup>
<col width="19%" />
<col width="22%" />
</colgroup>
<thead>
<tr class="header">
<th>Tabelle D</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>t_id</p></td>
<td><p>T_Ili_Tid</p></td>
</tr>
<tr class="even">
<td><p>6</p></td>
<td><p>2</p></td>
</tr>
</tbody>
</table>

Beispiel für Abbildungsvariante 2:

<table>
<colgroup>
<col width="18%" />
<col width="14%" />
<col width="20%" />
<col width="23%" />
<col width="23%" />
</colgroup>
<thead>
<tr class="header">
<th>Tabelle C</th>
<th></th>
<th></th>
<th></th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>t_id</p></td>
<td><p>t_seq</p></td>
<td><p>t_parentid</p></td>
<td><p>t_parenttype</p></td>
<td><p>t_parentattr</p></td>
</tr>
<tr class="even">
<td><p>7</p></td>
<td><p>0</p></td>
<td><p>6</p></td>
<td><p>D</p></td>
<td><p>attr1</p></td>
</tr>
<tr class="odd">
<td><p>8</p></td>
<td><p>1</p></td>
<td><p>6</p></td>
<td><p>D</p></td>
<td><p>attr1</p></td>
</tr>
<tr class="even">
<td><p>9</p></td>
<td><p>0</p></td>
<td><p>6</p></td>
<td><p>D</p></td>
<td><p>attr2</p></td>
</tr>
</tbody>
</table>

<table style="width:42%;">
<colgroup>
<col width="19%" />
<col width="22%" />
</colgroup>
<thead>
<tr class="header">
<th>Tabelle D</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>t_id</p></td>
<td><p>T_Ili_Tid</p></td>
</tr>
<tr class="even">
<td><p>6</p></td>
<td><p>2</p></td>
</tr>
</tbody>
</table>

Bei den folgenden Strukturen wird bei Smart-Mapping für die Strukturattribute eine alternative Abbildung verwendet:

-   Strukturen mit dem Interlis Metaattribut ili2db.mapping=MultiSurface

-   GeometryCHLV03\_V1.MultiSurface

-   GeometryCHLV95\_V1.MultiSurface

-   CatalogueObjects\_V1.Catalogues.CatalogueReference

-   CatalogueObjects\_V1.Catalogues.MandatoryCatalogueReference

-   LocalisationCH\_V1.MultilingualMText

-   LocalisationCH\_V1.MultilingualText

Aufzählungen <sub><sub><sub><sub>~</sub></sub></sub>~</sub>

Für die Abbildung von Aufzählungen gibt es zwei Varianten und verschiedene Optionen.

-   Variante 1 bei der der Aufzählwert als XTF-Code gespeichert wird

-   Variante 2 bei der der Aufzählwert als ITF-Code gespeichert wird

-   Optional kann eine zusätzliche Spalte erzeugt werden, die den Anzeigtext enthalten kann

-   Optional können zusätzliche Tabellen erzeugt werden, die alle Aufzählwerte enthalten.

<table>
<colgroup>
<col width="7%" />
<col width="31%" />
<col width="19%" />
<col width="41%" />
</colgroup>
<thead>
<tr class="header">
<th>Nummer</th>
<th>Beispiel INTERLIS</th>
<th>Beispiel SQL</th>
<th>Kommentare</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>1</p></td>
<td><p>::</p>
<p>farbe : (rot, blau, gruen);</p></td>
<td><p>::</p>
<p>farbe varchar(255) NULL</p></td>
<td><p>Default-Abbilung. Der XTF-Code (der Code wie er in der XTF-Transferdatei steht) wird als Aufzählwert in der Datenbank verwendet. Im Beispiel also: rot, blau oder gruen</p></td>
</tr>
<tr class="even">
<td><p>2</p></td>
<td><p>::</p>
<p>farbe : (rot, blau, gruen);</p></td>
<td><p>::</p>
<p>farbe integer NULL</p></td>
<td><p>Abbilung mit der Option <code>--createEnumColAsItfCode</code>. Der ITF-Code (der Code wie er in der ITF-Transferdatei steht) wird als Aufzählwert in der Datenbank verwendet. Im Beispiel also: 0, 1 oder 2. Diese Option ist nur zulässig, wenn im Modell keine Erweiterungen von Aufzählungen vorkommen.</p></td>
</tr>
<tr class="odd">
<td><p>3</p></td>
<td><p>::</p>
<p>farbe : (rot, blau, gruen);</p></td>
<td><p>::</p>
<p>farbe varchar(255) NULL, farbe_txt varchar(255) NULL</p></td>
<td><p>Abbilung mit der Option <code>--createEnumTxtCol</code>. Es wird eine zusätzliche Spalte mit dem Attributnamen+<code>_txt</code> erstellt (Im Besipiel <code>art_txt</code>). Die zusätzliche Spalte kann einen beliebigen Wert enthalten, der als Anzeigetext gedacht ist. Beim Import wird die Spalte mit dem XTF-Code befüllt. Die Option kann bei Variante 1 oder 2 benutzt werden.</p></td>
</tr>
<tr class="even">
<td><p>4</p></td>
<td><p>::</p>
<p>DOMAIN Farbe : (rot, blau, gruen);</p></td>
<td><p>::</p>
<p>CREATE TABLE Farbe ( itfCode integer PRIMARY KEY, iliCode varchar(1024) NOT NULL, seq integer NULL, dispName varchar(250) NOT NULL, inactive boolean NOT NULL );</p></td>
<td><p>Abbildung mit der Option <code>--createEnumTabs</code>. Es wird pro Aufzählungsdefinition eine Tabelle mit den einzelnen Aufzählwerten erstellt.</p>
<p>itfCode ist der ITF-Code des Aufzählwertes.</p>
<p>iliCode ist der qualifizierte Elementnamen (=XTF-Code) des Aufzählwertes.</p>
<p>seq Definiert die Reihenfolge der Aufzählelemente.</p>
<p>dispName definiert den Anzeigetext für das Aufzählelement. Beim Import wird die Spalte mit dem XTF-Code befüllt.</p>
<p>inactive TRUE um einen Aufzählwert für die Erfassung auszublenden, ohne dass er gelöscht werden muss. Wird beim Import mit FALSE befüllt.</p></td>
</tr>
<tr class="odd">
<td><p>5</p></td>
<td><p>::</p>
<p>DOMAIN Farbe : (rot, blau, gruen);</p></td>
<td><p>::</p>
<p>CREATE TABLE T_ILI2DB_ENUM ( thisClass varchar(1024) NOT NULL, baseClass varchar(1024) NOT NULL, itfCode integer NOT NULL, iliCode varchar(1024) NOT NULL, seq integer NULL, dispName varchar(250) NOT NULL, inactive boolean NOT NULL );</p></td>
<td><p>Abbildung mit der Option <code>--createSingleEnumTab</code>. Es wird eine einzige Tabelle für die Aufzählwerte aller Aufzählungen erstellt.</p>
<p>thisClass ist der qualifizierte Namen der Aufzählungsdefinition.</p>
<p>baseClass ist der qualifizierte Namen der Basis-Aufzählungsdefinition</p>
<p>itfCode ist der ITF-Code des Aufzählwertes.</p>
<p>iliCode ist der qualifizierte Elementnamen (=XTF-Code) des Aufzählwertes.</p>
<p>seq Definiert die Reihenfolge der Aufzählelemente.</p>
<p>dispName definiert den Anzeigetext für das Aufzählelement. Beim Import wird die Spalte mit dem XTF-Code befüllt.</p>
<p>inactive TRUE um einen Aufzählwert für die Erfassung auszublenden, ohne dass er gelöscht werden muss. Wird beim Import mit FALSE befüllt.</p></td>
</tr>
</tbody>
</table>

Metadaten <sub>~<sub>~</sub></sub>~

<table>
<colgroup>
<col width="11%" />
<col width="88%" />
</colgroup>
<thead>
<tr class="header">
<th>Tabelle</th>
<th>Beschreibung</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>t_ili2db_attrname</p></td>
<td><p>Abbildung von Attributnamen</p></td>
</tr>
<tr class="even">
<td><p>t_ili2db_basket</p></td>
<td><p>In der Datenbank vorhandene Baskets. Wird benötigt, wenn die Option --createBasketCol verwendet wird.</p></td>
</tr>
<tr class="odd">
<td><p>t_ili2db_classname</p></td>
<td><p>Abbildung der qualifizierten Interlis Klassennamen auf Sql-Namen. Nicht aus jedem Eintrag gibt es eine Datenbank-Tabelle, je nach Abbildungsart der Interlis-Klasse wird der Sql-Name nur als Inhalt der Spalte t_type verwendet.</p></td>
</tr>
<tr class="even">
<td><p>t_ili2db_dataset</p></td>
<td><p>In der Datenbank vorhandene Datensätze (Sammlung von Baskets). Wird benötigt, wenn die Option --createBasketCol verwendet wird.</p></td>
</tr>
<tr class="odd">
<td><p>t_ili2db_import</p></td>
<td><p>Nicht mehr verwenden, wird entfernt. Wird beim Export nicht benötigt.</p></td>
</tr>
<tr class="even">
<td><p>t_ili2db_import_basket</p></td>
<td><p>Nicht mehr verwenden, wird entfernt. Wird beim Export nicht benötigt.</p></td>
</tr>
<tr class="odd">
<td><p>t_ili2db_import_object</p></td>
<td><p>Nicht mehr verwenden, wird entfernt. Wird beim Export nicht benötigt.</p></td>
</tr>
<tr class="even">
<td><p>t_ili2db_inheritance</p></td>
<td><p>Abbildung der Interlis Klassen Vererbungshierarchie (in der Tabellen sind die qualifizierten Interlis Klassennamen). Wird beim Export nicht benötigt.</p></td>
</tr>
<tr class="odd">
<td><p>t_ili2db_enum</p></td>
<td><p>Aufzählwerte, falls die Option --createSingleEnumTab verwendet wird. Wird beim Export nicht benötigt.</p></td>
</tr>
<tr class="even">
<td><p>t_ili2db_model</p></td>
<td><p>Modelle, die beim Import benötigt wurden (so dass der Export mit denselben Modellen erfolgen kann).</p></td>
</tr>
<tr class="odd">
<td><p>t_ili2db_settings</p></td>
<td><p>Programmeinstellungen für ili2db</p></td>
</tr>
<tr class="even">
<td><p>t_ili2db_trafo</p></td>
<td><p>Konfiguration der semantischen Abbildung (insb. der Vererbung)</p></td>
</tr>
<tr class="odd">
<td><p>t_key_object</p></td>
<td><p>Hilfstabelle für den ID-Generator. Wird beim Export nicht benötigt.</p></td>
</tr>
</tbody>
</table>

TODO

Namenskonvention ~<sub><sub><sub><sub><sub><sub><sub>~</sub></sub></sub></sub></sub></sub></sub>

Die Abbildung der Klassennamen in Tabellennamen erfolgt nach drei möglichen Strategien:

Unqualifiziert Es wird der Klassennamen verwendet (ohne voranstellen von Topic- und/oder Model-Namen). Falls der Name schon benutzt ist, wird der voll qualifizierte Name verwendet.

Mit Topic qualifiziert Dem unqualifizierten Klassennamen wird der Topic-Name vorangestellt. Falls der Name schon benutzt ist, wird der voll qualifizierte Name verwendet.

Voll qualifiziert Der Tabellenname wird aus Model-, Topic- und Klassenname zusammengesetzt.

Falls der Tabellenname zu lang ist, wird er gekürzt, in dem die Vokale entfernt werden (ausser die ersten beiden und letzten beiden Buchstaben). Falls er danach immer noch zu lang ist, werden in der der Mitte des Namens Buchstaben entfernt.

Falls der Tabellenname nun einem SQL-Schlüsselwort entspricht, wird er um ein führeneds ‚a‘ ergänzt.

Falls der Tabellenname nun nicht eindeutig ist, wird er um eine Ziffer ergänzt: ‚0‘, ‚1‘, usw. bis er eindeutig ist.

Die automatische Namensabbildung kann übersteuert werden, indem vor dem ersten Import entsprechende Einträge in der Tabelle t\_ili2db\_classname gemacht werden.

.. \[1\] GML 3.2; die verwendeten Kodierungsregeln entsprechen eCH-0118-1.0

.. \[2\] Der SQL-Name ergibt sich aus den Namenskonventionen. Die konkrete Übersetzung ist in der Tabelle T\_ILI2DB\_CLASSNAME hinterlegt.

.. \[3\] Der SQL-Name ergibt sich aus den Namenskonventionen. Die konkrete Übersetzung ist in der Tabelle T\_ILI2DB\_ATTRNAME hinterlegt.

.. \[4\] Der SQL-Name ergibt sich aus den Namenskonventionen. Die konkrete Übersetzung ist in der Tabelle T\_ILI2DB\_CLASSNAME hinterlegt.
