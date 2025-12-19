# Исследование метаданных DNS трафика


## Цель работы

1.  Зекрепить практические навыки использования языка программирования R
    для обработки данных;
2.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R;
3.  Закрепить навыки исследования метаданных DNS трафика.

## Исходные данные

1.  Rstudio Desktop;
2.  Интерпретатор языка R 4.1;
3.  Экосистема tidyverse.

## Задание

Используя программный пакет dplyr, освоить анализ DNS логов с помощью
языка программирования R.

## Подготовка данных

1.  Импортируйте данные DNS –
    https://storage.yandexcloud.net/dataset.ctfsec/dns.zip

``` r
temp_dir <- tempdir()
download.file(url = "https://storage.yandexcloud.net/dataset.ctfsec/dns.zip", destfile = file.path(temp_dir, "dns.zip"),  mode = "wb")
unzip(zipfile = file.path(temp_dir, "dns.zip"), exdir = temp_dir)
```

### Обработка данных

2 и 3. Добавьте пропущенные данные о структуре данных (назначении
столбцов) и преобразуйте данные в столбцах в нужный формат

``` r
library(httr)
library(jsonlite)
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
library(stringr)
library(tidyr)
```

    Warning: пакет 'tidyr' был собран под R версии 4.5.2

``` r
library(knitr)
column_names <- c(
  "timestamp", "uid", "source_ip", "source_port", "destination_ip", 
  "destination_port", "protocol", "transaction_id", "query", "qclass", 
  "qclass_name", "qtype", "qtype_name", "rcode", "rcode_name", 
  "AA", "TC", "RD", "RA", "Z", "answers", "TTLS", "rejected"
)
log_files <- list.files(temp_dir, pattern = "\\.log$", full.names = TRUE)
dns_data <- invisible(read_delim(
  log_files[1],
  delim = "\t",
  col_names = column_names,
  comment = "#",
  na = c("", "NA", "-"),
  trim_ws = TRUE,
  show_col_types = FALSE
)) %>% as_tibble()
dns_data2 <- dns_data %>%
  mutate(
    timestamp = as.POSIXct(timestamp, origin = "1970-01-01"),
    source_port = as.numeric(source_port),
    destination_port = as.numeric(destination_port),
    transaction_id = as.numeric(transaction_id),
    qclass = as.numeric(qclass),
    qtype = as.numeric(qtype),
    rcode = as.numeric(rcode),
  ) %>% as_tibble()
head(dns_data2,10) %>% knitr::kable(format='markdown')
```

