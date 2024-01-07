
## Packages

``` r
# install.packages("devtools")
devtools::install_github("PedroTL/RefineGeo")
library(tidygeocoder)
library(knitr)
library(dplyr)
```

## Geocoding Data Preparation

To start, we can acquire a sample data frame from the `RefineGeo`
package. Typically, address information is distributed across separate
columns. Here, we will assemble a complete address, which will serve as
the input for Geocoding across three distinct services.

``` r
sample_address <- RefineGeo::sample_address

knitr::kable(head(sample_address), align = "c")
```

|                       address                       |   cep    | municipality | state | country |
|:---------------------------------------------------:|:--------:|:------------:|:-----:|:-------:|
|            Avenida A Distrito Industrial            | 13054712 |   Campinas   |  SP   | Brasil  |
|                 Rua A Novo Taquaral                 | 13077109 |   Campinas   |  SP   | Brasil  |
|             Viela A Jardim Metonopolis              | 13058451 |   Campinas   |  SP   | Brasil  |
| Rua A Loteamento Claude de Barros Penteado (Sousas) | 13105244 |   Campinas   |  SP   | Brasil  |
|               Rua A Jardim Rosalia IV               | 13067750 |   Campinas   |  SP   | Brasil  |
|             Rua A Chacaras Sao Martinho             | 13042830 |   Campinas   |  SP   | Brasil  |

The address information usually can be giving in separate columns. Here
we are going to make a complete address, being the input for Geocoding
in three different services.

``` r
# Creating a full address as input
sample_address <- sample_address %>%
  mutate(input_addr = RefineGeo::clean_address(paste(address,
                                                     municipality,
                                                     state,
                                                     cep,
                                                     country,
                                                     sep = " ")))

knitr::kable(head(sample_address), align = "c")
```

|                       address                       |   cep    | municipality | state | country |                                  input_addr                                   |
|:---------------------------------------------------:|:--------:|:------------:|:-----:|:-------:|:-----------------------------------------------------------------------------:|
|            Avenida A Distrito Industrial            | 13054712 |   Campinas   |  SP   | Brasil  |           AVENIDA A DISTRITO INDUSTRIAL CAMPINAS SP 13054712 BRASIL           |
|                 Rua A Novo Taquaral                 | 13077109 |   Campinas   |  SP   | Brasil  |                RUA A NOVO TAQUARAL CAMPINAS SP 13077109 BRASIL                |
|             Viela A Jardim Metonopolis              | 13058451 |   Campinas   |  SP   | Brasil  |            VIELA A JARDIM METONOPOLIS CAMPINAS SP 13058451 BRASIL             |
| Rua A Loteamento Claude de Barros Penteado (Sousas) | 13105244 |   Campinas   |  SP   | Brasil  | RUA A LOTEAMENTO CLAUDE DE BARROS PENTEADO SOUSAS CAMPINAS SP 13105244 BRASIL |
|               Rua A Jardim Rosalia IV               | 13067750 |   Campinas   |  SP   | Brasil  |              RUA A JARDIM ROSALIA IV CAMPINAS SP 13067750 BRASIL              |
|             Rua A Chacaras Sao Martinho             | 13042830 |   Campinas   |  SP   | Brasil  |            RUA A CHACARAS SAO MARTINHO CAMPINAS SP 13042830 BRASIL            |

### Geocoding Combination

Next, we’ll utilize `tidygeocoder::geocode_combine` to extract Latitude,
Longitude, and output addresses from three different Geocoding services:
Arcgis, Bing, and Here.

``` r
response <- 
  tidygeocoder::geocode_combine(
  sample_address,
  queries = list(
    list(method = 'arcgis', mode = 'single'),
    list(method = 'bing', mode = 'single'),
    list(method = 'here', mode = 'single')
  ),
  global_params = list(address = 'input_addr', full_results = TRUE),
  cascade = FALSE,
  return_list = TRUE
)
```

