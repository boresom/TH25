# Использование технологии Yandex Query для анализа данных сетевой
активности


## Цель работы

1.  Получить знания о методах исследования радиоэлектронной обстановки;
2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI;
3.  Закрепить практические навыки использования языка программирования R
    для обработки данных;
4.  Закрепить знания основных функций обработки данных экосистемы языка
    R.

## Исходные данные

1.  Rstudio Desktop;
2.  Интерпретатор языка R 4.1.

### Решение

1.  Импортируйте данные.

``` r
library(readr)
```

    Warning: пакет 'readr' был собран под R версии 4.5.2

``` r
library(dplyr)
```

    Warning: пакет 'dplyr' был собран под R версии 4.5.2


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
library(lubridate)
```

    Warning: пакет 'lubridate' был собран под R версии 4.5.2


    Присоединяю пакет: 'lubridate'

    Следующие объекты скрыты от 'package:base':

        date, intersect, setdiff, union

``` r
library(janitor)
```

    Warning: пакет 'janitor' был собран под R версии 4.5.2


    Присоединяю пакет: 'janitor'

    Следующие объекты скрыты от 'package:stats':

        chisq.test, fisher.test

``` r
temp_dir <- tempdir()
download.file(
  url = "https://storage.yandexcloud.net/dataset.ctfsec/P2_wifi_data.csv",
  destfile = file.path(temp_dir, "P2_wifi_data.csv"),
  mode = "wb"
)

wifi_ap <- read_csv(file.path(temp_dir, "P2_wifi_data.csv"),
                      n_max = 167)
```

    Rows: 167 Columns: 15

    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (6): BSSID, Privacy, Cipher, Authentication, LAN IP, ESSID
    dbl  (6): channel, Speed, Power, # beacons, # IV, ID-length
    lgl  (1): Key
    dttm (2): First time seen, Last time seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
clients <- read_csv(file.path(temp_dir, "P2_wifi_data.csv"),
                      skip = 169)
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 12081 Columns: 7
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (3): Station MAC, BSSID, Probed ESSIDs
    dbl  (2): Power, # packets
    dttm (2): First time seen, Last time seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
print(head(wifi_ap, 10))
```

    # A tibble: 10 × 15
       BSSID    `First time seen`   `Last time seen`    channel Speed Privacy Cipher
       <chr>    <dttm>              <dttm>                <dbl> <dbl> <chr>   <chr> 
     1 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195 WPA2    CCMP  
     2 6E:C7:E… 2023-07-28 09:13:03 2023-07-28 11:55:12       1   130 WPA2    CCMP  
     3 9A:75:A… 2023-07-28 09:13:03 2023-07-28 11:53:31       1   360 WPA2    CCMP  
     4 4A:EC:1… 2023-07-28 09:13:03 2023-07-28 11:04:01       7   360 WPA2    CCMP  
     5 D2:6D:5… 2023-07-28 09:13:03 2023-07-28 10:30:19       6   130 WPA2    CCMP  
     6 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     7 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:44      11   195 WPA2    CCMP  
     8 0A:C5:E… 2023-07-28 09:13:03 2023-07-28 11:36:31      11   130 WPA2    CCMP  
     9 38:1A:5… 2023-07-28 09:13:03 2023-07-28 10:25:02      11   130 WPA2    CCMP  
    10 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 10:29:21       1   195 WPA2    CCMP  
    # ℹ 8 more variables: Authentication <chr>, Power <dbl>, `# beacons` <dbl>,
    #   `# IV` <dbl>, `LAN IP` <chr>, `ID-length` <dbl>, ESSID <chr>, Key <lgl>