<table style="width:100%;">
<colgroup>
<col style="width: 7%" />
<col style="width: 6%" />
<col style="width: 5%" />
<col style="width: 4%" />
<col style="width: 5%" />
<col style="width: 5%" />
<col style="width: 3%" />
<col style="width: 5%" />
<col style="width: 20%" />
<col style="width: 2%" />
<col style="width: 4%" />
<col style="width: 2%" />
<col style="width: 3%" />
<col style="width: 2%" />
<col style="width: 3%" />
<col style="width: 2%" />
<col style="width: 2%" />
<col style="width: 2%" />
<col style="width: 2%" />
<col style="width: 1%" />
<col style="width: 2%" />
<col style="width: 1%" />
<col style="width: 3%" />
</colgroup>
<thead>
<tr>
<th style="text-align: left;">timestamp</th>
<th style="text-align: left;">uid</th>
<th style="text-align: left;">source_ip</th>
<th style="text-align: right;">source_port</th>
<th style="text-align: left;">destination_ip</th>
<th style="text-align: right;">destination_port</th>
<th style="text-align: left;">protocol</th>
<th style="text-align: right;">transaction_id</th>
<th style="text-align: left;">query</th>
<th style="text-align: right;">qclass</th>
<th style="text-align: left;">qclass_name</th>
<th style="text-align: right;">qtype</th>
<th style="text-align: left;">qtype_name</th>
<th style="text-align: right;">rcode</th>
<th style="text-align: left;">rcode_name</th>
<th style="text-align: left;">AA</th>
<th style="text-align: left;">TC</th>
<th style="text-align: left;">RD</th>
<th style="text-align: left;">RA</th>
<th style="text-align: right;">Z</th>
<th style="text-align: left;">answers</th>
<th style="text-align: left;">TTLS</th>
<th style="text-align: left;">rejected</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: left;">2012-03-16 16:30:05</td>
<td style="text-align: left;">CWGtK431H9XuaTN4fi</td>
<td style="text-align: left;">192.168.202.100</td>
<td style="text-align: right;">45658</td>
<td style="text-align: left;">192.168.27.203</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">udp</td>
<td style="text-align: right;">33008</td>
<td style="text-align: left;">*</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C_INTERNET</td>
<td style="text-align: right;">33</td>
<td style="text-align: left;">SRV</td>
<td style="text-align: right;">0</td>
<td style="text-align: left;">NOERROR</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
</tr>
<tr>
<td style="text-align: left;">2012-03-16 16:30:15</td>
<td style="text-align: left;">C36a282Jljz7BsbGH</td>
<td style="text-align: left;">192.168.202.76</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">192.168.202.255</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">udp</td>
<td style="text-align: right;">57402</td>
<td style="text-align: left;">HPE8AA67</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C_INTERNET</td>
<td style="text-align: right;">32</td>
<td style="text-align: left;">NB</td>
<td style="text-align: right;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">TRUE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
</tr>
<tr>
<td style="text-align: left;">2012-03-16 16:30:15</td>
<td style="text-align: left;">C36a282Jljz7BsbGH</td>
<td style="text-align: left;">192.168.202.76</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">192.168.202.255</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">udp</td>
<td style="text-align: right;">57402</td>
<td style="text-align: left;">HPE8AA67</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C_INTERNET</td>
<td style="text-align: right;">32</td>
<td style="text-align: left;">NB</td>
<td style="text-align: right;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">TRUE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
</tr>
<tr>
<td style="text-align: left;">2012-03-16 16:30:16</td>
<td style="text-align: left;">C36a282Jljz7BsbGH</td>
<td style="text-align: left;">192.168.202.76</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">192.168.202.255</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">udp</td>
<td style="text-align: right;">57402</td>
<td style="text-align: left;">HPE8AA67</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C_INTERNET</td>
<td style="text-align: right;">32</td>
<td style="text-align: left;">NB</td>
<td style="text-align: right;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">TRUE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
</tr>
<tr>
<td style="text-align: left;">2012-03-16 16:30:05</td>
<td style="text-align: left;">C36a282Jljz7BsbGH</td>
<td style="text-align: left;">192.168.202.76</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">192.168.202.255</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">udp</td>
<td style="text-align: right;">57398</td>
<td style="text-align: left;">WPAD</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C_INTERNET</td>
<td style="text-align: right;">32</td>
<td style="text-align: left;">NB</td>
<td style="text-align: right;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">TRUE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
</tr>
<tr>
<td style="text-align: left;">2012-03-16 16:30:06</td>
<td style="text-align: left;">C36a282Jljz7BsbGH</td>
<td style="text-align: left;">192.168.202.76</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">192.168.202.255</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">udp</td>
<td style="text-align: right;">57398</td>
<td style="text-align: left;">WPAD</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C_INTERNET</td>
<td style="text-align: right;">32</td>
<td style="text-align: left;">NB</td>
<td style="text-align: right;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">TRUE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
</tr>
<tr>
<td style="text-align: left;">2012-03-16 16:30:07</td>
<td style="text-align: left;">C36a282Jljz7BsbGH</td>
<td style="text-align: left;">192.168.202.76</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">192.168.202.255</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">udp</td>
<td style="text-align: right;">57398</td>
<td style="text-align: left;">WPAD</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C_INTERNET</td>
<td style="text-align: right;">32</td>
<td style="text-align: left;">NB</td>
<td style="text-align: right;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">TRUE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
</tr>
<tr>
<td style="text-align: left;">2012-03-16 16:30:06</td>
<td style="text-align: left;">ClEZCt3GLkJdtGGmAa</td>
<td style="text-align: left;">192.168.202.89</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">192.168.202.255</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">udp</td>
<td style="text-align: right;">62187</td>
<td style="text-align: left;">EWREP1</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C_INTERNET</td>
<td style="text-align: right;">32</td>
<td style="text-align: left;">NB</td>
<td style="text-align: right;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">TRUE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
</tr>
<tr>
<td style="text-align: left;">2012-03-16 16:30:07</td>
<td style="text-align: left;">ClEZCt3GLkJdtGGmAa</td>
<td style="text-align: left;">192.168.202.89</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">192.168.202.255</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">udp</td>
<td style="text-align: right;">62187</td>
<td style="text-align: left;">EWREP1</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C_INTERNET</td>
<td style="text-align: right;">32</td>
<td style="text-align: left;">NB</td>
<td style="text-align: right;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">TRUE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
</tr>
<tr>
<td style="text-align: left;">2012-03-16 16:30:07</td>
<td style="text-align: left;">ClEZCt3GLkJdtGGmAa</td>
<td style="text-align: left;">192.168.202.89</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">192.168.202.255</td>
<td style="text-align: right;">137</td>
<td style="text-align: left;">udp</td>
<td style="text-align: right;">62187</td>
<td style="text-align: left;">EWREP1</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">C_INTERNET</td>
<td style="text-align: right;">32</td>
<td style="text-align: left;">NB</td>
<td style="text-align: right;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: left;">TRUE</td>
<td style="text-align: left;">FALSE</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FALSE</td>
</tr>
</tbody>
</table>

