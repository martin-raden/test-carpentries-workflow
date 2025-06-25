---
title: "Datenverarbeitung"
teaching: 30
exercises: 10
---




:::::::::::::::::::::::::::::::::::::: questions

- Wie baue ich komplexe Workflows mit pipes?
- Wie transformiere ich Tabellen?
- Wie bearbeite ich Text?
- Wie führe ich mehrere Datensätze zusammen?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Überblick über die Möglichkeiten der Datenverarbeitung mit `tidyverse` Paketen

::::::::::::::::::::::::::::::::::::::::::::::::



## Workflows mittels Piping

In vielen Datenverarbeitungssprachen ist der Pipe-Operator ein nützliches Werkzeug, um Arbeitsschritte zu verketten, sodass das Abspeichern von Zwischenergebnisse in Variablen vermieden werden kann.

Seit R 4.1.0 ist der Pipe-Operator `|>` standardmäßig in R enthalten.
Alternativ bietet das `magrittr` package den Pipe-Operator `%>%`, welcher eine etwas umfangreichere Funktionalität bietet.
Wer sich für die Unterschiede interessiert, kann diese im  [Blogbetrag dazu](https://www.tidyverse.org/blog/2023/04/base-vs-magrittr-pipe/) nachlesen.
Für die meisten Anwendungen ist der Standard-Pipe-Operator `|>` ausreichend und äquivalent zu `%>%` verwendbar.

Der Pipe-Operator ist ein nützliches Werkzeug, um den Code lesbarer zu machen und die Arbeitsschritte in einer logischen Reihenfolge zu verketten.
Hierbei wird das Ergebnis des vorherigen Arbeitsschrittes als erstes Argument des nächsten Arbeitsschrittes verwendet.
Da das erste Argument von (den meisten) Funktionen der Datensatz ist, ermöglicht dies die Verknüpfung von Arbeitsschritten zu einem Workflow ohne Zwischenergebnisse speichern zu müssen oder Funktionsaufrufe zu verschachteln.

Zum Beispiel:


``` r
# einfaches Beispiel zur Verarbeitung eines Vektors mit base R Funktionen

# verschachtelte Funktionsaufrufe
# entspricht der Leserichtung von rechts nach links
# und einer Ergebnisbeschreibung ala "Wurzel der Summe ohne NA von 1, 2, 3, NA"
sqrt( sum( c(1, 2, 3, NA), na.rm=TRUE ) )

# mit Pipe-Operator
# entspricht der Leserichtung von links nach rechts
# und einer Beschreibung der Arbeitsschritte in ihrer Reihenfolge
# ala "nehme 1, 2, 3, NA, summiere ohne NA und davon die Wurzel"
c(1, 2, 3, NA) %>% 
  sum(na.rm = TRUE) %>% 
  sqrt()

# ok, hier nicht zwingend nötig, aber zur Veranschlaulichung des Prinzips.. ;)
```

Pipe-basierte workflows sind ...

- intuitiv lesbar, da sie der normalen Leserichtung entsprechen.
- einfacher zu schreiben, da die umschließende Klammerung von Funktionsaufrufen entfällt. Funktionsargumente sind direkt den jeweiligen Funktionen zuordenbar.
- einfach zu erweitern, da neue Arbeitsschritte einfach an das Ende der Kette angehängt werden können (z.B. `write_csv("bla.csv")` oder bestehende Arbeitsschritte ausgetauscht werden können.
- reduzieren oftmals die Anzahl der temporären Variablen, die im globalen Workspace herumliegen und die Lesbarkeit des Codes beeinträchtigen.
- einfach zu debuggen, da Zwischenergebnisse einfach ausgegeben werden können (einfach `view()` oder `print()` an das Ende der entsprechenden Zeile anhängen).
- einfach wiederzuverwenden, da der gesamte Workflow in einer Zeile zusammengefasst ist und immer als Block aufgerufen wird. Dadurch können keine Zwischenschritte vergessen werden und der Workflow ist immer konsistent.

::: callout

## Mehrzeilige R Kommandos

**Wichtig:** R Kommandos können über mehrere Zeilen verteilt werden, wie im obigen Beispiel zu sehen ist, was die Übersichtlichkeit des Codes erhöht.

Hierzu wird z.B. der Pipe-Operator `|>` ans Ende der Zeile geschrieben und der nächste Arbeitsschritt in der nächsten Zeile fortgesetzt.
Gleiches gilt für jeden Operator (`+`,  `==`, `&`, ..) sowie unvollständige Funktionsaufrufe, bei denen die Klammerung noch nicht geschlossen ist (d.h. schließende Klammer wird in der nächsten oder einer späteren Zeile geschrieben).

:::::::::::


## Tabellen verarbeiten

Die Transformation von Daten erfolgt i.d.R. mittels des `dplyr` packages.

*Grundlegend gilt: das erste Argument einer Funktion ist immer der Datensatz!*

Innerhalb von `dplyr` Funktionen können *Spaltennamen ohne Anführungzeichen* (quoting) verwendet werden.

Basistransformationen sind:

-   Filtern von Zeilen mit gegebenen Kriterien (formulieren was man BEHALTEN will!)
    -   `filter(storms, year == 2020)` = alle Sturmdaten aus dem Jahr 2020
    -   `filter(storms, year == 2020 & month == 6)` = alle Sturmdaten aus
        dem Juni 2020
    -  `filter(storms, !is.na(category))` = alle Sturmdaten, bei denen die Kategorie bekannt ist
- Konkrete Zeilenauswahl (via Index oder Anzahl)
    -   `slice(storms, 1, 3, 5)` = die Zeilen 1, 3 und 5
    -   `slice_tail(storms, n=10)` = die 10 letzten Zeilen
    -   `slice_max(storms, pressure)` = die Zeile mit dem höchsten Wert in der Spalte `pressure`
-   Sortieren von Zeilen
    -   `arrange(storms, year)` = Sturmdaten nach Jahr aufsteigend sortieren
    -   `arrange(storms, year, desc(month))` = Sturmdaten aufsteigend nach Jahr sortieren und innerhalb eines Jahres absteigend nach Monat
-   Duplikate entfernen
    -  `distinct(storms)` = alle Zeilen mit identischen Werten in allen Spalten entfernen
    -   `distinct(storms, year, month, day)` = alle Zeilen mit gleichen Werten in den Spalten `year`, `month` und `day` entfernen (reduziert die Spalten auf die Ausgewählten)
    -  `distinct(storms, year, month, day, .keep_all = TRUE)` = alle Zeilen mit gleichen Werten in den Spalten `year`, `month` und `day` entfernen, aber *alle Spalten behalten*
-   Auswählen/Entfernen von Spalten
    -   `select(storms, name, year, month, day)` = nur Spalten mit Zeitinformation und Namen der Stürme behalten
    -   `select(storms, -year, -month, -day)` = Spalte mit Zeitinformation entfernen
-   Umbenennen von Spalten
    -   `rename(storms, sturmname = name)` = Spalte `name` in `sturmname` umbenennen
-   Zusammenfassen von Daten: nur eine Zeile mit aggregierten Informationen (z.B. Mittelwert, Summe, Anzahl, etc.) pro Gruppe
    -   `summarize(storms, max_wind = max(wind), num_datasets = n())` = maximale Windgeschwindigkeit und Anzahl der Datensätze (Zeilen)
-   Gruppierung von Daten = "Zerlegung" des Datensatzes in Teiltabellen, für die anschliessende Arbeitsschritte unabängig voneinander durchgeführt werden. Wird i.d.R. verwendet, wenn die Aufgabe "pro ..." oder "für jede ..." lautet.
    -   `group_by(storms, year)` = Gruppierung der Sturmdaten nach Jahr (aber noch keine Aggregation!)
    -   `group_by(storms, year) |>  summarize(max_wind = max(wind))` = maximale Windgeschwindigkeit *pro Jahr* (eine "summarize" Zeile pro Teiltabelle = Gruppe = Jahr)
    -  `group_by(storms, year) |> filter(wind == max(wind)) |> ungroup()` = alle Sturmdaten, bei denen die maximale Windgeschwindigkeit *pro Jahr* erreicht wurde (keine Zusammenfassung!)
    - Grouping ist ein extrem mächtiges Werkzeug, das in vielen Situationen verwendet wird, um Daten zu transformieren. Allerdings braucht es etwas Übung, um zu verstehen, wie es funktioniert.

- Spalten hinzufügen/ausrechnen oder bestehende Spalten verändern (z.B. Einheiten umrechnen)
    -   `mutate(storms, wind_kmh = wind * 1.60934)` = Windgeschwindigkeit in km/h berechnen und als *neue Spalte hinzufügen*
    -   `mutate(storms, wind = wind * 1.60934)` = Windgeschwindigkeit in km/h umrechnen und *bestehende Spalte überschreiben*
    -  `mutate(storms, wind_kmh = wind * 1.60934, wind_kmh_rounded = round(wind_kmh, 1))` = es können auch mehrere Spalten auf einmal berechnet werden und dabei direkt neue angelegte Spalten in anschliessenden Formeln verwendet werden (hier Runden auf eine Nachkommastelle)


:::::::::::::::::::: challenge

## Stürme vor 1980

*Erstelle eine Tabelle, welche für jeden Sturm vor 1980 neben dessen Namen nur das Jahr und dessen Status beinhaltet und nach Jahr und Status sortiert ist.*

:::::::::::: solution

# Hinweise

Verwenden sie eine Pipe die folgende Funktionen verbindet
- `filter()`
- `select()`
- `distinct()`
- `arrange()`

:::::::::::::::::::::

:::::::::::: solution

# Lösung


``` r
# Ausgangsdatensatz = Beginn der Pipe
storms |>
  # Zeilen filtern
  filter(year < 1980) |>
  # Spalten auswählen
  select(name, year, status) |>
  # doppelte Zeilen entfernen
  distinct() |>
  # sortieren
  arrange(year, status)
```

``` output
# A tibble: 129 × 3
   name      year status       
   <chr>    <dbl> <fct>        
 1 Amy       1975 extratropical
 2 Blanche   1975 extratropical
 3 Doris     1975 extratropical
 4 Eloise    1975 extratropical
 5 Gladys    1975 extratropical
 6 Hallie    1975 extratropical
 7 Blanche   1975 hurricane    
 8 Caroline  1975 hurricane    
 9 Doris     1975 hurricane    
10 Eloise    1975 hurricane    
# ℹ 119 more rows
```
:::::::::::::::::::::

::::::::::::::::::::::::::::::


## Gruppieren und Aggregieren

Eine der wichtigsten Funktionen in `dplyr` ist das Gruppieren von Daten und das Aggregieren von Werten innerhalb dieser Gruppen.
Dies wird in der Regel mit den Funktionen `group_by()` und `summarize()` durchgeführt.

`group_by()` teilt den Datensatz in Gruppen (imaginäre Teiltabellen) auf, basierend auf den Werten in einer oder mehreren Spalten.
Im Anschluß wird, vereinfacht gesagt, für jede Gruppe eine separate Berechnung durchgeführt.

Die Funktion `summarize()` ist die am häufigsten verwendete Funktion, um Werte innerhalb dieser Gruppen zu aggregieren.

Ein einfaches Beispiel:


``` r
storms |>
  # Tabelle nach Jahr gruppieren
  group_by(year) |>
  # maximale Windgeschwindigkeit pro Jahr berechnen
  summarize(max_wind = max(wind))
```

``` output
# A tibble: 48 × 2
    year max_wind
   <dbl>    <int>
 1  1975      120
 2  1976      105
 3  1977      150
 4  1978      120
 5  1979      150
 6  1980      165
 7  1981      115
 8  1982      115
 9  1983      100
10  1984      115
# ℹ 38 more rows
```

In diesem Beispiel wird die Tabelle `storms` nach dem Jahr gruppiert und für jede Gruppe (d.h. jedes Jahr) die maximale Windgeschwindigkeit berechnet.
Das Ergebnis ist eine Tabelle mit zwei Spalten: `year` und `max_wind`.

::: callout

## Achtung

Zu beachten ist, dass Gruppierungen in `dplyr` nur virtuell sind und nicht zu einer physischen Aufteilung des Datensatzes führen.
Allerdings wird diese Gruppierung aufrecht erhalten, bis sie explizit aufgehoben wird (z.B. durch `ungroup()`) oder dies implizit durch eine andere Funktion geschieht (z.B. "schließt" `summarize()` die letzte Gruppierung).

:::::::::::


:::::::::::::::::::: challenge

## Gruppiertes Filtern

*Erstelle eine Tabelle, welche für jeden Sturmstatus das Jahr und den Namen des letzten Sturms auflistet.*


:::::::::::: solution

### Erwartete Ausgabe


``` output
# A tibble: 9 × 3
  status                  year name  
  <fct>                  <dbl> <chr> 
1 tropical wave           2018 Kirk  
2 subtropical depression  2020 Dolly 
3 disturbance             2022 Julia 
4 extratropical           2022 Martin
5 subtropical storm       2022 Nicole
6 hurricane               2022 Nicole
7 tropical storm          2022 Nicole
8 tropical depression     2022 Nicole
9 other low               2022 Nicole
```

:::::::::::::::::::::

:::::::::::: solution

## Hinweise

- Gruppieren sie die Tabellendaten nach `status`, um für jeden Sturmtyp eine "Teiltabelle" zu erhalten
- Um den letzten Sturm zu finden
  - Möglichkeit 1: neue Spalte mit Zeitinformation zusammensetzen und damit die Zeile mit maximalen Datum (pro Teiltabelle) extrahieren
  - Möglichkeit 2: Teiltabellen bzgl. Zeitspalten sortieren und erste bzw. letzte Zeile extrahieren

:::::::::::::::::::::

:::::::::::: solution

## Lösungen 

### Alternative 1


``` r
storms |>
  # decompose table by storm status
  group_by(status) |>
  # encode time information of each entry in a single column
  mutate(date = parse_date_time(str_c(year,month,day,hour,sep="-"), "%Y-%m-%d-%H")) |>
  
  # filter for the latest date
  filter(date == max(date)) |>
# ALTERNATIVELY cut out row with latest date
  # slice_max(date, n=1) |>
  
  # rejoin table information and undo grouping
  ungroup() |> 
  select(status, year, name)
```

### Alternative 2


``` r
storms |>
  # decompose table by storm status
  group_by(status) |>
  
  # sort ascending by date (hierarchical sorting)
  arrange(year,month,day,hour) |>
  # pick last row w.r.t. sorting
  slice_tail(n=1) |>
# ALTERNATIVELY pick row with last index
  # slice( n() ) |> 
  
# OR do both (sort+pick) directly with slice_max
  # slice_max( tibble(year,month,day,hour), n=1, with_ties = F ) |> 
  
  # rejoin table information and undo grouping
  ungroup() |> 
  select(status, year, name) 
```
:::::::::::::::::::::

::::::::::::::::::::::::::::::

Eine Zusammenfassung der wichtigsten Funktionen in `dplyr` finden sich in dessen Cheat Sheet.


[![dplry cheatsheet](https://raw.githubusercontent.com/rstudio/cheatsheets/main/pngs/thumbnails/data-transformation-cheatsheet-thumbs.png){width="100%" alt="CLICK TO ENLARGE: cheat sheet for dplyr ackage"}](https://raw.githubusercontent.com/rstudio/cheatsheets/main/data-transformation.pdf)



## Tabellengestalt ändern

Die "tidy" Form ist eine spezielle Form von tabellarischen Daten, die für viele Datenverarbeitungsschritte geeignet ist. In der "tidy" Form sind die Daten so organisiert, dass jede Beobachtung (z.B. Messung) in einer Zeile notiert ist und jede Variable (also z.B. Beschreibungungen und Messwerte) eine Spalte definieren.
Diese Art der Tabellenform wird auch "long table" oder "schmal" genannt, weil die Daten in einer langen, schmalen Tabelle organisiert sind.

Allerdings werden Rohdaten häufig in einer "ungünstigen" Form vorliegen, die für die weitere Verarbeitung nicht optimal ist. 
Manuelle Datenerfassung erfolgt oft in grafischen Oberflächen wie MS Excel, worin die Daten i.d.R. in einer  "breiten" oder "wide table" Form gespeichert, in der eine Variable (z.B. Messwert) in mehreren Spalten gespeichert ist. 
In der "breiten" Form sind die Daten so organisiert, dass Beobachtungen (und ihre Messwerte) in Spalten gruppiert sind, während die Variablen in den Zeilen stehen.

Hier ein Beispiel aus dem Tutorial [Introduction to R/tidyverse for Exploratory Data Analysis](https://tavareshugo.github.io/r-intro-tidyverse-gapminder/)

![](https://github.com/tavareshugo/r-intro-tidyverse-gapminder/blob/e08446803314d9c8d59f309e3dee7b1cdd9a3158/fig/07-tidyr_pivot.png?raw=true){alt="wide and long table examples and respective pivoting calls" width="500px"}

Das obige Beispiel zeigt eine "breite" Tabelle (rechts) und eine "lange" Tabelle (links) mit den gleichen Daten.
Die Transformation zwischen beiden Formaten kann durch die Funktionen `pivot_longer()` und `pivot_wider()` erreicht werden.
Beachten sie im Beispiel, dass die Spaltennamen des wide table Formats in beide Transformationsrichtungen verarbeitet werden können.

::: callout

## Vor- und Nachteile

Während das wide table Format kompakter und damit ggf. besser zur Datenerfassung und -übersicht geeignet ist, ist das long table Format besser für die Datenanalyse und -visualisierung geeignet.
In letzterem liegt Information redundant vor, da in jeder Zeile die komplette Information einer Beobachtung vorliegen muss.

:::::::::::

Die tidyverse Bibliothek `tidyr` bietet Funktionen, um Daten zwischen diesen beiden Formen zu transformieren. Dies wird auch als "reshaping" oder "pivotieren" bezeichnet.
Hiermit können Spalten in Zeilen und umgekehrt umgeformt werden. 

Das Beispiel hierfür ist etwas länger, um für Anschauungszwecke die Datentabelle erst etwas einzukürzen.


``` r
storms |>
  filter(name == "Arthur" & year == 2020) |> # speziellen Sturm auswählen
  select(name, year, month, day, wind, pressure) |> # (zur Vereinfachung) nur spezifische Spalten
  # Verteile Wind und Druck in separate Zeilen mit entsprechenden Labels in einer neuen Spalte "measure"
  pivot_longer(cols = c(wind, pressure), names_to = "measure", values_to = "value") |>
  slice_head(n=4) # nur die ersten 4 Zeilen anzeigen
```

``` output
# A tibble: 4 × 6
  name    year month   day measure  value
  <chr>  <dbl> <dbl> <int> <chr>    <int>
1 Arthur  2020     5    16 wind        30
2 Arthur  2020     5    16 pressure  1008
3 Arthur  2020     5    17 wind        35
4 Arthur  2020     5    17 pressure  1006
```

Details und weitere Funktionen und Beispiele sind im Cheat Sheets des `tidyr` Paketes übersichtlich zusammengefasst.

[![tidyr cheatsheet](https://raw.githubusercontent.com/rstudio/cheatsheets/master/pngs/thumbnails/tidyr-thumbs.png){width="100%" alt="CLICK TO ENLARGE: cheat sheet for tidyr ackage"}](https://raw.githubusercontent.com/rstudio/cheatsheets/main/tidyr.pdf)


## Textdaten verändern und verwenden

Um Textdaten (*Strings*) zu verändern, gibt es verschiedene Funktionen des `stringr` Paketes, die in Kombination mit `mutate()` oder anderen Funktionen verwendet werden können.
Die meisten Funktionen des `stringr` Paketes sind sehr intuitiv und haben eine ähnliche Syntax wie die Funktionen des `dplyr` Paketes.
Zudem sind viele mit regulären Ausdrücken verwendbar, um Textdaten aufgrund von allgemeineren Textmustern zu erkennen und zu verändern.
Alle Funktionen des `stringr` Paketes beginnen mit `str_` und sind in der [Dokumentation](https://stringr.tidyverse.org/reference/index.html) aufgeführt.
Einige wichtige sind:

- `str_c()`: Verkettet mehrere Strings zu einem, analog to `paste0()`
- `str_detect()`: Prüft, ob ein String einen bestimmten Textteil enthält
- `str_replace()`: Ersetzt einen Teil eines Strings durch einen anderen
- `str_to_lower()`/`str_to_upper`: Wandelt alle Buchstaben in Klein-/Grossbuchstaben um
- `str_sub()`: Extrahiert einen Teil eines Strings
- `str_trim()`: Entfernt Leerzeichen am Anfang und Ende eines Strings
- `str_extract()`: Extrahiert einen Teil eines Strings, der einem bestimmten Muster entspricht
- `str_remove()`: Entfernt einen Teil eines Strings, der einem bestimmten Muster entspricht
- `str_split()`: Teilt einen String an einem bestimmten Trennzeichen auf

Die meisten Funktionen liefern nur einen Treffer oder verändern nur den ersten Treffer.
Daher gibt es von vielen Funktionen Varianten, die alle Treffer finden und/oder verarbeiten, z.B. `str_detect_all()`, `str_replace_all()`, `str_extract_all()`.

Die Funktionen sind im [`stringr` Cheat Sheet](https://raw.githubusercontent.com/rstudio/cheatsheets/master/strings.pdf) zusammengefasst.

[![stringr cheatsheet](https://raw.githubusercontent.com/rstudio/cheatsheets/main/pngs/thumbnails/strings-cheatsheet-thumbs.png){width="100%" alt="CLICK TO ENLARGE: cheat sheet for stringr ackage"}](https://raw.githubusercontent.com/rstudio/cheatsheets/main/strings.pdf)


Beispiele für deren Verwendung mit `mutate()` und Co. sind:


``` r
# Beispiel: Grossbuchstaben in Spalte "name" in Kleinbuchstaben umwandeln
# (und nur eindeutige Einträge in Spalte "name" anzeigen)
mutate(storms, name = str_to_lower(name)) |> distinct(name)
# Beispiel: Jahrhundert aus Spalte "year" extrahieren
mutate(storms, century = str_sub(year, 1, 2)) |> distinct(name, century)
 # oder via regulärem Ausdruck
mutate(storms, century = str_extract(year, "^\\d{2}"))
mutate(storms, century = str_remove(year, "..$"))
# Beispiel: Nur Stürme mit "storm" im Status behalten
filter(storms, str_detect(status, "storm")) |> distinct(name)
```

Beachte:

1.  wenn die (Daten)Eingabe einer `stringr` Funktion ein *Vektor* ist, wird die Funktion auf jedes Element des Vektors angewendet und gibt einen Vektor mit den Ergebnissen zurück. (= *vektorisierte Prozessierung* = zentrales Verarbeitungsprinzip in R)
2.  wenn die Eingabe *kein* String ist (z.B. eine Zahl), wird die Eingabe in einen String umgewandelt und die Funktion auf den entstandenen String angewendet. (= *automatische Typumwandlung*, sogenanntes *coercion* in R)


## Reguläre Ausdrücke

Für komplexe Textverarbeitung werden oft reguläre Ausdrücke verwendet.
Diese erlauben es, Textmuster zu definieren, die in einem Text gesucht, ersetzt oder extrahiert werden können.

Für eine Einführung in reguläre Ausdrücke siehe [RegexOne Tutorial](https://www.regexone.com/) oder [RStudio RegEx Cheat Sheet (pdf)](https://rstudio.github.io/cheatsheets/regex.pdf).


:::::::::::::::::::: challenge

## Reguläre Ausdrücke

Betrachten sie noch einmal die beiden `mutate()` Beispiele von oben, in denen das Jahrhundert aus der Spalte "year" mit Hilfe von regulären Ausdrücken extrahiert wird.

*Was genau definieren die beiden regulären Ausdrücke `^\\d{2}` und `..$`?*


:::::::: solution

### Lösung

- `^\\d{2}` = "die ersten zwei (`{2}`) Ziffern (`\\d`) am Textanfang (`^`)"
  - hier genutzt um das Jahrhundert mit `str_extract()` zu extrahieren
- `..$` = "die letzten zwei beliebigen Zeichen (`..`) am Ende des Textes (`$`)"
  - hier genutzt um die letzten zwei Zeichen (Jahrzehnt) mit `str_remove()` zu entfernen

:::::::::::::::::

::::::::::::::::::::::::::::::




## Daten zusammenführen

Oftmals liegen daten in mehreren Tabellen vor, die zusammengeführt werden müssen, um die Daten zu analysieren.
Zum Beispiel können Daten in einer Tabelle die Anzahl der Sturmtage pro Jahr enthalten und in einer anderen Tabelle die Kosten für Sturmschäden pro Jahr.


``` r
# Datensatz mit Sturmschäden pro Jahr (zufällige Werte) für 2015-2024
costs <-
  tibble(year = 2015:2024, # Jahresbereich festlegen
         costs = runif(length(year), 1e6, 1e8)) # Zufallsdaten gleichverteilt erzeugen

# Anzahl der Sturmtage pro Jahr
stormyDays <-
  storms |>
  # Datensatz in Einzeljahre aufteilen
  group_by(year) |>
  # nur eine Zeile pro Monat/Jahr Kombination (pro Jahr) behalten
  distinct(month, day) |>
  # zählen, wie viele Zeilen pro Jahr noch vorhanden sind
  count() |>
  ungroup()


# Beispiel (1): Sturmtaginformation (links) mit Kosteninformation (rechts) ergänzen
# BEACHTE: für Jahre ohne Eintrag in `costs` wird `NA` eingetragen !
left_join(stormyDays, costs, by = "year") |>
  # nur die ersten und letzten 3 Zeilen anzeigen
  slice( c(1:3, (n()-2):n()) )

# Beispiel (2): nur Datensätze für die beides, d.h. Sturmtage als auch Kosten, vorhanden ist
inner_join(stormyDays, costs, by = "year")
```


:::::::::::::::::::: challenge

## Spaltennamen

Studieren sie die [Hilfeseite der join Funktionen](https://dplyr.tidyverse.org/reference/mutate-joins.html) und finden sie heraus, *wie sie die Spaltennamen der beiden Tabellen angeben können*, um die Daten zu verbinden, wenn die gemeinsamen Daten in unterschiedlich benannten Spalten stehen.

:::::::: solution

## Lösung

Das mapping der Spalten erfolgt über einen *benannten Vektor*, d.h. die Spaltennamen der linken Tabelle werden als Elementnamen für die entsprechenden Spaltennamen der rechten Tabelle verwendet:


``` r
stormyDays |> 
  rename( Jahr = year ) |> # Jahresspalte exemplarisch umbenannt
  filter( Jahr >= 2013 ) |> 
  # geänderte Sturmtaginformation (links) aus der pipe
  # mit Kosteninformation (rechts) aus expliziter Tabelle "costs" ergänzen
  left_join( costs, 
      # Explizites mapping von costs$year auf die "Jahr" Spalte der linken Tabelle
      # Beachten sie das quoting der Spaltennamen!
             by = c(Jahr = "year")
      # Alternativ kann das mapping auch über eine Funktion erfolgen:
      # Hier sind keine Anführungszeichen notwendig:
      #  by = join_by(Jahr = year)
      ) 
```

:::::::::::::::::

::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints

- Speichern sie Daten nur in Variablen zwischen, wenn sie diese Daten mehrfach benötigen.
- Verwenden sie Pipes (`|>`) um Daten durch eine Reihe von Transformationen zu leiten.
- Versuchen sie die Datenverarbeitung in kleine, leicht verständliche Schritte zu unterteilen.
- Vermeiden sie unnötige Schleifen und Schachtelungen, das meiste lässt sich mit Grouping, vektorisierten Operationen und Joins kompakter und eleganter lösen.
- Auch explizite Elementzugriffe (z.B. `df$col`) und -operationen können i.d.R. effizient durch `dplyr` Funktionen ersetzt werden.
- Zusammenfassungen der Pakete:
  - [`dplry` Cheat Sheet](https://raw.githubusercontent.com/rstudio/cheatsheets/main/data-transformation.pdf)
  - [`tidyr` Cheat Sheet](https://raw.githubusercontent.com/rstudio/cheatsheets/main/tidyr.pdf)
  - [`stringr` Cheat Sheet](https://raw.githubusercontent.com/rstudio/cheatsheets/main/strings.pdf)




::::::::::::::::::::::::::::::::::::::::::::::::


-----------------------------------------------

Dieses Dokument wurde mit Unterstützung von GitHub Copilot erstellt, einem KI-gestützten Autocompletion-Tool, das auf der OpenAI GPT-3-Technologie basiert.

