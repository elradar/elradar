---
title: 'Un CRAN a la vez: janitor'
author: ~
date: '2019-10-19'
slug: un-cran-a-la-vez-janitor
categories: [uncranalavez, janitor]
tags: [limpieza, etl, datawrangling]
publishdate: '2019-10-19T16:38:25-03:00'
comments: yes
---

Así damos inicio a la sección dada a llamar **Un CRAN a la vez** (inspirada en [este proyecto](https://djnavarro.net/post/a-random-walk-on-cran/) de la gran Danielle Navarro), mostrando lo que hace una simpática biblioteca llamada janitor, hecha por Sam Firke y subida [aqui](https://github.com/sfirke/janitor), en conjunción con nuestro querido tidyverse.


```{r}

library(tidyverse)
library(janitor)

```
janitor es una biblioteca que ayuda a limpiar nombres de columnas que vienen formateados de forma subóptima (para ser generosos) para el análisis de datos. Generalmente sirve para aquellos Excel que nos pasan con nombres de columna con tildes, puntos como separadores, duplicados, entre otros problemillas. 


![](https://pbs.twimg.com/media/ED1UxS4U8AEh31k?format=jpg&name=medium)


Veamos un ejemplo (real) de un dataset, truncado por temas de confidencialidad, en el que una biblioteca como janitor nos vino al pelo.


```{r levanto la base, message=FALSE, warning=FALSE}


datos_originales <- read.csv("https://elradar.github.io/data/data_janitor.csv")

```

Al husmear la estructura del dataset, vemos que las columnas *Género* y *Teléfono* tienen tilde, lo cual es muy correcto a nivel idiomático pero muy desaconsejado a nivel pragmático ya que tener caracteres especiales en el nombre de columnas puede afectar muchas operaciones que se encuentran penadas y adaptadas al inglés, o el esperanto de la era digital.
Por otro lado, podemos ver que en muchas de las columnas han decidido usar el punto como separador. Esto tiene más que ver con convención (o hedonismo estético) que por utilidad: pero en general se prefiere utilizar el guión bajo o underscore como separador. Por último, que todo esté en minúscula no nos vendría del todo mal tampoco en caso de que querramos ahorrar tiempo si tipeamos (el viaje de ida y vuelta al Caps Lock).

```{r nombres_columnas, warning=FALSE}

datos_limpios <- datos_originales %>% 
  clean_names()
  
colnames(datos_limpios)

```

 Lo primero que salta a la vista es que janitor y tidyverse se llevan increíblemente bien. Lo segundo, es que con una simple función todas las variables perdieron los caracteres especiales (en nuestro caso las tildes), los puntos fueron reemplazador por "_" como separadores y todo está en bella letra minúscula.
 Vamos a ver si hay filas enteramente vacías (todo es posible cuando viene de Excel), que tengan siempre el mismo valor (categoría constante) y si hay alguna línea duplicada.
 
```{r remuevo_filas, message=FALSE, warning=FALSE}

datos_limpios <- datos_originales %>% 
  clean_names() %>% 
  remove_empty("rows") %>% 
  remove_constant() %>% 
  convert_to_NA("factor")

duplicados <- datos_limpios %>% 
  get_dupes()


dim(datos_limpios)
dim(duplicados)

```
 
No tenemos filas vacías ni duplicados (tenemos la misma cantidad de observaciones) aunque sí encontró una fila con todos valores constantes y la removió (fecha_ult_ingreso). 



```{r recodear, message=FALSE, warning=FALSE}

datos_limpios <- datos_limpios %>% 
  mutate(genero = recode(genero,
                           "A" = "hombre",
                           "B" = "mujer"),
         apoyo_espiritual = recode(apoyo_espiritual,
                           "A" = "no quiso",
                           "B" = "tuvo apoyo"))

```



Una gran ventaja de janitor es que nos permite además es hacer tablas relativamente lindas de manera sencilla y pipe compatible. Vamos a cruzar la variable género con apoyo espiritual.


```{r tablas, message=FALSE, warning=FALSE}

datos_limpios %>% 
  tabyl(genero, apoyo_espiritual)

```

Genial, tenemos la cantidad de casos en cada celda. Pero, ¿no nos interesaría mucho más poder ver estos mismos datos como porcentajes?


```{r tablas_relativas, message=FALSE, warning=FALSE}

datos_limpios %>% 
  tabyl(genero, apoyo_espiritual) %>% 
  adorn_totals("row") %>%   ##muestra el total por categoria de columna
  adorn_percentages("row") %>% ##nos muestra las celdas en porcentajes
  adorn_pct_formatting() %>% ##formatea el porcentaje de manera legible
  adorn_ns() %>% ##le suma a cada celda el número de casos absolutos entre paréntesis
  adorn_title() ##le suma el título del cruce en la esquina superior izquierda
  
  
```


Y así tenemos una tabla bastante linda, con bastante margen de formateo y sin salir del universo tidyverse (yey). Por supuesto que no sacaremos ninguna conclusión sobre estos datos ya que forman parte de una muestra no aleatoria ni representativa de un conjunto de información no disponible (aún) para su publicación. 

**BONUS TRACK**: Otra joyita que tiene esta biblioteca janitor es la función excel_numeric_to_date() para hacer ingeniería inversa de aquellas fechas de Excel que llegan como enteros a R (si les pasó alguna vez reconocerán rápido este problema)