1.  Сколько участников информационного обмена в сети Доброй Организации?

``` r
length(unique(c(unique(dns_data2$source_ip),unique(dns_data2$destination_ip))))
```

    [1] 1359

1.  Соотношение участников обмена внутри сети и участников обращений к
    внешним ресурсам.

``` r
unique_ips = unique(c(unique(dns_data2$source_ip),unique(dns_data2$destination_ip)))

is_internal_ip <- function(ip_str) {
  if (is.na(ip_str)) return(FALSE)
  internal_patterns <- c(
    "^10\\.",                    # 10.0.0.0
    "^172\\.(1[6-9]|2[0-9]|3[0-1])\\.", # 172.16.0.0
    "^192\\.168\\.",             # 192.168.0.0
    "^127\\."                    # 127.0.0.0
  )
  return(any(sapply(internal_patterns, function(p) str_detect(ip_str, p))))
}

unique_ip_df <- data.frame(ip = unique_ips) %>%  mutate(is_internal = sapply(ip, is_internal_ip), type = ifelse(is_internal, "Internal", "External"))

ip_counts <- unique_ip_df %>% group_by(type) %>%summarise(unique_count = n())
print(ip_counts)
```

    # A tibble: 2 × 2
      type     unique_count
      <chr>           <int>
    1 External           92
    2 Internal         1267

1.  Найдите топ-10 участников сети, проявляющих наибольшую сетевую
    активность.

``` r
all_ips <- dns_data2 %>%
  select(source_ip, destination_ip) %>%
  tidyr::pivot_longer(
    cols = everything(),
    names_to = "role",
    values_to = "ip_address"
  )

active_participants <- all_ips %>%
  filter(!is.na(ip_address)) %>%
  group_by(ip_address) %>%
  summarise(
    request_count = n()
  ) %>%
  ungroup() %>%
  arrange(desc(request_count))

print(head(active_participants, 10))
```

    # A tibble: 10 × 2
       ip_address      request_count
       <chr>                   <int>
     1 192.168.207.4          266627
     2 10.10.117.210           75943
     3 192.168.202.255         68720
     4 192.168.202.93          26522
     5 172.19.1.100            25481
     6 192.168.202.103         18121
     7 192.168.202.76          16978
     8 192.168.202.97          16176
     9 192.168.202.141         14976
    10 192.168.202.110         14493

1.  Найдите топ-10 доменов, к которым обращаются пользователи сети и
    соответственное количество обращений

``` r
domain_counts <- dns_data2 %>%
  filter(!is.na(query)) %>%
  group_by(query) %>%
  summarise(
    request_count = n()
  ) %>%
  ungroup() %>%
  arrange(desc(request_count))

print(head(domain_counts, 10))
```

    # A tibble: 10 × 2
       query                                                           request_count
       <chr>                                                                   <int>
     1 "teredo.ipv6.microsoft.com"                                             39273
     2 "tools.google.com"                                                      14057
     3 "www.apple.com"                                                         13390
     4 "time.apple.com"                                                        13109
     5 "safebrowsing.clients.google.com"                                       11658
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00…         10401
     7 "WPAD"                                                                   9134
     8 "44.206.168.192.in-addr.arpa"                                            7248
     9 "HPE8AA67"                                                               6929
    10 "ISATAP"                                                                 6569

1.  Опеределите базовые статистические характеристики (функция summary()
    ) интервала времени между последовательными обращениями к топ-10
    доменам.

