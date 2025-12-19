# Основы обработки данных с помощью R и Dplyr


## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета dplyr – функции select(), filter(), mutate(), arrange(),
    group_by()

## Исходные данные

1.  Rstudio Desktop;
2.  Интерпретатор языка R 4.1;
3.  Программный пакет `dplyr`.

## План

Проанализировать встроенный в пакет dplyr набор данных starwars с
помощью языка R и ответить на вопросы.

## Решение:

``` r
library(dplyr)
```

    Warning: пакет 'dplyr' был собран под R версии 4.5.2


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

1.  Сколько строк в датафрейме?

<!-- -->

    > starwars %>% nrow()
    [1] 87

1.  Сколько столбцов в датафрейме?

<!-- -->

    starwars %>% ncol()
    [1] 14

1.  Как просмотреть примерный вид датафрейма?

``` r
starwars %>% glimpse()
```

    Rows: 87
    Columns: 14
    $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
    $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
    $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
    $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
    $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
    $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
    $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
    $ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
    $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
    $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
    $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
    $ films      <list> <"A New Hope", "The Empire Strikes Back", "Return of the J…
    $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
    $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…

1.  Сколько уникальных рас персонажей (species) представлено в данных?

``` r
starwars %>% distinct(species) %>% nrow()
```

    [1] 38

1.  Найти самого высокого персонажа.

``` r
starwars %>%
  filter(height == max(height,na.rm = TRUE)) %>%
  pull(name)
```

    [1] "Yarael Poof"

1.  Найти всех персонажей ниже 170.

``` r
select(starwars, name, height) %>% filter(height>170)
```

    # A tibble: 55 × 2
       name              height
       <chr>              <int>
     1 Luke Skywalker       172
     2 Darth Vader          202
     3 Owen Lars            178
     4 Biggs Darklighter    183
     5 Obi-Wan Kenobi       182
     6 Anakin Skywalker     188
     7 Wilhuff Tarkin       180
     8 Chewbacca            228
     9 Han Solo             180
    10 Greedo               173
    # ℹ 45 more rows

1.  Подсчитать ИМТ (индекс массы тела) для всех персонажей.

``` r
mutate(starwars, BMI=mass/((height/100)^2)) %>% select(name, height, mass, BMI)
```

    # A tibble: 87 × 4
       name               height  mass   BMI
       <chr>               <int> <dbl> <dbl>
     1 Luke Skywalker        172    77  26.0
     2 C-3PO                 167    75  26.9
     3 R2-D2                  96    32  34.7
     4 Darth Vader           202   136  33.3
     5 Leia Organa           150    49  21.8
     6 Owen Lars             178   120  37.9
     7 Beru Whitesun Lars    165    75  27.5
     8 R5-D4                  97    32  34.0
     9 Biggs Darklighter     183    84  25.1
    10 Obi-Wan Kenobi        182    77  23.2
    # ℹ 77 more rows

1.  Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по
    отношению массы (mass) к росту (height) персонажей.

``` r
starwars %>% mutate(stretch_ratio = mass / height) %>% filter(!is.na(stretch_ratio)) %>% arrange(desc(stretch_ratio)) %>% head(10) %>% select(name, mass, height, stretch_ratio)
```

    # A tibble: 10 × 4
       name                   mass height stretch_ratio
       <chr>                 <dbl>  <int>         <dbl>
     1 Jabba Desilijic Tiure  1358    175         7.76 
     2 Grievous                159    216         0.736
     3 IG-88                   140    200         0.7  
     4 Owen Lars               120    178         0.674
     5 Darth Vader             136    202         0.673
     6 Jek Tono Porkins        110    180         0.611
     7 Bossk                   113    190         0.595
     8 Tarfful                 136    234         0.581
     9 Dexter Jettster         102    198         0.515
    10 Chewbacca               112    228         0.491

1.  Найти средний возраст персонажей каждой расы вселенной Звездных
    войн.

``` r
starwars %>% mutate(age = 100 + birth_year) %>% group_by(species) %>% summarise(avg_age = mean(age, na.rm = TRUE), count = n()) %>% filter(!is.na(avg_age)) %>% arrange(desc(avg_age))
```

    # A tibble: 15 × 3
       species        avg_age count
       <chr>            <dbl> <int>
     1 Yoda's species    996      1
     2 Hutt              700      1
     3 Wookiee           300      2
     4 Cerean            192      1
     5 Zabrak            154      2
     6 Human             154.    35
     7 Droid             153.     6
     8 Trandoshan        153      1
     9 Gungan            152      3
    10 Mirialan          149      2
    11 Twi'lek           148      2
    12 Rodian            144      1
    13 Mon Calamari      141      1
    14 Kel Dor           122      1
    15 Ewok              108      1

1.  Найти самый распространенный цвет глаз персонажей вселенной Звездных
    войн.

``` r
starwars %>% count(eye_color, sort = TRUE) %>% filter(!is.na(eye_color)) %>% slice(1) %>% pull(eye_color)
```

    [1] "brown"

1.  Подсчитать среднюю длину имени в каждой расе вселенной Звездных
    войн.

``` r
starwars %>% mutate(name_length = nchar(name)) %>% filter(!is.na(species)) %>% group_by(species) %>% summarise(avg_name_length = mean(name_length, na.rm = TRUE),count = n()) %>% arrange(desc(avg_name_length))
```

    # A tibble: 37 × 3
       species   avg_name_length count
       <chr>               <dbl> <int>
     1 Ewok                 21       1
     2 Hutt                 21       1
     3 Geonosian            17       1
     4 Besalisk             15       1
     5 Mirialan             14       2
     6 Toong                14       1
     7 Aleena               12       1
     8 Cerean               12       1
     9 Gungan               11.7     3
    10 Human                11.3    35
    # ℹ 27 more rows

## Оценка результата

В результате лабораторной работы мы проанализировали встроенный в пакет
dplyr набор данных starwars с помощью языка R.

## Вывод

Таким образом, мы развили практические навыки использования языка
программирования R для обработки данных, закрепили знания базовых типов
данных языка R, развили практические навыки использования функций
обработки данных пакета dplyr – функции select(), filter(), mutate(),
arrange(), group_by().