For Bing and Here services, it’s necessary to set an API key in the
environment. You can use `usethis::edit_r_environ()` to set
`BINGMAPS_API_KEY = "Your_API_KEY"` and `HERE_API_KEY = "Your_API_KEY"`.
When dealing with multiple addresses, consider using the
`mode = 'batch'` argument and refer to
`tidygeocoder::batch_limit_reference`. Approximately 3 minutes are
required to obtain full results for all Geocoding services when dealing
with 100 addresses.

### Retrieving Responses for Each Service

From the responses, our primary interest lies in obtaining the Latitude,
Longitude, and the corresponding returned address.

``` r
arcgis_response <- response$arcgis
arcgis_response <- arcgis_response %>%
  select("input_addr", "input_cep" = "cep", "output_addr_1" = "arcgis_address", "lon1" = "location.x", "lat1" = "location.y") %>%
  as.data.frame()

knitr::kable(head(arcgis_response), align = "c")
```

|                                  input_addr                                   | input_cep |              output_addr_1               |   lon1    |   lat1    |
|:-----------------------------------------------------------------------------:|:---------:|:----------------------------------------:|:---------:|:---------:|
|           AVENIDA A DISTRITO INDUSTRIAL CAMPINAS SP 13054712 BRASIL           | 13054712  |        13054, Campinas, São Paulo        | -47.12534 | -22.99891 |
|                RUA A NOVO TAQUARAL CAMPINAS SP 13077109 BRASIL                | 13077109  | Rua Nove, Campinas, São Paulo, 13086-700 | -47.04187 | -22.79913 |
|            VIELA A JARDIM METONOPOLIS CAMPINAS SP 13058451 BRASIL             | 13058451  |        13058, Campinas, São Paulo        | -47.19283 | -22.94312 |
| RUA A LOTEAMENTO CLAUDE DE BARROS PENTEADO SOUSAS CAMPINAS SP 13105244 BRASIL | 13105244  |    13105, Sousas, Campinas, São Paulo    | -46.97759 | -22.88881 |
|              RUA A JARDIM ROSALIA IV CAMPINAS SP 13067750 BRASIL              | 13067750  |  Rua A, Campinas, São Paulo, 13067-750   | -47.15228 | -22.87627 |
|            RUA A CHACARAS SAO MARTINHO CAMPINAS SP 13042830 BRASIL            | 13042830  |        13042, Campinas, São Paulo        | -47.04266 | -22.96936 |

This process is then repeated for the other two services.

``` r
bing_response <- response$bing
bing_response <- bing_response %>%
  select("input_addr", "input_cep" = "cep", "output_addr_2" = "name", "lon2" = "long", "lat2" = "lat") %>%
  as.data.frame()

knitr::kable(head(bing_response), align = "c")
```

|                                  input_addr                                   | input_cep |                                  output_addr_2                                   |   lon2    |   lat2    |
|:-----------------------------------------------------------------------------:|:---------:|:--------------------------------------------------------------------------------:|:---------:|:---------:|
|           AVENIDA A DISTRITO INDUSTRIAL CAMPINAS SP 13054712 BRASIL           | 13054712  |                                      Brazil                                      | -53.20000 | -10.33333 |
|                RUA A NOVO TAQUARAL CAMPINAS SP 13077109 BRASIL                | 13077109  |      Rua Novo Horizonte, Taquaral, Campinas - São Paulo, 13090-670, Brazil       | -47.04456 | -22.88920 |
|            VIELA A JARDIM METONOPOLIS CAMPINAS SP 13058451 BRASIL             | 13058451  |                                13058-451, Brazil                                 | -47.16528 | -22.98058 |
| RUA A LOTEAMENTO CLAUDE DE BARROS PENTEADO SOUSAS CAMPINAS SP 13105244 BRASIL | 13105244  | Rodovia Doutor Heitor Penteado, Sousas, Campinas - São Paulo, 13105-000, Brazil  | -46.99386 | -22.89283 |
|              RUA A JARDIM ROSALIA IV CAMPINAS SP 13067750 BRASIL              | 13067750  |               Rua A, Jardim Santiago, Campinas - São Paulo, Brazil               | -47.15225 | -22.87631 |
|            RUA A CHACARAS SAO MARTINHO CAMPINAS SP 13042830 BRASIL            | 13042830  | Rua A & Avenida Martinho Lutero, Ouro Verde, Campinas - São Paulo, 13056, Brazil | -47.14041 | -22.98175 |