``` r
top_10_domains_list <- head(domain_counts, 10) %>% pull(query)

interval_data <- dns_data2 %>%
  filter(query %in% top_10_domains_list) %>%
  arrange(query, timestamp) %>%
  group_by(query) %>%
  mutate(
    time_diff = lead(timestamp) - timestamp
  ) %>%
  filter(!is.na(time_diff)) %>%
  ungroup()

summary_results <- interval_data %>%
    group_by(query) %>%
    summarise(
      Min = min(time_diff),
      Q1 = quantile(time_diff, 0.25),
      Median = median(time_diff),
      Mean = mean(time_diff),
      Q3 = quantile(time_diff, 0.75),
      Max = max(time_diff),
      Units = unique(units(time_diff))
    )

print(summary_results)
```

    # A tibble: 10 × 8
       query                              Min   Q1    Median Mean  Q3    Max   Units
       <chr>                              <drt> <drt> <drtn> <drt> <drt> <drt> <chr>
     1 "*\\x00\\x00\\x00\\x00\\x00\\x00\… 0 se… 0.14… 0.500… 11.2…  1.5… 5272… secs 
     2 "44.206.168.192.in-addr.arpa"      0 se… 2.08… 4.000… 16.0… 20.0… 4967… secs 
     3 "HPE8AA67"                         0 se… 0.75… 0.750… 16.6… 25.4… 5004… secs 
     4 "ISATAP"                           0 se… 0.75… 0.759… 17.4…  1.0… 5199… secs 
     5 "WPAD"                             0 se… 0.75… 0.750… 12.6…  1.1… 5004… secs 
     6 "safebrowsing.clients.google.com"  0 se… 0.00… 1.000… 10.0…  2.0… 4995… secs 
     7 "teredo.ipv6.microsoft.com"        0 se… 0.00… 0.000…  2.9…  0.5… 5038… secs 
     8 "time.apple.com"                   0 se… 0.36… 1.760…  8.6…  4.7… 5092… secs 
     9 "tools.google.com"                 0 se… 0.00… 0.000…  8.1…  1.0… 5036… secs 
    10 "www.apple.com"                    0 se… 0.00… 1.000…  8.6…  3.0… 5096… secs 

1.  Часто вредоносное программное обеспечение использует DNS канал в
    качестве канала управления, периодически отправляя запросы на
    подконтрольный злоумышленникам DNS сервер. По периодическим запросам
    на один и тот же домен можно выявить скрытый DNS канал. Есть ли
    такие IP адреса в исследуемом датасете?

``` r
library(stats)
MIN_REQUESTS <- 10

potential_tunnels <- dns_data2 %>%
  select(timestamp, source_ip, query) %>%
  filter(!is.na(query), !is.na(source_ip)) %>%
  group_by(source_ip, query) %>%
  filter(n() >= MIN_REQUESTS) %>%
  arrange(timestamp) %>%
  mutate(
    time_interval = as.numeric(timestamp - lag(timestamp), units = "secs")
  ) %>%
  filter(!is.na(time_interval)) %>%
  ungroup()

tunnel_statistics <- potential_tunnels %>%
  group_by(source_ip, query) %>%
  summarise(
    total_requests = n() + 1,
    mean_interval_sec = round(mean(time_interval), 2),
    sd_interval_sec = round(sd(time_interval), 2),
    cv_ratio = sd_interval_sec / mean_interval_sec,
    
    .groups = 'drop'
  ) %>%
  arrange(desc(total_requests), cv_ratio)

potential_tunneling_ips <- tunnel_statistics %>%
  filter(cv_ratio < 0.1) %>%
  select(source_ip, query, total_requests, mean_interval_sec, sd_interval_sec, cv_ratio)

print(potential_tunneling_ips %>% pull(source_ip) %>% unique())
```

     [1] "192.168.202.94"  "192.168.203.64"  "192.168.202.92"  "192.168.22.25"  
     [5] "192.168.229.252" "192.168.202.100" "192.168.202.112" "192.168.202.79" 
     [9] "192.168.203.45"  "192.168.202.132" "192.168.202.89" 

1.  Определите местоположение (страну, город) и организацию-провайдера
    для топ-10 доменов. Для этого можно использовать сторонние сервисы,
    например http://ip-api.com (API-эндпоинт – http://ip-api.com/json).