``` r
print(head(clients, 10))
```

    # A tibble: 10 × 7
       `Station MAC` `First time seen`   `Last time seen`    Power `# packets` BSSID
       <chr>         <dttm>              <dttm>              <dbl>       <dbl> <chr>
     1 CA:66:3B:8F:… 2023-07-28 09:13:03 2023-07-28 10:59:44   -33         858 BE:F…
     2 96:35:2D:3D:… 2023-07-28 09:13:03 2023-07-28 09:13:03   -65           4 (not…
     3 5C:3A:45:9E:… 2023-07-28 09:13:03 2023-07-28 11:51:54   -39         432 BE:F…
     4 C0:E4:34:D8:… 2023-07-28 09:13:03 2023-07-28 11:53:16   -61         958 BE:F…
     5 5E:8E:A6:5E:… 2023-07-28 09:13:04 2023-07-28 09:13:04   -53           1 (not…
     6 10:51:07:CB:… 2023-07-28 09:13:05 2023-07-28 11:56:06   -43         344 (not…
     7 68:54:5A:40:… 2023-07-28 09:13:06 2023-07-28 11:50:50   -31         163 1E:9…
     8 74:4C:A1:70:… 2023-07-28 09:13:06 2023-07-28 09:20:01   -71           3 E8:2…
     9 8A:A3:5A:33:… 2023-07-28 09:13:06 2023-07-28 10:20:27   -74         115 00:2…
    10 CA:54:C4:8B:… 2023-07-28 09:13:06 2023-07-28 11:55:04   -65         437 00:2…
    # ℹ 1 more variable: `Probed ESSIDs` <chr>

1.  Привести датасеты в вид “аккуратных данных”, преобразовать типы
    столбцов в соответствии с типом данных и посмотреть с помощью
    `glimpse`.

``` r
wifi_ap_clean <- wifi_ap %>%
  rename_with(~ gsub("\\s+", "_", .x)) %>%
  rename(
    bssid = BSSID,
    first_time_seen = `First_time_seen`,
    last_time_seen = `Last_time_seen`,
    channel = channel,
    speed = Speed,
    privacy = Privacy,
    cipher = Cipher,
    authentication = Authentication,
    power = Power,
    beacons = `#_beacons`,
    iv = `#_IV`,
    lan_ip = LAN_IP,
    id_length = `ID-length`,
    essid = ESSID,
    key = Key
  ) %>%
  mutate(
    first_time_seen = as.POSIXct(first_time_seen, format = "%Y-%m-%d %H:%M:%S"),
    last_time_seen = as.POSIXct(last_time_seen, format = "%Y-%m-%d %H:%M:%S"),
    channel = as.integer(channel),
    speed = as.integer(speed),
    power = as.integer(power),
    beacons = as.integer(beacons),
    iv = as.integer(iv),
    id_length = as.integer(id_length),
    privacy = as.factor(privacy),
    cipher = as.factor(cipher),
    authentication = as.factor(authentication),
    lan_ip = gsub("\\s+", "", lan_ip)
  )

print(head(wifi_ap_clean, 10))
```

    # A tibble: 10 × 15
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195 WPA2    CCMP  
     2 6E:C7:E… 2023-07-28 09:13:03 2023-07-28 11:55:12       1   130 WPA2    CCMP  
     3 9A:75:A… 2023-07-28 09:13:03 2023-07-28 11:53:31       1   360 WPA2    CCMP  
     4 4A:EC:1… 2023-07-28 09:13:03 2023-07-28 11:04:01       7   360 WPA2    CCMP  
     5 D2:6D:5… 2023-07-28 09:13:03 2023-07-28 10:30:19       6   130 WPA2    CCMP  
     6 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     7 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:44      11   195 WPA2    CCMP  
     8 0A:C5:E… 2023-07-28 09:13:03 2023-07-28 11:36:31      11   130 WPA2    CCMP  
     9 38:1A:5… 2023-07-28 09:13:03 2023-07-28 10:25:02      11   130 WPA2    CCMP  
    10 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 10:29:21       1   195 WPA2    CCMP  
    # ℹ 8 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>

``` r
clients_clean <- clients %>%
  rename_with(~ gsub("\\s+", "_", .x)) %>%
  rename(
    station_mac = Station_MAC,
    first_time_seen = `First_time_seen`,
    last_time_seen = `Last_time_seen`,
    power = Power,
    packets = `#_packets`,
    bssid = BSSID,
    probed_essids = `Probed_ESSIDs`
  ) %>%
  mutate(
    first_time_seen = as.POSIXct(first_time_seen, format = "%Y-%m-%d %H:%M:%S"),
    last_time_seen = as.POSIXct(last_time_seen, format = "%Y-%m-%d %H:%M:%S"),
    power = as.integer(power),
    packets = as.integer(packets),
    bssid = ifelse(bssid == "(not associated)", NA, bssid)
  )

print(head(clients_clean, 10))
```

    # A tibble: 10 × 7
       station_mac       first_time_seen     last_time_seen      power packets bssid
       <chr>             <dttm>              <dttm>              <int>   <int> <chr>
     1 CA:66:3B:8F:56:DD 2023-07-28 09:13:03 2023-07-28 10:59:44   -33     858 BE:F…
     2 96:35:2D:3D:85:E6 2023-07-28 09:13:03 2023-07-28 09:13:03   -65       4 <NA> 
     3 5C:3A:45:9E:1A:7B 2023-07-28 09:13:03 2023-07-28 11:51:54   -39     432 BE:F…
     4 C0:E4:34:D8:E7:E5 2023-07-28 09:13:03 2023-07-28 11:53:16   -61     958 BE:F…
     5 5E:8E:A6:5E:34:81 2023-07-28 09:13:04 2023-07-28 09:13:04   -53       1 <NA> 
     6 10:51:07:CB:33:E7 2023-07-28 09:13:05 2023-07-28 11:56:06   -43     344 <NA> 
     7 68:54:5A:40:35:9E 2023-07-28 09:13:06 2023-07-28 11:50:50   -31     163 1E:9…
     8 74:4C:A1:70:CE:F7 2023-07-28 09:13:06 2023-07-28 09:20:01   -71       3 E8:2…
     9 8A:A3:5A:33:76:57 2023-07-28 09:13:06 2023-07-28 10:20:27   -74     115 00:2…
    10 CA:54:C4:8B:B5:3A 2023-07-28 09:13:06 2023-07-28 11:55:04   -65     437 00:2…
    # ℹ 1 more variable: probed_essids <chr>

1.  Определить небезопасные точки доступа (без шифрования – OPN)

``` r
unsafe_ap <- wifi_ap_clean %>%
  filter(privacy == "OPN")

print(head(unsafe_ap, 10))
```

    # A tibble: 10 × 15
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     2 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     3 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:11       6   130 OPN     <NA>  
     4 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:10       6    -1 OPN     <NA>  
     5 00:25:0… 2023-07-28 09:13:06 2023-07-28 11:56:21      44    -1 OPN     <NA>  
     6 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
     7 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:27:06       6   130 OPN     <NA>  
     8 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:39:43       6   130 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:13:17 2023-07-28 11:52:32       1   130 OPN     <NA>  
    10 E8:28:C… 2023-07-28 09:13:50 2023-07-28 11:43:39      11   130 OPN     <NA>  
    # ℹ 8 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>

1.  Определить производителя для каждого обнаруженного устройства

``` r
library(httr)
library(jsonlite)

get_manufacturer_by_mac <- function(mac_address, timeout = 10) {
  url <- paste0("https://www.macvendorlookup.com/api/v2/", mac_address)
  tryCatch({
    response <- httr::GET(
      url,
      httr::timeout(timeout),
      httr::add_headers(
        "User-Agent" = "Mozilla/5.0 (compatible; R script)"
      )
    )
    if (httr::status_code(response) != 200) {
      return(NULL)
    }
    
    content <- httr::content(response, "text", encoding = "UTF-8")
    data <- jsonlite::fromJSON(content)
    if (length(data) == 0 || is.null(data$company) || data$company == "") {
      return(NULL)
    }
    return(data$company[1])
    
  }, error = function(e) {
    return(NULL)
  })
}
wifi_ap_clean$manufacturer <- sapply(wifi_ap_clean$bssid, function(mac) {
  result <- get_manufacturer_by_mac(mac)
  if (is.null(result)) {
    return(NA)
  } else {
    return(result)
  }
})

print(head(wifi_ap_clean, 10))
```

    # A tibble: 10 × 16
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195 WPA2    CCMP  
     2 6E:C7:E… 2023-07-28 09:13:03 2023-07-28 11:55:12       1   130 WPA2    CCMP  
     3 9A:75:A… 2023-07-28 09:13:03 2023-07-28 11:53:31       1   360 WPA2    CCMP  
     4 4A:EC:1… 2023-07-28 09:13:03 2023-07-28 11:04:01       7   360 WPA2    CCMP  
     5 D2:6D:5… 2023-07-28 09:13:03 2023-07-28 10:30:19       6   130 WPA2    CCMP  
     6 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     7 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:44      11   195 WPA2    CCMP  
     8 0A:C5:E… 2023-07-28 09:13:03 2023-07-28 11:36:31      11   130 WPA2    CCMP  
     9 38:1A:5… 2023-07-28 09:13:03 2023-07-28 10:25:02      11   130 WPA2    CCMP  
    10 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 10:29:21       1   195 WPA2    CCMP  
    # ℹ 9 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>

1.  Выявить устройства, использующие последнюю версию протокола
    шифрования WPA3, и названия точек доступа, реализованных на этих
    устройствах

``` r
wifi_wpa3_ap <- wifi_ap_clean %>% filter(grepl("WPA3", privacy))
print(head(wifi_wpa3_ap, 10))
```

    # A tibble: 8 × 16
      bssid     first_time_seen     last_time_seen      channel speed privacy cipher
      <chr>     <dttm>              <dttm>                <int> <int> <fct>   <fct> 
    1 26:20:53… 2023-07-28 09:15:45 2023-07-28 09:33:10      44   866 WPA3 W… CCMP  
    2 A2:FE:FF… 2023-07-28 09:41:52 2023-07-28 09:41:52       6   130 WPA3 W… CCMP  
    3 96:FF:FC… 2023-07-28 09:52:54 2023-07-28 10:25:02      44   866 WPA3 W… CCMP  
    4 CE:48:E7… 2023-07-28 09:59:20 2023-07-28 10:04:15      44   866 WPA3 W… CCMP  
    5 8E:1F:94… 2023-07-28 10:08:32 2023-07-28 10:15:27      44   866 WPA3 W… CCMP  
    6 BE:FD:EF… 2023-07-28 10:15:24 2023-07-28 10:15:28       6   130 WPA3 W… CCMP  
    7 3A:DA:00… 2023-07-28 10:27:01 2023-07-28 10:27:10       6   130 WPA3 W… CCMP  
    8 76:C5:A0… 2023-07-28 11:16:36 2023-07-28 11:16:38       6   130 WPA3 W… CCMP  
    # ℹ 9 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>

1.  Отсортировать точки доступа по интервалу времени, в течение которого
    они находились на связи, по убыванию

``` r
wifi_ap_clean <- wifi_ap_clean %>%
  mutate(life_time = as.numeric(difftime(last_time_seen, first_time_seen, units = "mins"))) %>%
  arrange(desc(life_time))

print(head(wifi_ap_clean, 10))
```

    # A tibble: 10 × 17
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 00:25:0… 2023-07-28 09:13:06 2023-07-28 11:56:21      44    -1 OPN     <NA>  
     2 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
     3 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     4 08:3A:2… 2023-07-28 09:13:27 2023-07-28 11:55:53      14    -1 WPA     <NA>  
     5 6E:C7:E… 2023-07-28 09:13:03 2023-07-28 11:55:12       1   130 WPA2    CCMP  
     6 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     7 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:11       6   130 OPN     <NA>  
     8 48:5B:3… 2023-07-28 09:13:06 2023-07-28 11:55:11       1   270 WPA2    CCMP  
     9 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:10       6    -1 OPN     <NA>  
    10 8E:55:4… 2023-07-28 09:13:06 2023-07-28 11:55:09       6    65 WPA2    CCMP  
    # ℹ 10 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>, life_time <dbl>

1.  Обнаружить топ-10 самых быстрых точек доступа

``` r
top10_fastest <- wifi_ap_clean %>%
  arrange(desc(speed)) %>%
  head(10)
print(top10_fastest)
```

    # A tibble: 10 × 17
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 96:FF:F… 2023-07-28 09:52:54 2023-07-28 10:25:02      44   866 WPA3 W… CCMP  
     2 26:20:5… 2023-07-28 09:15:45 2023-07-28 09:33:10      44   866 WPA3 W… CCMP  
     3 8E:1F:9… 2023-07-28 10:08:32 2023-07-28 10:15:27      44   866 WPA3 W… CCMP  
     4 CE:48:E… 2023-07-28 09:59:20 2023-07-28 10:04:15      44   866 WPA3 W… CCMP  
     5 9A:75:A… 2023-07-28 09:13:03 2023-07-28 11:53:31       1   360 WPA2    CCMP  
     6 E8:28:C… 2023-07-28 09:18:30 2023-07-28 11:55:10      52   360 OPN     <NA>  
     7 E8:28:C… 2023-07-28 09:18:30 2023-07-28 11:55:10      52   360 OPN     <NA>  
     8 E8:28:C… 2023-07-28 09:18:16 2023-07-28 11:51:48      48   360 OPN     <NA>  
     9 14:EB:B… 2023-07-28 09:25:01 2023-07-28 11:53:36       3   360 WPA2    CCMP  
    10 E8:28:C… 2023-07-28 09:18:30 2023-07-28 11:43:23      48   360 OPN     <NA>  
    # ℹ 10 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>, life_time <dbl>

1.  Отсортировать точки доступа по частоте отправки запросов (beacons) в
    единицу времени по их убыванию

``` r
print(wifi_ap_clean %>%
  mutate(
    beacons_per_minute = beacons / life_time
  ) %>%
  filter(life_time > 0) %>%
  arrange(desc(beacons_per_minute)) %>%
  select(essid, bssid, beacons_per_minute, beacons, life_time, first_time_seen, last_time_seen)
  %>% head(10))
```

    # A tibble: 10 × 7
       essid          bssid beacons_per_minute beacons life_time first_time_seen    
       <chr>          <chr>              <dbl>   <int>     <dbl> <dttm>             
     1 "iPhone (Ulia… F2:3…              51.4        6    0.117  2023-07-28 10:27:02
     2 "Михаил's Gal… B2:C…              48          4    0.0833 2023-07-28 10:40:54
     3 "iPhone XS Ma… 3A:D…              33.3        5    0.15   2023-07-28 10:27:01
     4 "MT_FREE"      02:B…              30          1    0.0333 2023-07-28 09:24:46
     5 "MT_FREE"      00:3…              30          1    0.0333 2023-07-28 10:34:03
     6  <NA>          76:C…              30          1    0.0333 2023-07-28 11:16:36
     7 "Саня"         D2:2…              23.1        5    0.217  2023-07-28 09:45:29
     8 "C322U21 0566" BE:F…              10.4     1647  158.     2023-07-28 09:13:03
     9 "MT_FREE"      00:0…              10          1    0.1    2023-07-28 10:29:13
    10 "EBFCD57F-EE8… 38:1…               9.78     704   72.0    2023-07-28 09:13:03
    # ℹ 1 more variable: last_time_seen <dttm>

1.  Определить производителя для каждого обнаруженного устройства

<!-- -->

    clients_clean$manufacturer <- sapply(clients_clean$station_mac, function(mac) {
      result <- get_manufacturer_by_mac(mac)
      if (is.null(result)) {
        return(NA)
      } else {
        return(result)
      }
    })

    print(clients_clean %>% head(10))

1.  Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес

Выборка тех MAC-адресов, в которых второй бит первого байта не 1, то
есть не управляемый на местном уровне, а регулируемый производителем.

``` r
clients_strong <- clients_clean %>% filter(!substr(station_mac, 2,2) %in% c("2", "3", "6", "7", "A", "B", "E", "F"))
print(head(clients_strong, 10))
```

    # A tibble: 10 × 7
       station_mac       first_time_seen     last_time_seen      power packets bssid
       <chr>             <dttm>              <dttm>              <int>   <int> <chr>
     1 5C:3A:45:9E:1A:7B 2023-07-28 09:13:03 2023-07-28 11:51:54   -39     432 BE:F…
     2 C0:E4:34:D8:E7:E5 2023-07-28 09:13:03 2023-07-28 11:53:16   -61     958 BE:F…
     3 10:51:07:CB:33:E7 2023-07-28 09:13:05 2023-07-28 11:56:06   -43     344 <NA> 
     4 68:54:5A:40:35:9E 2023-07-28 09:13:06 2023-07-28 11:50:50   -31     163 1E:9…
     5 74:4C:A1:70:CE:F7 2023-07-28 09:13:06 2023-07-28 09:20:01   -71       3 E8:2…
     6 BC:F1:71:D4:DB:04 2023-07-28 09:13:07 2023-07-28 10:57:52   -45     265 <NA> 
     7 4C:44:5B:14:76:E3 2023-07-28 09:13:09 2023-07-28 09:47:44    -1      71 E8:2…
     8 A0:E7:0B:AE:D5:44 2023-07-28 09:13:09 2023-07-28 11:34:42   -37     125 0A:C…
     9 00:95:69:E7:7F:35 2023-07-28 09:13:11 2023-07-28 11:56:07   -69    2245 <NA> 
    10 00:95:69:E7:7C:ED 2023-07-28 09:13:11 2023-07-28 11:56:13   -55    4096 <NA> 
    # ℹ 1 more variable: probed_essids <chr>

1.  Кластеризовать запросы от устройств к точкам доступа по их именам.
    Определить время появления устройства в зоне радиовидимости и время
    выхода его из нее

``` r
library(tidyverse)
```

    Warning: пакет 'tidyverse' был собран под R версии 4.5.2

    Warning: пакет 'ggplot2' был собран под R версии 4.5.2

    Warning: пакет 'tibble' был собран под R версии 4.5.2

    Warning: пакет 'tidyr' был собран под R версии 4.5.2

    Warning: пакет 'purrr' был собран под R версии 4.5.2

    Warning: пакет 'forcats' был собран под R версии 4.5.2

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ forcats 1.0.1     ✔ stringr 1.5.2
    ✔ ggplot2 4.0.1     ✔ tibble  3.3.0
    ✔ purrr   1.2.0     ✔ tidyr   1.3.1
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter()  masks stats::filter()
    ✖ purrr::flatten() masks jsonlite::flatten()
    ✖ dplyr::lag()     masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
all_networks <- clients_clean %>%
  filter(!is.na(probed_essids) & probed_essids != "") %>%
  distinct(probed_essids) %>%
  pull(probed_essids)

client_network_matrix <- clients_clean %>%
  filter(!is.na(probed_essids) & probed_essids != "") %>%
  group_by(station_mac, probed_essids) %>%
  summarise(probed_count = n(), .groups = 'drop') %>%
  pivot_wider(
    names_from = probed_essids,
    values_from = probed_count,
    values_fill = 0,
    names_prefix = "network_"
  ) %>%
  mutate(across(-station_mac, ~ replace_na(., 0)))

network_frequency <- colSums(client_network_matrix[, -1] > 0)
popular_networks <- names(network_frequency[network_frequency >= 5])

client_network_filtered <- client_network_matrix %>%
  select(station_mac, all_of(popular_networks))

cluster_data <- client_network_filtered %>%
  select(-station_mac) %>%
  as.matrix()

library(factoextra)
```

    Warning: пакет 'factoextra' был собран под R версии 4.5.2

    Welcome! Want to learn more? See two factoextra-related books at https://goo.gl/ve3WBa

``` r
fviz_nbclust(cluster_data, kmeans, method = "wss", k.max = 10) +
  labs(title = "Метод локтя для определения числа кластеров")
```

![](L5.markdown_strict_files/figure-markdown_strict/unnamed-chunk-10-1.png)

``` r
set.seed(123)
k <- 4
kmeans_result <- kmeans(cluster_data, centers = k, nstart = 25)

client_network_filtered$cluster <- as.factor(kmeans_result$cluster)

cluster_profiles <- client_network_filtered %>%
  group_by(cluster) %>%
  summarise(across(starts_with("network_"), mean)) %>%
  pivot_longer(
    cols = starts_with("network_"),
    names_to = "network",
    names_prefix = "network_",
    values_to = "proportion"
  ) %>%
  group_by(cluster) %>%
  arrange(cluster, desc(proportion)) %>%
  slice_max(proportion, n = 5)

ggplot(cluster_profiles, aes(x = reorder(network, proportion), y = proportion, fill = cluster)) +
  geom_col() +
  facet_wrap(~ cluster, scales = "free_y") +
  coord_flip() +
  labs(
    title = "Топ-5 исследуемых сетей по кластерам",
    x = "Сеть",
    y = "Доля устройств в кластере, искавших сеть"
  ) +
  theme_minimal()
```

![](L5.markdown_strict_files/figure-markdown_strict/unnamed-chunk-10-2.png)

1.  Оценить стабильность уровня сигнала внури кластера во времени.
    Выявить наиболее стабильный кластер

``` r
clients_with_clusters <- clients_clean %>%
  inner_join(client_network_filtered %>% select(station_mac, cluster), by = "station_mac") %>%
  filter(!is.na(power) & power != -1)

clients_analysis <- clients_with_clusters %>%
  mutate(
    hour = hour(first_time_seen),
    time_bin = cut(hour, breaks = c(0, 6, 12, 18, 24), 
                   labels = c("Ночь", "Утро", "День", "Вечер"))
  )

cluster_stability <- clients_analysis %>%
  group_by(cluster) %>%
  summarise(
    n_measurements = n(),
    mean_power = mean(power, na.rm = TRUE),
    median_power = median(power, na.rm = TRUE),
    sd_power = sd(power, na.rm = TRUE),
    cv_power = sd_power / abs(mean_power),
    min_power = min(power, na.rm = TRUE),
    max_power = max(power, na.rm = TRUE),
    range_power = max_power - min_power,
    iqr_power = IQR(power, na.rm = TRUE),
    q1_power = quantile(power, 0.25, na.rm = TRUE),
    q3_power = quantile(power, 0.75, na.rm = TRUE)
  ) %>%
  arrange(cv_power)

stability_ranking <- cluster_stability %>%
  select(cluster, mean_power, sd_power, cv_power, range_power, iqr_power) %>%
  mutate(
    stability_score = 1/cv_power, 
    rank_stability = rank(cv_power)  
  ) %>%
  arrange(rank_stability)

cat("РЕЙТИНГ СТАБИЛЬНОСТИ КЛАСТЕРОВ:\n")
```

    РЕЙТИНГ СТАБИЛЬНОСТИ КЛАСТЕРОВ:

``` r
print(stability_ranking)
```

    # A tibble: 4 × 8
      cluster mean_power sd_power cv_power range_power iqr_power stability_score
      <fct>        <dbl>    <dbl>    <dbl>       <int>     <dbl>           <dbl>
    1 3            -58.0     5.73   0.0987          36         8           10.1 
    2 4            -54.5     5.83   0.107           32         8            9.34
    3 1            -61.5     7.33   0.119           66         6            8.39
    4 2            -54.5     9.09   0.167           41         8            5.99
    # ℹ 1 more variable: rank_stability <dbl>

``` r
most_stable_cluster <- stability_ranking %>%
  filter(rank_stability == 1)

cat("\nСАМЫЙ СТАБИЛЬНЫЙ КЛАСТЕР:\n")
```


    САМЫЙ СТАБИЛЬНЫЙ КЛАСТЕР:

``` r
cat("Кластер:", most_stable_cluster$cluster, "\n")
```

    Кластер: 3 

``` r
cat("Средний уровень сигнала:", round(most_stable_cluster$mean_power, 2), "dBm\n")
```

    Средний уровень сигнала: -58.05 dBm

``` r
cat("Стандартное отклонение:", round(most_stable_cluster$sd_power, 2), "dBm\n")
```

    Стандартное отклонение: 5.73 dBm

``` r
cat("Коэффициент вариации:", round(most_stable_cluster$cv_power, 3), "\n")
```

    Коэффициент вариации: 0.099 

``` r
cat("Размах:", most_stable_cluster$range_power, "dBm\n")
```

    Размах: 36 dBm

## Оценка результата

В рамках практческой работы была исследована радиоэлектронная обстановка
и составлено представление о механизмах работы Wi-Fi сетей на канальном
и сетевом уровне модели OSI.

## Вывод

В практической работе мы использовали навыки написания кода на языке
программирования R для обработки данных и закрепили знания основных
функций обработки данных экосистемы tidyverse языка R.