``` r
here_response <- response$here
here_response <- here_response %>%
  select("input_addr", "input_cep" = "cep", "output_addr_3" = "here_address.label", "output_addr_cep_3" = "here_address.postalCode" ,"lon3" = "long", "lat3" = "lat") %>%
  as.data.frame()

knitr::kable(head(here_response), align = "c")
```

|                                  input_addr                                   | input_cep |                                        output_addr_3                                        | output_addr_cep_3 |   lon3    |   lat3    |
|:-----------------------------------------------------------------------------:|:---------:|:-------------------------------------------------------------------------------------------:|:-----------------:|:---------:|:---------:|
|           AVENIDA A DISTRITO INDUSTRIAL CAMPINAS SP 13054712 BRASIL           | 13054712  |                SP-075, Distrito Industrial, Campinas - SP, 13050-009, Brasil                |     13050-009     | -47.11482 | -23.00864 |
|                RUA A NOVO TAQUARAL CAMPINAS SP 13077109 BRASIL                | 13077109  |                               Taquaral, Campinas, SP, Brasil                                |     13076-061     | -47.05498 | -22.88508 |
|            VIELA A JARDIM METONOPOLIS CAMPINAS SP 13058451 BRASIL             | 13058451  |                          Jardim Metonópolis, Campinas, SP, Brasil                           |     13058-450     | -47.18618 | -22.95159 |
| RUA A LOTEAMENTO CLAUDE DE BARROS PENTEADO SOUSAS CAMPINAS SP 13105244 BRASIL | 13105244  |          Rua Nazário Basílio de Almeida, Sousas, Campinas - SP, 13105-617, Brasil           |     13105-617     | -46.97582 | -22.88952 |
|              RUA A JARDIM ROSALIA IV CAMPINAS SP 13067750 BRASIL              | 13067750  |                           Rua A, Campinas - SP, 13067-750, Brasil                           |     13067-750     | -47.15227 | -22.87627 |
|            RUA A CHACARAS SAO MARTINHO CAMPINAS SP 13042830 BRASIL            | 13042830  | Chácaras São Martinho, Rua Guilherme Herculano P. de Cam., Campinas - SP, 13042-836, Brasil |     13042-836     | -47.03497 | -22.97315 |

### Compiling a Final Dataset

``` r
sample_address_geo <-
  plyr::join_all(list(arcgis_response,
                      bing_response,
                      here_response),
                 by = c("input_addr", "input_cep"),
                 type = 'left') %>%
  mutate("output_addr_cep_1" = RefineGeo::extr_cep(output_addr_1),
         "output_addr_cep_2" = RefineGeo::extr_cep(output_addr_2),
         "output_addr_cep_3" = RefineGeo::extr_cep(output_addr_cep_3),
         "input_cep" = as.character(input_cep),
         "output_addr_1" = RefineGeo::clean_address(output_addr_1),
         "output_addr_2" = RefineGeo::clean_address(output_addr_2),
         "output_addr_3" = RefineGeo::clean_address(output_addr_3)) %>%
  select("input_addr", "input_cep", "output_addr_1", "output_addr_2", "output_addr_3", "output_addr_cep_1", "output_addr_cep_2", "output_addr_cep_3", "lat1", "lon1", "lat2", "lon2", "lat3", "lon3")

knitr::kable(head(sample_address_geo), align = "c")
```

