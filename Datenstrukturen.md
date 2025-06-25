---
title: "Datenstrukturen"
teaching: 15
exercises: 5
---



:::::::::::::::::::::::::::::::::::::: questions

- Welche "Datenarten" gibt es?
- Wie organisiere ich tabellarische Daten?
- Was ist "tidy"?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Standarddatentypen in R
- `data.frame` und `tibble`
- "tidy" Konzept

::::::::::::::::::::::::::::::::::::::::::::::::


Für die Verarbeitung von tabellarischen Daten werden in R i.d.R. `data.frame`s verwendet.
Das `tidyverse` package liefert eine erweiterte Version des `data.frame`s, das `tibble`, welches etwas transparenter in seiner Verwendung ist.
In einem `tibble` (oder `data.frame`) entspricht jede Zeile einem Datensatz (oder einer Beobachtung) und jede Spalte einer Variable (oder einem Merkmal) dieses Datensatzes.
Damit haben alle Daten in der gleichen Spalte den gleichen Datentyp, z.B. "Zahl" (`numeric`), "Kommazahl" (`double`), "Ganzzahl" (`integer`), "Text" (`character`), "Datum" (`date`), etc.
Im Folgenden wird der [Beispieldatensatz `storms`](https://dplyr.tidyverse.org/reference/storms.html) aus dem `dplyr` package gezeigt.


``` r
dplyr::storms
```

``` output
# A tibble: 19,537 × 13
   name   year month   day  hour   lat  long status      category  wind pressure
   <chr> <dbl> <dbl> <int> <dbl> <dbl> <dbl> <fct>          <dbl> <int>    <int>
 1 Amy    1975     6    27     0  27.5 -79   tropical d…       NA    25     1013
 2 Amy    1975     6    27     6  28.5 -79   tropical d…       NA    25     1013
 3 Amy    1975     6    27    12  29.5 -79   tropical d…       NA    25     1013
 4 Amy    1975     6    27    18  30.5 -79   tropical d…       NA    25     1013
 5 Amy    1975     6    28     0  31.5 -78.8 tropical d…       NA    25     1012
 6 Amy    1975     6    28     6  32.4 -78.7 tropical d…       NA    25     1012
 7 Amy    1975     6    28    12  33.3 -78   tropical d…       NA    25     1011
 8 Amy    1975     6    28    18  34   -77   tropical d…       NA    30     1006
 9 Amy    1975     6    29     0  34.4 -75.8 tropical s…       NA    35     1004
10 Amy    1975     6    29     6  34   -74.8 tropical s…       NA    40     1002
# ℹ 19,527 more rows
# ℹ 2 more variables: tropicalstorm_force_diameter <int>,
#   hurricane_force_diameter <int>
```
Bei der Verwendung von `tibble`s sind unter den Spaltennamen die Datentypen der Spalten abgekürzt dargestellt.

`data.frame`s oder `tibble`s können auch aus anderen Datenstrukturen erstellt werden, z.B. aus `vectors`, `lists` oder `matrices`.
Kleinere Datensätze können auch direkt als `tibble` erstellt werden, indem die Daten in die Funktion `tibble()` eingegeben werden.


``` r
marriages <- 
  tibble(
    name = c("Alice", "Bob", "Charlie","Diana"),
    age = c(25, 30, 35, 12),
    married = c(TRUE, FALSE, TRUE, FALSE)
  )
```


Ziel der Datenverarbeitung ist es, die Daten so zu transformieren, dass sie für die Analyse und Visualisierung geeignet sind und ein Format wie oben gezeigt vorweisen.
Wenn das der Fall ist, sprich

- jede Zeile entspricht einem Datensatz und
- jede Spalte entspricht einer Variable,

wird der Datensatz als "tidy" bezeichnet, was im Folgenden visualisiert wird.

![](https://raw.githubusercontent.com/hadley/r4ds/main/images/tidy-1.png){width="40%" alt="tidy data concept"}



:::::::::::::::::::::::::: challenge

## Spaltenzugriff

Betrachten sie den Datensatz `marriages` von oben.

*Wie können Sie nur die Spalte `name` aus dem Datensatz extrahieren?*

:::::::::::::::: solution

## Antwort


``` r
# base R
marriages$name                  # mit "$" ohne Anführungszeichen (Standardfall)
marriages$`name`                # mit "$" und "backtick" quotes für Variablennamen mit Sonderzeichen
marriages[["name"]]             # WICHTIG: Doppelte eckige Klammern und Name in quotes, siehe unten !
marriages[[1]]                  # mit doppelter eckiger Klammer und Spaltenindex (nicht empfohlen, sicherer via Namen)
# tidyverse dplyr package 
dplyr::pull(marriages, name)    # ohne "dplyr::" wenn dplyr geladen ist (library(dplyr))
# oder in einer pipe, was später diskutiert wird
marriages %>% dplyr::pull(name)
```

Achtung: 
- EINFACHE eckige Klammern `marriages["name"]` geben ein `data.frame` zurück (*REDUZIEREN* die aktuelle Datenstruktur), während
- DOPPELTE eckige Klammern `marriages[["name"]]` den `vector` der Spalte `name` zurückgeben (also das benannte Element *EXTRAHIEREN*)!

:::::::::::::::::::::::::

## Datentypen

*Welche Datentypen haben die Spalten des Datensatzes?*

:::::::::::::::: solution

## Antwort

- `name`: `character`
- `age`: `numeric` also generell "Zahlen" 
  - entspricht i.d.R. `double` ist eine Unterart von `numeric`, die auch Kommazahlen enthält.
  - `integer` wäre auch möglich, aber `numeric` ist der Standarddatentyp für Zahlen in R.
  - wenn wir *explizit* Ganzzahlen definieren wollen, müssen diese mit einem "L" versehen werden, z.B. `25L`, oder die Spalte explizit als `integer` definier/umgewandelt werden, z.B. `as.integer(c(25, 30, 35))`.
- `married`: `logical`

:::::::::::::::::::::::::

## Aufbau

*Wieviele Beobachtungen und Variablen hat der Datensatz?*

:::::::::::::::: solution

## Antwort

- Beobachtungen: 4 = Anzahl Zeilen (oder Länge der Spalten)
- Variablen: 3 = Anzahl Spalten

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::

Um zu verdeutlichen, worin der Unterschied zwischen `data.frame` und `tibble` besteht, führen sie folgende Code in einer R-Session aus:



``` r
# Erstellen eines data.frame
data.frame( "das sind daten" = c(1, 2, 3), "das sind auch daten" = c(4, 5, 6) )
# Erstellen eines äquivalenten tibbles
tibble::tibble( "das sind daten" = c(1, 2, 3), "das sind auch daten" = c(4, 5, 6) )
```


:::::::::::::::::::::::::: challenge

## Unterschiede

*Was fällt ihnen auf?* 

:::::::::::::::: solution

## Antwort


``` output
  das.sind.daten das.sind.auch.daten
1              1                   4
2              2                   5
3              3                   6
```

``` output
# A tibble: 3 × 2
  `das sind daten` `das sind auch daten`
             <dbl>                 <dbl>
1                1                     4
2                2                     5
3                3                     6
```


- Spaltennamen in `data.frame` werden automatisch bereinigt, damit sie den normalen Konventionen für Variablennamen entsprechen, z.B. Leerzeichen werden durch Punkte ersetzt.
- `tibble` zeigt die Spaltennamen so an, wie sie eingegeben wurden, und unterstützen auch Sonderzeichen in den Spaltennamen.
  - beachten Sie, dass Sonderzeichen in Spaltennamen in R nicht empfohlen werden, da sie zu Problemen führen können.
  - ausserdem müssen solche Spaltennamen später in der Verwendung in backtick "`" quotes gesetzt werden, siehe oben
- ausserdem (hier nicht zu sehen) zeigt `tibble` nur die ersten 10 Zeilen an, während `data.frame` alle Zeilen anzeigt.
- zudem werden die Datentypen für jede Spalte in `tibble` angezeigt, was bei `data.frame` nicht der Fall ist.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- Spalten in einem `tibble` werden auch Variablen (des Datensatzes) genannt.
  - Eine Spalte ist ein `vector`, d.h. es können nur Werte des gleichen Datentyps enthalten sein.
- Zeilen in einem `tibble` werden auch Beobachtungen (des Datensatzes) genannt.
- Ein Datensatz ist "tidy", 
  - wenn jede Zeile einem Datensatz und jede Spalte einer Variable entspricht. 
  - Vereinfacht: wenn man beim Visualisieren der Daten nur jeweils eine Zeile pro Datenpunkt benötigt und keine doppelt verwendet wird.


::::::::::::::::::::::::::::::::::::::::::::::::


-----------------------------------------------

Dieses Dokument wurde mit Unterstützung von GitHub Copilot erstellt, einem KI-gestützten Autocompletion-Tool, das auf der OpenAI GPT-3-Technologie basiert.