``` r
get_geo_info <- function(ip) {
  if (is.na(ip) || ip == "") {
     return(tibble(
      ip_address = NA_character_,
      country = NA,
      city = NA,
      isp = NA
    ))
  }
  if (grepl("^(10\\.|192\\.168\\.|172\\.(1[6-9]|2[0-9]|3[0-1])\\.)", ip)) {
    return(tibble(
      ip_address = ip,
      country = "Internal IP",
      city = "Internal IP",
      isp = "Internal IP"
    ))
  }
  url <- paste0("http://ip-api.com/json/", ip)
  response <- GET(url)
  if (status_code(response) == 200) {
    data <- fromJSON(content(response, "text"))
    if (data$status == "success") {
      return(tibble(
        ip_address = ip,
        country = data$country,
        city = data$city,
        isp = data$isp
      ))
    } else {
      return(tibble(
        ip_address = ip,
        country = paste("API error"),
        city = paste("API errors"),
        isp = paste("API error")
      ))
    }
  } else {
    return(tibble(
      ip_address = ip,
      country = "API error",
      city = "API error",
      isp = "API error"
    ))
  }
}

dns_with_dest_ip <- dns_data2 %>%
  filter(!is.na(destination_ip)) %>%
  select(query, destination_ip) %>%
  distinct()
top_10_domains <- dns_data2%>%count(query, sort = TRUE) %>%
  as_tibble() %>% head(10)
top_10_domains
```

    # A tibble: 10 × 2
       query                                                                       n
       <chr>                                                                   <int>
     1 "teredo.ipv6.microsoft.com"                                             39273
     2 "tools.google.com"                                                      14057
     3 "www.apple.com"                                                         13390
     4 "time.apple.com"                                                        13109
     5 "safebrowsing.clients.google.com"                                       11658
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x… 10401
     7 "WPAD"                                                                   9134
     8 "44.206.168.192.in-addr.arpa"                                            7248
     9 "HPE8AA67"                                                               6929
    10 "ISATAP"                                                                 6569

``` r
relevant_dns <- dns_with_dest_ip %>%
  filter(query %in% top_10_domains$query)
geo_results_df <- tibble(
  ip_address = character(),
  country = character(),
  city = character(),
  isp = character()
)
unique_ips_to_check <- unique(relevant_dns$destination_ip)
for (ip in unique_ips_to_check) {
  geo_info_row <- get_geo_info(ip)
  geo_results_df <- bind_rows(geo_results_df, geo_info_row)
}
domain_geo_info_final <- relevant_dns %>%
  left_join(geo_results_df, by = c("destination_ip" = "ip_address")) %>%
  rename(ip_address = destination_ip) %>%
  select(domain = query, ip_address, country, city, isp)
domain_order_factor <- factor(domain_geo_info_final$domain, levels = top_10_domains$query)
domain_geo_info_final_sorted <- domain_geo_info_final %>%
  mutate(domain_order = domain_order_factor) %>%
  arrange(domain_order) %>%
  select(-domain_order)
print(domain_geo_info_final_sorted)
```

    # A tibble: 1,213 × 5
       domain                    ip_address       country       city        isp     
       <chr>                     <chr>            <chr>         <chr>       <chr>   
     1 teredo.ipv6.microsoft.com fec0:0:0:ffff::2 Switzerland   Morat       Interne…
     2 teredo.ipv6.microsoft.com fec0:0:0:ffff::1 Switzerland   Morat       Interne…
     3 teredo.ipv6.microsoft.com fec0:0:0:ffff::3 Switzerland   Morat       Interne…
     4 teredo.ipv6.microsoft.com 192.168.207.4    Internal IP   Internal IP Interna…
     5 teredo.ipv6.microsoft.com 192.168.0.1      Internal IP   Internal IP Interna…
     6 tools.google.com          192.168.207.4    Internal IP   Internal IP Interna…
     7 tools.google.com          192.168.206.44   Internal IP   Internal IP Interna…
     8 tools.google.com          156.154.70.22    United States New York    Neustar…
     9 tools.google.com          8.26.56.26       United States Clifton     Flexent…
    10 tools.google.com          68.87.75.198     United States Pittsburgh  Comcast…
    # ℹ 1,203 more rows

## Оценка результата

В результате лабораторной работы мы проанализировали DNS трафик Доброй
Организации с помощью языка R.

## Вывод

Таким образом, мы развили практические навыки использования языка
программирования R для обработки данных, закрепили знания базовых типов
данных языка R, развили практические навыки использования функций
обработки DNS трафика.