|                                  input_addr                                   | input_cep |            output_addr_1             |                              output_addr_2                               |                                   output_addr_3                                    | output_addr_cep_1 | output_addr_cep_2 | output_addr_cep_3 |   lat1    |   lon1    |   lat2    |   lon2    |   lat3    |   lon3    |
|:-----------------------------------------------------------------------------:|:---------:|:------------------------------------:|:------------------------------------------------------------------------:|:----------------------------------------------------------------------------------:|:-----------------:|:-----------------:|:-----------------:|:---------:|:---------:|:---------:|:---------:|:---------:|:---------:|
|           AVENIDA A DISTRITO INDUSTRIAL CAMPINAS SP 13054712 BRASIL           | 13054712  |       13054 CAMPINAS SAO PAULO       |                                  BRAZIL                                  |               SP075 DISTRITO INDUSTRIAL CAMPINAS SP 13050009 BRASIL                |        NA         |        NA         |     13050009      | -22.99891 | -47.12534 | -10.33333 | -53.20000 | -23.00864 | -47.11482 |
|                RUA A NOVO TAQUARAL CAMPINAS SP 13077109 BRASIL                | 13077109  | RUA NOVE CAMPINAS SAO PAULO 13086700 |      RUA NOVO HORIZONTE TAQUARAL CAMPINAS SAO PAULO 13090670 BRAZIL      |                            TAQUARAL CAMPINAS SP BRASIL                             |     13086700      |     13090670      |     13076061      | -22.79913 | -47.04187 | -22.88920 | -47.04456 | -22.88508 | -47.05498 |
|            VIELA A JARDIM METONOPOLIS CAMPINAS SP 13058451 BRASIL             | 13058451  |       13058 CAMPINAS SAO PAULO       |                             13058451 BRAZIL                              |                       JARDIM METONOPOLIS CAMPINAS SP BRASIL                        |        NA         |     13058451      |     13058450      | -22.94312 | -47.19283 | -22.98058 | -47.16528 | -22.95159 | -47.18618 |
| RUA A LOTEAMENTO CLAUDE DE BARROS PENTEADO SOUSAS CAMPINAS SP 13105244 BRASIL | 13105244  |   13105 SOUSAS CAMPINAS SAO PAULO    | RODOVIA DOUTOR HEITOR PENTEADO SOUSAS CAMPINAS SAO PAULO 13105000 BRAZIL |         RUA NAZARIO BASILIO DE ALMEIDA SOUSAS CAMPINAS SP 13105617 BRASIL          |        NA         |     13105000      |     13105617      | -22.88881 | -46.97759 | -22.89283 | -46.99386 | -22.88952 | -46.97582 |
|              RUA A JARDIM ROSALIA IV CAMPINAS SP 13067750 BRASIL              | 13067750  |  RUA A CAMPINAS SAO PAULO 13067750   |             RUA A JARDIM SANTIAGO CAMPINAS SAO PAULO BRAZIL              |                         RUA A CAMPINAS SP 13067750 BRASIL                          |     13067750      |        NA         |     13067750      | -22.87627 | -47.15228 | -22.87631 | -47.15225 | -22.87627 | -47.15227 |
|            RUA A CHACARAS SAO MARTINHO CAMPINAS SP 13042830 BRASIL            | 13042830  |       13042 CAMPINAS SAO PAULO       | RUA A AVENIDA MARTINHO LUTERO OURO VERDE CAMPINAS SAO PAULO 13056 BRAZIL | CHACARAS SAO MARTINHO RUA GUILHERME HERCULANO P DE CAM CAMPINAS SP 13042836 BRASIL |        NA         |        NA         |     13042836      | -22.96936 | -47.04266 | -22.98175 | -47.14041 | -22.97315 | -47.03497 |

It’s noteworthy that the `Here` Geocoding service provides the
`PostCode` column already, requiring minimal cleaning. Depending on the
chosen service, it’s advisable to review all outputs before selecting
variables. In essence, the optimal data set structure is outlined. Now,
`RefineGeo` can be utilized to gain a more accurate perspective on the
quality of the provided coordinates.

This data is available in `RefineGeo::sample_address_geo`
