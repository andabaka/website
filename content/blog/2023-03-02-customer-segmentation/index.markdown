---
title: Customer segmentation
author: Marijana Andabaka
date: '2023-03-02'
slug: []
categories: []
tags: []
excerpt: "Segmentiranje kupaca pomoću klaster analize, k-means algoritma i vizualizacija klastera sa UMAP algoritmom "
---

<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<link href="{{< blogdown/postref >}}index_files/datatables-css/datatables-crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/datatables-binding/datatables.js"></script>
<script src="{{< blogdown/postref >}}index_files/jquery/jquery-3.6.0.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/dt-core/css/jquery.dataTables.min.css" rel="stylesheet" />
<link href="{{< blogdown/postref >}}index_files/dt-core/css/jquery.dataTables.extra.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/dt-core/js/jquery.dataTables.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/crosstalk/css/crosstalk.min.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/crosstalk/js/crosstalk.min.js"></script>
<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<script src="{{< blogdown/postref >}}index_files/plotly-binding/plotly.js"></script>
<script src="{{< blogdown/postref >}}index_files/typedarray/typedarray.min.js"></script>
<script src="{{< blogdown/postref >}}index_files/jquery/jquery.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/crosstalk/css/crosstalk.min.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/crosstalk/js/crosstalk.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/plotly-htmlwidgets-css/plotly-htmlwidgets.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/plotly-main/plotly-latest.min.js"></script>
<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<link href="{{< blogdown/postref >}}index_files/datatables-css/datatables-crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/datatables-binding/datatables.js"></script>
<script src="{{< blogdown/postref >}}index_files/jquery/jquery-3.6.0.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/dt-core/css/jquery.dataTables.min.css" rel="stylesheet" />
<link href="{{< blogdown/postref >}}index_files/dt-core/css/jquery.dataTables.extra.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/dt-core/js/jquery.dataTables.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/crosstalk/css/crosstalk.min.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/crosstalk/js/crosstalk.min.js"></script>

<img src="img/mall.jpg" width="100%" style="display: block; margin: auto;" />

## UVOD

Segmentacija je proces podjele u segmente (grupe). Podjela može biti prema bihevioralnim, demografskim, zemljopisnim i drugim karakteristikama. Segmenti bi trebali biti relativno homogeni unutar sebe i heterogeni u odnosu na druge. Glavni cilj pristupa je pronaći bazu visoko profitabilnih i niskoprofitabilnih kupaca/klijenata te različitim marketinškim strategijama ciljati svaku skupinu kako bi se izvukla maksimalna vrijednost[^1].

Prednosti primjene segmentacije kupaca u poslovanju su: učinkovitije marketinške strategije, predviđanje ponašanja kupaca, poboljšanje lojalnosti i zadržavanje kupaca, uspješan razvoj proizvoda, optimizacija cijena.

Za segmentaciju je korištena klaster analiza te je primijenjen algoritam *k*-sredina (engl. *k-means*). Ovo je jedan od najčešće korištenih algoritama nenadziranog (engl. *unsupervised*) strojnog učenja (engl. *machine learning*).

Algoritam klasificira skup podataka u *k* grupe (klastere) gdje *k* predstavlja broj grupa koji se unaprijed određuje. Svaki klaster je predstavljen svojim centrom (centroidom) koje odgovara sredini točaka dodijeljenih klasteru[^2].

Za vizulizaciju konačnih rezultata segmentacije kupaca koristit će se algoritam UMAP (engl. *Uniform Manifold Approximation and Projection*). UMAP omugućuje prikaz višedimenzionalnih podataka unutar dvije dimenzije[^3].

Proces provedbe projekta temelji se na CRISP-DM metodologiji (engl. *CRoss Industry Standard Process for Data Mining*)[^4].

## ANALIZA

U ovom projektu cilj je segmentirati kupce koji su izvršili kupovinu u trgovačkom centru u klastere (grupe) na temelju podataka o spolu, dobi, godišnjim prihodima i potrošačkom kodu. Pojedinu grupu će u konačnici predstavljati kupci sličnih osobina kojima se onda ciljano pristupa.

### Učitavanje paketa

Za provedbu projekta učitani su potrebni paketi.

``` r
library(tidyverse)
library(tidyquant)
library(broom)
library(umap)
library(ggrepel)
library(plotly)
library(DT)
```

### Baza podataka

Baza podataka trgovačkog centra sadrži osnovne informacije o kupcima kao što su ID Kupca, Dob, Godišnji prihodi (1000\$) i Potrošački kod (1-100). Potrošački kod (engl. *Spending Score*) se dodjeljuje svakom kupcu na temelju određenih parametara kao što npr. podaci o potrošnji. Baza se sastoji od 200 individulanih jedinica upisa (redaka)[^5].

``` r
# Unos baze podataka
mall_customers_tbl<-readr::read_csv("Data/Mall_Customers.csv")

# Pogled na podatke
mall_customers_tbl %>% glimpse()
```

    ## Rows: 200
    ## Columns: 5
    ## $ CustomerID               <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14…
    ## $ Gender                   <chr> "Male", "Male", "Female", "Female", "Female",…
    ## $ Age                      <dbl> 19, 21, 20, 23, 31, 22, 35, 23, 64, 30, 67, 3…
    ## $ `Annual Income (k$)`     <dbl> 15, 15, 16, 16, 17, 17, 18, 18, 19, 19, 19, 1…
    ## $ `Spending Score (1-100)` <dbl> 39, 81, 6, 77, 40, 76, 6, 94, 3, 72, 14, 99, …

``` r
# Prijevod i formatiranje varijable spol u faktorski oblik) 
mall_customers_tbl <- mall_customers_tbl %>% 
    set_names(c("ID Kupca", "Spol", "Dob", "Godišnji Prihodi(1000$)", 
                "Potrošački kod(1-100)")) %>% 
    mutate(Spol = str_replace_all(Spol, "Male", "Muško")) %>% 
    mutate(Spol = str_replace_all(Spol, "Female", "Žensko")) %>% 
    mutate(Spol = Spol %>% as.factor())

# Interaktivna baza kupaca trgovačkog centra
datatable(mall_customers_tbl, caption = htmltools::tags$caption(
    style = 'caption-side: top; text-align: center;',
    'Tablica 1: ', htmltools::em('Interaktivna baza kupaca trgovačkog centra '))
    )
```

<div id="htmlwidget-1" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"filter":"none","vertical":false,"caption":"<caption style=\"caption-side: top; text-align: center;\">\n  Tablica 1: \n  <em>Interaktivna baza kupaca trgovačkog centra <\/em>\n<\/caption>","data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31","32","33","34","35","36","37","38","39","40","41","42","43","44","45","46","47","48","49","50","51","52","53","54","55","56","57","58","59","60","61","62","63","64","65","66","67","68","69","70","71","72","73","74","75","76","77","78","79","80","81","82","83","84","85","86","87","88","89","90","91","92","93","94","95","96","97","98","99","100","101","102","103","104","105","106","107","108","109","110","111","112","113","114","115","116","117","118","119","120","121","122","123","124","125","126","127","128","129","130","131","132","133","134","135","136","137","138","139","140","141","142","143","144","145","146","147","148","149","150","151","152","153","154","155","156","157","158","159","160","161","162","163","164","165","166","167","168","169","170","171","172","173","174","175","176","177","178","179","180","181","182","183","184","185","186","187","188","189","190","191","192","193","194","195","196","197","198","199","200"],[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200],["Muško","Muško","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Žensko","Muško","Muško","Žensko","Muško","Muško","Žensko","Muško","Muško","Žensko","Muško","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Muško","Žensko","Muško","Muško","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Muško","Muško","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Muško","Žensko","Muško","Žensko","Muško","Muško","Muško","Žensko","Žensko","Muško","Muško","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Žensko","Muško","Muško","Žensko","Muško","Žensko","Žensko","Muško","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Žensko","Žensko","Žensko","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Muško","Žensko","Žensko","Muško","Muško","Muško","Žensko","Žensko","Muško","Muško","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Muško","Muško","Muško","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Muško","Muško","Žensko","Žensko","Muško","Muško","Muško","Žensko","Žensko","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Muško","Muško","Muško","Muško","Muško","Žensko","Žensko","Muško","Muško","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Žensko","Žensko","Muško","Žensko","Žensko","Žensko","Žensko","Muško","Muško","Muško"],[19,21,20,23,31,22,35,23,64,30,67,35,58,24,37,22,35,20,52,35,35,25,46,31,54,29,45,35,40,23,60,21,53,18,49,21,42,30,36,20,65,24,48,31,49,24,50,27,29,31,49,33,31,59,50,47,51,69,27,53,70,19,67,54,63,18,43,68,19,32,70,47,60,60,59,26,45,40,23,49,57,38,67,46,21,48,55,22,34,50,68,18,48,40,32,24,47,27,48,20,23,49,67,26,49,21,66,54,68,66,65,19,38,19,18,19,63,49,51,50,27,38,40,39,23,31,43,40,59,38,47,39,25,31,20,29,44,32,19,35,57,32,28,32,25,28,48,32,34,34,43,39,44,38,47,27,37,30,34,30,56,29,19,31,50,36,42,33,36,32,40,28,36,36,52,30,58,27,59,35,37,32,46,29,41,30,54,28,41,36,34,32,33,38,47,35,45,32,32,30],[15,15,16,16,17,17,18,18,19,19,19,19,20,20,20,20,21,21,23,23,24,24,25,25,28,28,28,28,29,29,30,30,33,33,33,33,34,34,37,37,38,38,39,39,39,39,40,40,40,40,42,42,43,43,43,43,44,44,46,46,46,46,47,47,48,48,48,48,48,48,49,49,50,50,54,54,54,54,54,54,54,54,54,54,54,54,57,57,58,58,59,59,60,60,60,60,60,60,61,61,62,62,62,62,62,62,63,63,63,63,63,63,64,64,65,65,65,65,67,67,67,67,69,69,70,70,71,71,71,71,71,71,72,72,73,73,73,73,74,74,75,75,76,76,77,77,77,77,78,78,78,78,78,78,78,78,78,78,78,78,79,79,81,81,85,85,86,86,87,87,87,87,87,87,88,88,88,88,93,93,97,97,98,98,99,99,101,101,103,103,103,103,113,113,120,120,126,126,137,137],[39,81,6,77,40,76,6,94,3,72,14,99,15,77,13,79,35,66,29,98,35,73,5,73,14,82,32,61,31,87,4,73,4,92,14,81,17,73,26,75,35,92,36,61,28,65,55,47,42,42,52,60,54,60,45,41,50,46,51,46,56,55,52,59,51,59,50,48,59,47,55,42,49,56,47,54,53,48,52,42,51,55,41,44,57,46,58,55,60,46,55,41,49,40,42,52,47,50,42,49,41,48,59,55,56,42,50,46,43,48,52,54,42,46,48,50,43,59,43,57,56,40,58,91,29,77,35,95,11,75,9,75,34,71,5,88,7,73,10,72,5,93,40,87,12,97,36,74,22,90,17,88,20,76,16,89,1,78,1,73,35,83,5,93,26,75,20,95,27,63,13,75,10,92,13,86,15,69,14,90,32,86,15,88,39,97,24,68,17,85,23,69,8,91,16,79,28,74,18,83]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>ID Kupca<\/th>\n      <th>Spol<\/th>\n      <th>Dob<\/th>\n      <th>Godišnji Prihodi(1000$)<\/th>\n      <th>Potrošački kod(1-100)<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[1,3,4,5]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>

## CRISP-DM metodologija

### Razumijevanje problema

Menadžer jednog trgovačkog centra želi identificirati ciljne skupine kupaca te u skladu s time prilagoditi marketinške strategije.

### Razumijevanje podataka

Prvo ćemo provjeriti da li baza sadrži nedostajuće vrijednosti.

``` r
mall_customers_tbl %>% 
    summarise_all(~sum(is.na(.)))
```

    ## # A tibble: 1 × 5
    ##   `ID Kupca`  Spol   Dob `Godišnji Prihodi(1000$)` `Potrošački kod(1-100)`
    ##        <int> <int> <int>                     <int>                   <int>
    ## 1          0     0     0                         0                       0

Nakon što smo utvrdili da ne postoje nedostajuće vrijednosti vizualizirat ćemo osnovne činjenice koje se nalaze u podacima.

``` r
# Distribucija podataka 
mall_customers_tbl %>% 
    ggplot(aes(Dob))+
    geom_histogram(fill = "#5b0b15")+
    theme_tq()+
    labs(title = "Distribucija dobi")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" />

``` r
mall_customers_tbl %>% 
    ggplot(aes(`Godišnji Prihodi(1000$)`))+
    geom_histogram(fill = "#5b0b15")+
    scale_x_continuous(breaks=seq(15, 140, 25))+
    theme_tq()+
    labs(title = "Distribucija godišnjih prihoda(1000$)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-2.png" width="100%" style="display: block; margin: auto;" />

``` r
mall_customers_tbl %>% 
    ggplot(aes(`Potrošački kod(1-100)`))+
    geom_histogram(fill = "#5b0b15")+
    theme_tq()+
    labs(title = "Distribucija potrošačkog koda(1-100)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-3.png" width="100%" style="display: block; margin: auto;" />

Prema histogramima zaključujemo da podaci ne prate normalnu distribuciju te da je najveći broj kupaca u tridesetim godinama sa godišnjim prihodima od 60.000\$ i potrošačkim kodom 40.

``` r
# Box-plot dob, godišnji prihodi i potrošački kod po spolu

mall_customers_tbl %>% 
    ggplot(aes(Dob, Spol, fill= Spol))+
    geom_boxplot(color = c("#414a37", "#5b0b15"),alpha= 0.3) +
    coord_flip() +
    theme_tq()+
    scale_fill_manual(values = c("#414a37", "#5b0b15"))+
    theme(legend.position = "none")+
    labs(title = "Box-plot: dob u odnosu na spol")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="100%" style="display: block; margin: auto;" />

``` r
mall_customers_tbl %>% 
    ggplot(aes(`Godišnji Prihodi(1000$)`, Spol, fill = Spol))+
    geom_boxplot(color = c("#414a37", "#5b0b15"),alpha= 0.3) +
    coord_flip() +
    scale_x_continuous(breaks=seq(15, 140, 25))+
    theme_tq()+
    scale_fill_manual(values = c("#414a37", "#5b0b15"))+
    theme(legend.position = "none")+
    labs(title = "Box-plot: godišnji prihodi(1000$) u odnosu na spol")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-2.png" width="100%" style="display: block; margin: auto;" />

``` r
mall_customers_tbl %>% 
    ggplot(aes(`Potrošački kod(1-100)`, Spol, fill = Spol))+
    geom_boxplot(color = c("#414a37", "#5b0b15"),alpha= 0.3) +
    coord_flip() +
    theme_tq()+
    scale_fill_manual(values = c("#414a37", "#5b0b15"))+
    theme(legend.position = "none")+
    labs(title = "Box-plot: potrošački kod(1-100) u odnosu na spol")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-3.png" width="100%" style="display: block; margin: auto;" />

Box-plot dijagrami pokazuju da kupci muškog spola uglavnom imaju 37 godina, godišnje prihode od 64.000\$ i potrošački kod 50. Kupci ženskog spola uglavnom imaju 35 godina, godišnje prihode od 60.000\$ i potrošački kod 50. Većina muških kupaca ima između 27 i 51 godinu, godišnje prihode između 46.000\$ i 78.000\$ te potrošački kod 25 - 73. Većina ženskih kupaca ima između 29 i 48 godina, godišnje prihode između 40.000 i 76.000 te potrošački kod 40 - 74.

### Priprema podataka

Prije primjene k-means algoritma provest ćemo normalizaciju podataka pomoću funkcije `scale()` kako bi se izbjeglo davanje veće težine varijablama s većim vrijednostima.

``` r
# Normalizacija
mall_customers_tbl_scl<-mall_customers_tbl %>% 
    select(3:5) %>% 
    scale()
```

### Modeliranje

#### Primjena K-means algoritma

Za određivanje optimalnog broja klastera postoje različite metode. Ovdje je korišten jedan vrlo jednostavan pristup. Prvo je primjenjeno klasteriranja *k* -sredinama koristeći različite brojeve *k* klastera. Zatim je na temelju grafa (engl. *Scree Plot*), koji prikazuje zbroj kvadrata unutar klastera (engl. *within cluster sum of squares* (WCSS)) i određeni broj klastera, donešena odluka o broju klastera. WCSS predstavlja kvadrat prosječne udaljenosti svih točaka unutar klastera do središta klastera. Kriterij za donošenje odluke o odgovarajućem broju *k* klastera smatra se položaj koljena na Scree Plot grafu (lokacija savijanja).

Nakon što smo standardizirali podatke, primjenit ćemo algoritam k-sredina te izračunati WCSS za različite vrijednosti *k* klastera (1:15) pomoću funkcije koju ćemo izraditi. Zatim ćemo nacrtati graf (Scree Plot), WCSS vrijednosti sa pripadajućim brojevima *k*, te odrediti optimalan broj *k* klastera (položaj koljena).

``` r
# Funkcija
set.seed(123)
centers <- 3
kmeans_function <- function(centers = 3) {
    mall_customers_tbl_scl %>% 
    kmeans(centers = centers, nstart = 100)
}

# Iteracija fukcije kroz sve sredine
kmeans_tbl <- tibble(centers = 1:15) %>% 
    mutate(k_means = centers %>% map(kmeans_function)) %>% 
    mutate(glance = k_means %>% map(glance))

kmeans_tbl %>% unnest(glance) %>% 
    select(centers, tot.withinss)
```

    ## # A tibble: 15 × 2
    ##    centers tot.withinss
    ##      <int>        <dbl>
    ##  1       1        597. 
    ##  2       2        387. 
    ##  3       3        294. 
    ##  4       4        204. 
    ##  5       5        167. 
    ##  6       6        133. 
    ##  7       7        116. 
    ##  8       8        103. 
    ##  9       9         91.5
    ## 10      10         81.0
    ## 11      11         71.5
    ## 12      12         66.3
    ## 13      13         62.6
    ## 14      14         58.3
    ## 15      15         55.1

``` r
# Scree plot
kmeans_tbl %>% unnest(glance) %>% 
    select(centers, tot.withinss) %>% 
    ggplot(aes(centers, tot.withinss))+
    geom_point(color = "#414a37", size = 0.8)+
    geom_line(color = "#414a37", size = 0.8) +
    ggrepel::geom_label_repel(aes(label = centers), color = "#414a37")+
    geom_vline(xintercept = 6, color = "#5b0b15")+
    theme_tq() +
    labs(title = "Scree Plot")+
    xlab("Broj k klastera")+
    ylab("Total within-clusters sum of squares(WCSS)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="100%" style="display: block; margin: auto;" />

Na Scree Plot-u možemo vidjeti da varijanca unutar klastera opada povećanjem broja klastera. Savijanje ili lakat se uočava na k=6. Ovo savijanje ukazuje na to da dodatni klasteri iza šestog imaju malu vrijednost odnosno da će razlika između klastera nakon te točke biti manje značajna. Prema tome, odabiremo 6 *k* klastera za segmentaciju baze kupaca.

### UMAP vizualizacija

Odabrani optimalni broj k klastera vizulizirati ćemo algoritmom UMAP.

``` r
# Set seed 
custom.config <- umap.defaults
custom.config$random_state <- 123

# Primjena UMAP algoritma
umap_obj <- mall_customers_tbl_scl %>% 
    umap(config=custom.config)

# Ekstrakcija rezultata, priprema i sredivanje podataka za vizualizaciju
umap_results_tbl <- umap_obj$layout %>% 
    as_tibble() %>% 
    set_names("x", "y") %>% 
    bind_cols(
        mall_customers_tbl %>% select(`ID Kupca`))

kmeans_6_obj <- kmeans_tbl %>% 
    pull(k_means) %>% 
    pluck(6)

kmeans_6_clusters <- kmeans_6_obj %>% augment(mall_customers_tbl) %>% 
    select(`ID Kupca`, .cluster, Dob, Spol, `Godišnji Prihodi(1000$)`, `Potrošački kod(1-100)`)

umap_kmeans_6_results_tbl <- umap_results_tbl %>% 
    left_join(kmeans_6_clusters)

umap_tbl <-umap_kmeans_6_results_tbl %>% 
    mutate(lable_text = str_glue("ID Kupca: {`ID Kupca`}
                                Spol: {Spol}
                                Dob: {Dob}
                                Godišnji prihodi: {`Godišnji Prihodi(1000$)`}
                                Potrošački kod: {`Potrošački kod(1-100)`}
                                Cluster: {.cluster}"))

# Vizualizacija UMAP projekcije 
gg1 <- umap_tbl %>% 
     ggplot(aes(x, y, color = .cluster)) + 
     geom_point(aes(text = lable_text)) + 
     theme_tq() +
     scale_color_tq() + 
     labs(title = "Segmentacija kupaca: 2D Projection",
          subtitle = "UMAP 2D projekcija with sa k-means klasterima") +
     theme(legend.position = "none")

# Interaktivna vizualizacija koristeći ggplotly()
ggplotly(gg1, tooltip = "text")
```

<div id="htmlwidget-2" style="width:100%;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-2">{"x":{"data":[{"x":[2.82868024497028,2.75633943604082,2.80831196220846,2.9313305919684,2.84345677431295,2.89170687641893,2.69115691072975,2.79021863221301,2.47032166444375,2.60427159259974,2.7661635152893,2.71307358201494,2.36853562784431,2.47306547192324,2.75161110324981,2.80014709573123,2.69261571497274,2.622894583801,2.47371606389475,-0.933521000254167,2.25042466191851],"y":[-1.90979715735228,-1.53817884853078,-2.2014563985542,-3.06209196378746,-3.05975026919943,-2.96304436576676,-2.18571772162209,-1.68081181594554,-2.69764861024666,-1.47600207686436,-2.80838006182282,-2.84860281341388,-2.23589497097592,-1.92548677941901,-3.16937039568439,-3.02909221747149,-2.76016077908989,-2.34442560324731,-1.8844045092282,-1.45064886144784,-2.70480122255802],"text":["ID Kupca: 3<br />Spol: Žensko<br />Dob: 20<br />Godišnji prihodi: 16<br />Potrošački kod: 6<br />Cluster: 1","ID Kupca: 5<br />Spol: Žensko<br />Dob: 31<br />Godišnji prihodi: 17<br />Potrošački kod: 40<br />Cluster: 1","ID Kupca: 7<br />Spol: Žensko<br />Dob: 35<br />Godišnji prihodi: 18<br />Potrošački kod: 6<br />Cluster: 1","ID Kupca: 9<br />Spol: Muško<br />Dob: 64<br />Godišnji prihodi: 19<br />Potrošački kod: 3<br />Cluster: 1","ID Kupca: 11<br />Spol: Muško<br />Dob: 67<br />Godišnji prihodi: 19<br />Potrošački kod: 14<br />Cluster: 1","ID Kupca: 13<br />Spol: Žensko<br />Dob: 58<br />Godišnji prihodi: 20<br />Potrošački kod: 15<br />Cluster: 1","ID Kupca: 15<br />Spol: Muško<br />Dob: 37<br />Godišnji prihodi: 20<br />Potrošački kod: 13<br />Cluster: 1","ID Kupca: 17<br />Spol: Žensko<br />Dob: 35<br />Godišnji prihodi: 21<br />Potrošački kod: 35<br />Cluster: 1","ID Kupca: 19<br />Spol: Muško<br />Dob: 52<br />Godišnji prihodi: 23<br />Potrošački kod: 29<br />Cluster: 1","ID Kupca: 21<br />Spol: Muško<br />Dob: 35<br />Godišnji prihodi: 24<br />Potrošački kod: 35<br />Cluster: 1","ID Kupca: 23<br />Spol: Žensko<br />Dob: 46<br />Godišnji prihodi: 25<br />Potrošački kod: 5<br />Cluster: 1","ID Kupca: 25<br />Spol: Žensko<br />Dob: 54<br />Godišnji prihodi: 28<br />Potrošački kod: 14<br />Cluster: 1","ID Kupca: 27<br />Spol: Žensko<br />Dob: 45<br />Godišnji prihodi: 28<br />Potrošački kod: 32<br />Cluster: 1","ID Kupca: 29<br />Spol: Žensko<br />Dob: 40<br />Godišnji prihodi: 29<br />Potrošački kod: 31<br />Cluster: 1","ID Kupca: 31<br />Spol: Muško<br />Dob: 60<br />Godišnji prihodi: 30<br />Potrošački kod: 4<br />Cluster: 1","ID Kupca: 33<br />Spol: Muško<br />Dob: 53<br />Godišnji prihodi: 33<br />Potrošački kod: 4<br />Cluster: 1","ID Kupca: 35<br />Spol: Žensko<br />Dob: 49<br />Godišnji prihodi: 33<br />Potrošački kod: 14<br />Cluster: 1","ID Kupca: 37<br />Spol: Žensko<br />Dob: 42<br />Godišnji prihodi: 34<br />Potrošački kod: 17<br />Cluster: 1","ID Kupca: 39<br />Spol: Žensko<br />Dob: 36<br />Godišnji prihodi: 37<br />Potrošački kod: 26<br />Cluster: 1","ID Kupca: 43<br />Spol: Muško<br />Dob: 48<br />Godišnji prihodi: 39<br />Potrošački kod: 36<br />Cluster: 1","ID Kupca: 45<br />Spol: Žensko<br />Dob: 49<br />Godišnji prihodi: 39<br />Potrošački kod: 28<br />Cluster: 1"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(44,62,80,1)","opacity":1,"size":5.66929133858268,"symbol":"circle","line":{"width":1.88976377952756,"color":"rgba(44,62,80,1)"}},"hoveron":"points","name":"1","legendgroup":"1","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[-0.808367849209236,-1.39880057814874,-1.06791664262092,-0.146264108385058,-0.900140389864916,-1.4161146394095,-0.149724953184884,-1.28557706270766,-0.488138535977437,-0.964125867096838,-0.946889693375438,-1.12646178013357,-0.596754237673678,-0.592784347954891,-0.101439720848785,-1.31933055312712,-0.952371108932585,-0.645068502128102,-0.749801914264565,-0.582023474399459,-1.52629988115695,-1.61467541821359,-1.56194685721482,-0.800256800935052,-1.24530319461061,-0.813282206405199,-1.56082390772955,-1.01427847944759,-0.736302796142565,-0.823560851679926,-0.958026563926313,-1.01298389509324,-0.876135295065798],"y":[-3.07136795407841,-4.71818175885235,-4.81500095664894,-4.65069756615166,-4.96135545023497,-4.81545935345665,-4.67181426705995,-3.89798870459685,-5.07992200789826,-5.05536214854446,-5.17186039677761,-4.76413368184627,-5.03919780880059,-5.07991375140043,-4.62813631798825,-4.50239224602413,-5.33064550948455,-5.51272518640264,-5.35738515304492,-5.21472243099812,-4.91917431108446,-4.77270885123474,-4.83441892084357,-5.78139423604507,-5.57913955655756,-5.72448147844004,-5.0961396499124,-5.79802235046183,-5.94708662664909,-6.04797369659031,-5.98126750000177,-5.92276512361515,-6.00578454200627],"text":["ID Kupca: 127<br />Spol: Muško<br />Dob: 43<br />Godišnji prihodi: 71<br />Potrošački kod: 35<br />Cluster: 2","ID Kupca: 129<br />Spol: Muško<br />Dob: 59<br />Godišnji prihodi: 71<br />Potrošački kod: 11<br />Cluster: 2","ID Kupca: 131<br />Spol: Muško<br />Dob: 47<br />Godišnji prihodi: 71<br />Potrošački kod: 9<br />Cluster: 2","ID Kupca: 135<br />Spol: Muško<br />Dob: 20<br />Godišnji prihodi: 73<br />Potrošački kod: 5<br />Cluster: 2","ID Kupca: 137<br />Spol: Žensko<br />Dob: 44<br />Godišnji prihodi: 73<br />Potrošački kod: 7<br />Cluster: 2","ID Kupca: 141<br />Spol: Žensko<br />Dob: 57<br />Godišnji prihodi: 75<br />Potrošački kod: 5<br />Cluster: 2","ID Kupca: 145<br />Spol: Muško<br />Dob: 25<br />Godišnji prihodi: 77<br />Potrošački kod: 12<br />Cluster: 2","ID Kupca: 147<br />Spol: Muško<br />Dob: 48<br />Godišnji prihodi: 77<br />Potrošački kod: 36<br />Cluster: 2","ID Kupca: 149<br />Spol: Žensko<br />Dob: 34<br />Godišnji prihodi: 78<br />Potrošački kod: 22<br />Cluster: 2","ID Kupca: 151<br />Spol: Muško<br />Dob: 43<br />Godišnji prihodi: 78<br />Potrošački kod: 17<br />Cluster: 2","ID Kupca: 153<br />Spol: Žensko<br />Dob: 44<br />Godišnji prihodi: 78<br />Potrošački kod: 20<br />Cluster: 2","ID Kupca: 155<br />Spol: Žensko<br />Dob: 47<br />Godišnji prihodi: 78<br />Potrošački kod: 16<br />Cluster: 2","ID Kupca: 157<br />Spol: Muško<br />Dob: 37<br />Godišnji prihodi: 78<br />Potrošački kod: 1<br />Cluster: 2","ID Kupca: 159<br />Spol: Muško<br />Dob: 34<br />Godišnji prihodi: 78<br />Potrošački kod: 1<br />Cluster: 2","ID Kupca: 163<br />Spol: Muško<br />Dob: 19<br />Godišnji prihodi: 81<br />Potrošački kod: 5<br />Cluster: 2","ID Kupca: 165<br />Spol: Muško<br />Dob: 50<br />Godišnji prihodi: 85<br />Potrošački kod: 26<br />Cluster: 2","ID Kupca: 167<br />Spol: Muško<br />Dob: 42<br />Godišnji prihodi: 86<br />Potrošački kod: 20<br />Cluster: 2","ID Kupca: 169<br />Spol: Žensko<br />Dob: 36<br />Godišnji prihodi: 87<br />Potrošački kod: 27<br />Cluster: 2","ID Kupca: 171<br />Spol: Muško<br />Dob: 40<br />Godišnji prihodi: 87<br />Potrošački kod: 13<br />Cluster: 2","ID Kupca: 173<br />Spol: Muško<br />Dob: 36<br />Godišnji prihodi: 87<br />Potrošački kod: 10<br />Cluster: 2","ID Kupca: 175<br />Spol: Žensko<br />Dob: 52<br />Godišnji prihodi: 88<br />Potrošački kod: 13<br />Cluster: 2","ID Kupca: 177<br />Spol: Muško<br />Dob: 58<br />Godišnji prihodi: 88<br />Potrošački kod: 15<br />Cluster: 2","ID Kupca: 179<br />Spol: Muško<br />Dob: 59<br />Godišnji prihodi: 93<br />Potrošački kod: 14<br />Cluster: 2","ID Kupca: 181<br />Spol: Žensko<br />Dob: 37<br />Godišnji prihodi: 97<br />Potrošački kod: 32<br />Cluster: 2","ID Kupca: 183<br />Spol: Muško<br />Dob: 46<br />Godišnji prihodi: 98<br />Potrošački kod: 15<br />Cluster: 2","ID Kupca: 185<br />Spol: Žensko<br />Dob: 41<br />Godišnji prihodi: 99<br />Potrošački kod: 39<br />Cluster: 2","ID Kupca: 187<br />Spol: Žensko<br />Dob: 54<br />Godišnji prihodi: 101<br />Potrošački kod: 24<br />Cluster: 2","ID Kupca: 189<br />Spol: Žensko<br />Dob: 41<br />Godišnji prihodi: 103<br />Potrošački kod: 17<br />Cluster: 2","ID Kupca: 191<br />Spol: Žensko<br />Dob: 34<br />Godišnji prihodi: 103<br />Potrošački kod: 23<br />Cluster: 2","ID Kupca: 193<br />Spol: Muško<br />Dob: 33<br />Godišnji prihodi: 113<br />Potrošački kod: 8<br />Cluster: 2","ID Kupca: 195<br />Spol: Žensko<br />Dob: 47<br />Godišnji prihodi: 120<br />Potrošački kod: 16<br />Cluster: 2","ID Kupca: 197<br />Spol: Žensko<br />Dob: 45<br />Godišnji prihodi: 126<br />Potrošački kod: 28<br />Cluster: 2","ID Kupca: 199<br />Spol: Muško<br />Dob: 32<br />Godišnji prihodi: 137<br />Potrošački kod: 18<br />Cluster: 2"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(227,26,28,1)","opacity":1,"size":5.66929133858268,"symbol":"circle","line":{"width":1.88976377952756,"color":"rgba(227,26,28,1)"}},"hoveron":"points","name":"2","legendgroup":"2","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[2.91025343856058,6.16930764003755,6.14278731350684,6.11226568968742,6.12807216775703,5.4738850772755,5.85355426752978,5.93337498425942,6.05078510466815,5.98020332503298,5.95921453246818,5.61009896645527,5.24133020388682,5.3185070853705,3.31236246282773,5.80831739894642,5.42209639646111,5.97408293996376,5.67396675468719,5.09812022312464,5.43927657583398,5.82529489767751,3.21323004922159,4.92906779295966],"y":[-1.27802390863461,-0.82344634576385,-0.856449828329696,-0.856916635870488,-0.705960703689479,-0.984619194878719,-1.06482043549122,-0.99930323075041,-0.746011713842507,-1.04652168129387,-0.981970510939147,-0.87957590080234,-0.896397041383878,-0.830316239375394,-0.512616816213575,-0.485279570217416,-0.480001572723996,-0.590243856658408,-0.40390102089069,-0.760622044044957,-0.320129009461951,-0.438559040084211,-0.468006548348054,-0.290029395003876],"text":["ID Kupca: 1<br />Spol: Muško<br />Dob: 19<br />Godišnji prihodi: 15<br />Potrošački kod: 39<br />Cluster: 3","ID Kupca: 2<br />Spol: Muško<br />Dob: 21<br />Godišnji prihodi: 15<br />Potrošački kod: 81<br />Cluster: 3","ID Kupca: 4<br />Spol: Žensko<br />Dob: 23<br />Godišnji prihodi: 16<br />Potrošački kod: 77<br />Cluster: 3","ID Kupca: 6<br />Spol: Žensko<br />Dob: 22<br />Godišnji prihodi: 17<br />Potrošački kod: 76<br />Cluster: 3","ID Kupca: 8<br />Spol: Žensko<br />Dob: 23<br />Godišnji prihodi: 18<br />Potrošački kod: 94<br />Cluster: 3","ID Kupca: 10<br />Spol: Žensko<br />Dob: 30<br />Godišnji prihodi: 19<br />Potrošački kod: 72<br />Cluster: 3","ID Kupca: 12<br />Spol: Žensko<br />Dob: 35<br />Godišnji prihodi: 19<br />Potrošački kod: 99<br />Cluster: 3","ID Kupca: 14<br />Spol: Žensko<br />Dob: 24<br />Godišnji prihodi: 20<br />Potrošački kod: 77<br />Cluster: 3","ID Kupca: 16<br />Spol: Muško<br />Dob: 22<br />Godišnji prihodi: 20<br />Potrošački kod: 79<br />Cluster: 3","ID Kupca: 18<br />Spol: Muško<br />Dob: 20<br />Godišnji prihodi: 21<br />Potrošački kod: 66<br />Cluster: 3","ID Kupca: 20<br />Spol: Žensko<br />Dob: 35<br />Godišnji prihodi: 23<br />Potrošački kod: 98<br />Cluster: 3","ID Kupca: 22<br />Spol: Muško<br />Dob: 25<br />Godišnji prihodi: 24<br />Potrošački kod: 73<br />Cluster: 3","ID Kupca: 24<br />Spol: Muško<br />Dob: 31<br />Godišnji prihodi: 25<br />Potrošački kod: 73<br />Cluster: 3","ID Kupca: 26<br />Spol: Muško<br />Dob: 29<br />Godišnji prihodi: 28<br />Potrošački kod: 82<br />Cluster: 3","ID Kupca: 28<br />Spol: Muško<br />Dob: 35<br />Godišnji prihodi: 28<br />Potrošački kod: 61<br />Cluster: 3","ID Kupca: 30<br />Spol: Žensko<br />Dob: 23<br />Godišnji prihodi: 29<br />Potrošački kod: 87<br />Cluster: 3","ID Kupca: 32<br />Spol: Žensko<br />Dob: 21<br />Godišnji prihodi: 30<br />Potrošački kod: 73<br />Cluster: 3","ID Kupca: 34<br />Spol: Muško<br />Dob: 18<br />Godišnji prihodi: 33<br />Potrošački kod: 92<br />Cluster: 3","ID Kupca: 36<br />Spol: Žensko<br />Dob: 21<br />Godišnji prihodi: 33<br />Potrošački kod: 81<br />Cluster: 3","ID Kupca: 38<br />Spol: Žensko<br />Dob: 30<br />Godišnji prihodi: 34<br />Potrošački kod: 73<br />Cluster: 3","ID Kupca: 40<br />Spol: Žensko<br />Dob: 20<br />Godišnji prihodi: 37<br />Potrošački kod: 75<br />Cluster: 3","ID Kupca: 42<br />Spol: Muško<br />Dob: 24<br />Godišnji prihodi: 38<br />Potrošački kod: 92<br />Cluster: 3","ID Kupca: 44<br />Spol: Žensko<br />Dob: 31<br />Godišnji prihodi: 39<br />Potrošački kod: 61<br />Cluster: 3","ID Kupca: 46<br />Spol: Žensko<br />Dob: 24<br />Godišnji prihodi: 39<br />Potrošački kod: 65<br />Cluster: 3"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(24,188,156,1)","opacity":1,"size":5.66929133858268,"symbol":"circle","line":{"width":1.88976377952756,"color":"rgba(24,188,156,1)"}},"hoveron":"points","name":"3","legendgroup":"3","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[-2.36859000767908,-1.78609454776784,-2.45179777796307,-1.63868609307555,-1.6484032426972,-1.7248067864177,-2.59860631187153,-1.68505125776299,-1.52605986383753,-2.74234820297625,-2.70475357142857,-2.81302668626107,-1.65046558341645,-2.96072948323447,-2.4228733831667,-1.61779935676781,-2.77014133114099,-2.1083459680333,-1.86810724382796,-2.53147077363107,-2.93660921954205,-1.95607007153125,-3.27878895828584,-2.32317400604801,-2.45021900958934,-3.28614394175817,-3.04980732644014,-2.33011570929369,-3.46138285493386,-3.185393973625,-3.38867891378168,-3.42628649728335,-2.75856969719505,-3.3637671436456,-2.95439768363366,-3.55460087845323,-3.43351416340133,-3.40780438119947,-3.48422654103003],"y":[7.41307668365678,8.16100111946738,7.43089030561839,7.82997647510207,7.79812641345725,8.23155757951003,7.67356892810779,8.00345446773589,7.97031833753423,7.49703512279018,7.62807539257056,7.62183258919721,8.35347569443124,7.43418448717391,7.37897381052377,7.84439384800968,7.82012786339265,8.26881074868036,8.28538513916234,8.14201712150941,7.74385395631431,8.16485534507032,7.9212215279005,8.602369874495,8.49068452990808,7.71508100229879,8.19181553522359,8.56755241233898,8.02073227133617,8.24865324908602,8.28673479165631,8.03535776398314,8.61038179218453,8.44629564113517,8.66221150173755,8.57697308691538,8.66593968717877,8.78268292277015,8.62719537491543],"text":["ID Kupca: 124<br />Spol: Muško<br />Dob: 39<br />Godišnji prihodi: 69<br />Potrošački kod: 91<br />Cluster: 4","ID Kupca: 126<br />Spol: Žensko<br />Dob: 31<br />Godišnji prihodi: 70<br />Potrošački kod: 77<br />Cluster: 4","ID Kupca: 128<br />Spol: Muško<br />Dob: 40<br />Godišnji prihodi: 71<br />Potrošački kod: 95<br />Cluster: 4","ID Kupca: 130<br />Spol: Muško<br />Dob: 38<br />Godišnji prihodi: 71<br />Potrošački kod: 75<br />Cluster: 4","ID Kupca: 132<br />Spol: Muško<br />Dob: 39<br />Godišnji prihodi: 71<br />Potrošački kod: 75<br />Cluster: 4","ID Kupca: 134<br />Spol: Žensko<br />Dob: 31<br />Godišnji prihodi: 72<br />Potrošački kod: 71<br />Cluster: 4","ID Kupca: 136<br />Spol: Žensko<br />Dob: 29<br />Godišnji prihodi: 73<br />Potrošački kod: 88<br />Cluster: 4","ID Kupca: 138<br />Spol: Muško<br />Dob: 32<br />Godišnji prihodi: 73<br />Potrošački kod: 73<br />Cluster: 4","ID Kupca: 140<br />Spol: Žensko<br />Dob: 35<br />Godišnji prihodi: 74<br />Potrošački kod: 72<br />Cluster: 4","ID Kupca: 142<br />Spol: Muško<br />Dob: 32<br />Godišnji prihodi: 75<br />Potrošački kod: 93<br />Cluster: 4","ID Kupca: 144<br />Spol: Žensko<br />Dob: 32<br />Godišnji prihodi: 76<br />Potrošački kod: 87<br />Cluster: 4","ID Kupca: 146<br />Spol: Muško<br />Dob: 28<br />Godišnji prihodi: 77<br />Potrošački kod: 97<br />Cluster: 4","ID Kupca: 148<br />Spol: Žensko<br />Dob: 32<br />Godišnji prihodi: 77<br />Potrošački kod: 74<br />Cluster: 4","ID Kupca: 150<br />Spol: Muško<br />Dob: 34<br />Godišnji prihodi: 78<br />Potrošački kod: 90<br />Cluster: 4","ID Kupca: 152<br />Spol: Muško<br />Dob: 39<br />Godišnji prihodi: 78<br />Potrošački kod: 88<br />Cluster: 4","ID Kupca: 154<br />Spol: Žensko<br />Dob: 38<br />Godišnji prihodi: 78<br />Potrošački kod: 76<br />Cluster: 4","ID Kupca: 156<br />Spol: Žensko<br />Dob: 27<br />Godišnji prihodi: 78<br />Potrošački kod: 89<br />Cluster: 4","ID Kupca: 158<br />Spol: Žensko<br />Dob: 30<br />Godišnji prihodi: 78<br />Potrošački kod: 78<br />Cluster: 4","ID Kupca: 160<br />Spol: Žensko<br />Dob: 30<br />Godišnji prihodi: 78<br />Potrošački kod: 73<br />Cluster: 4","ID Kupca: 162<br />Spol: Žensko<br />Dob: 29<br />Godišnji prihodi: 79<br />Potrošački kod: 83<br />Cluster: 4","ID Kupca: 164<br />Spol: Žensko<br />Dob: 31<br />Godišnji prihodi: 81<br />Potrošački kod: 93<br />Cluster: 4","ID Kupca: 166<br />Spol: Žensko<br />Dob: 36<br />Godišnji prihodi: 85<br />Potrošački kod: 75<br />Cluster: 4","ID Kupca: 168<br />Spol: Žensko<br />Dob: 33<br />Godišnji prihodi: 86<br />Potrošački kod: 95<br />Cluster: 4","ID Kupca: 170<br />Spol: Muško<br />Dob: 32<br />Godišnji prihodi: 87<br />Potrošački kod: 63<br />Cluster: 4","ID Kupca: 172<br />Spol: Muško<br />Dob: 28<br />Godišnji prihodi: 87<br />Potrošački kod: 75<br />Cluster: 4","ID Kupca: 174<br />Spol: Muško<br />Dob: 36<br />Godišnji prihodi: 87<br />Potrošački kod: 92<br />Cluster: 4","ID Kupca: 176<br />Spol: Žensko<br />Dob: 30<br />Godišnji prihodi: 88<br />Potrošački kod: 86<br />Cluster: 4","ID Kupca: 178<br />Spol: Muško<br />Dob: 27<br />Godišnji prihodi: 88<br />Potrošački kod: 69<br />Cluster: 4","ID Kupca: 180<br />Spol: Muško<br />Dob: 35<br />Godišnji prihodi: 93<br />Potrošački kod: 90<br />Cluster: 4","ID Kupca: 182<br />Spol: Žensko<br />Dob: 32<br />Godišnji prihodi: 97<br />Potrošački kod: 86<br />Cluster: 4","ID Kupca: 184<br />Spol: Žensko<br />Dob: 29<br />Godišnji prihodi: 98<br />Potrošački kod: 88<br />Cluster: 4","ID Kupca: 186<br />Spol: Muško<br />Dob: 30<br />Godišnji prihodi: 99<br />Potrošački kod: 97<br />Cluster: 4","ID Kupca: 188<br />Spol: Muško<br />Dob: 28<br />Godišnji prihodi: 101<br />Potrošački kod: 68<br />Cluster: 4","ID Kupca: 190<br />Spol: Žensko<br />Dob: 36<br />Godišnji prihodi: 103<br />Potrošački kod: 85<br />Cluster: 4","ID Kupca: 192<br />Spol: Žensko<br />Dob: 32<br />Godišnji prihodi: 103<br />Potrošački kod: 69<br />Cluster: 4","ID Kupca: 194<br />Spol: Žensko<br />Dob: 38<br />Godišnji prihodi: 113<br />Potrošački kod: 91<br />Cluster: 4","ID Kupca: 196<br />Spol: Žensko<br />Dob: 35<br />Godišnji prihodi: 120<br />Potrošački kod: 79<br />Cluster: 4","ID Kupca: 198<br />Spol: Muško<br />Dob: 32<br />Godišnji prihodi: 126<br />Potrošački kod: 74<br />Cluster: 4","ID Kupca: 200<br />Spol: Muško<br />Dob: 30<br />Godišnji prihodi: 137<br />Potrošački kod: 83<br />Cluster: 4"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(204,190,147,1)","opacity":1,"size":5.66929133858268,"symbol":"circle","line":{"width":1.88976377952756,"color":"rgba(204,190,147,1)"}},"hoveron":"points","name":"4","legendgroup":"4","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[-2.99276453361282,-1.44895513230782,-1.38536755241229,-2.35938117259697,-1.20377318734929,-1.01700138463866,-1.49126562141126,-2.94961875254271,-1.62386998646757,-3.05408905409203,-3.08963951356422,-2.02772615713677,-2.69157383847492,-0.55135192027376,-3.01633940294363,-3.16667071043482,-0.949163275603545,-2.51185662330026,-2.55702577257975,-2.64160554069293,-0.735794487311263,-0.946540176810984,-2.27014795164112,-3.02764667560923,-0.93292475715876,-1.05756783726236,-2.17931775172344,-1.03407150220317,-3.31698819823553,-0.946538876378772,-0.895427968112198,-0.856551958717559,-1.07842166871094,-3.35464949690342,-1.35888532037849,-3.43925893976046,-1.49312154913069,-3.30473799040022,-3.52358931165776,-3.38938006739545,-3.47941830120169,-1.37224771910061,-1.15873205068617,-1.43646618670916,-1.47155549265577],"y":[-1.29889466046458,-1.21232293341852,-1.1638298643058,-1.02725398484008,-1.3152465720476,-1.49362387831115,-1.13756853923875,-1.35941118808584,-1.13152967564798,-1.36998627364996,-1.20932377095556,-1.19346124372947,-1.10132540732851,-1.67568238905765,-1.22275543373385,-1.2072154185541,-1.79546554013824,-0.942896681549639,-0.980202386666211,-1.08448999485032,-1.91292794632069,-2.06878960316495,-1.11380233106922,-1.21608132919516,-2.02265961054082,-1.99455927104165,-1.42254533356573,-2.41151147165098,-0.841202695454439,-2.34136040471776,-2.37343745679453,-2.61601311175534,-2.56882612018014,-0.956742378609163,-2.39087258353479,-1.12612781191805,-2.71590971797913,-1.01222091165324,-1.07772853600277,-0.955555717179905,-1.16351515952286,-2.28793271960422,-2.84652890157414,-2.31560885845508,-4.12718816587208],"text":["ID Kupca: 41<br />Spol: Žensko<br />Dob: 65<br />Godišnji prihodi: 38<br />Potrošački kod: 35<br />Cluster: 5","ID Kupca: 47<br />Spol: Žensko<br />Dob: 50<br />Godišnji prihodi: 40<br />Potrošački kod: 55<br />Cluster: 5","ID Kupca: 51<br />Spol: Žensko<br />Dob: 49<br />Godišnji prihodi: 42<br />Potrošački kod: 52<br />Cluster: 5","ID Kupca: 54<br />Spol: Muško<br />Dob: 59<br />Godišnji prihodi: 43<br />Potrošački kod: 60<br />Cluster: 5","ID Kupca: 55<br />Spol: Žensko<br />Dob: 50<br />Godišnji prihodi: 43<br />Potrošački kod: 45<br />Cluster: 5","ID Kupca: 56<br />Spol: Muško<br />Dob: 47<br />Godišnji prihodi: 43<br />Potrošački kod: 41<br />Cluster: 5","ID Kupca: 57<br />Spol: Žensko<br />Dob: 51<br />Godišnji prihodi: 44<br />Potrošački kod: 50<br />Cluster: 5","ID Kupca: 58<br />Spol: Muško<br />Dob: 69<br />Godišnji prihodi: 44<br />Potrošački kod: 46<br />Cluster: 5","ID Kupca: 60<br />Spol: Muško<br />Dob: 53<br />Godišnji prihodi: 46<br />Potrošački kod: 46<br />Cluster: 5","ID Kupca: 61<br />Spol: Muško<br />Dob: 70<br />Godišnji prihodi: 46<br />Potrošački kod: 56<br />Cluster: 5","ID Kupca: 63<br />Spol: Žensko<br />Dob: 67<br />Godišnji prihodi: 47<br />Potrošački kod: 52<br />Cluster: 5","ID Kupca: 64<br />Spol: Žensko<br />Dob: 54<br />Godišnji prihodi: 47<br />Potrošački kod: 59<br />Cluster: 5","ID Kupca: 65<br />Spol: Muško<br />Dob: 63<br />Godišnji prihodi: 48<br />Potrošački kod: 51<br />Cluster: 5","ID Kupca: 67<br />Spol: Žensko<br />Dob: 43<br />Godišnji prihodi: 48<br />Potrošački kod: 50<br />Cluster: 5","ID Kupca: 68<br />Spol: Žensko<br />Dob: 68<br />Godišnji prihodi: 48<br />Potrošački kod: 48<br />Cluster: 5","ID Kupca: 71<br />Spol: Muško<br />Dob: 70<br />Godišnji prihodi: 49<br />Potrošački kod: 55<br />Cluster: 5","ID Kupca: 72<br />Spol: Žensko<br />Dob: 47<br />Godišnji prihodi: 49<br />Potrošački kod: 42<br />Cluster: 5","ID Kupca: 73<br />Spol: Žensko<br />Dob: 60<br />Godišnji prihodi: 50<br />Potrošački kod: 49<br />Cluster: 5","ID Kupca: 74<br />Spol: Žensko<br />Dob: 60<br />Godišnji prihodi: 50<br />Potrošački kod: 56<br />Cluster: 5","ID Kupca: 75<br />Spol: Muško<br />Dob: 59<br />Godišnji prihodi: 54<br />Potrošački kod: 47<br />Cluster: 5","ID Kupca: 77<br />Spol: Žensko<br />Dob: 45<br />Godišnji prihodi: 54<br />Potrošački kod: 53<br />Cluster: 5","ID Kupca: 80<br />Spol: Žensko<br />Dob: 49<br />Godišnji prihodi: 54<br />Potrošački kod: 42<br />Cluster: 5","ID Kupca: 81<br />Spol: Muško<br />Dob: 57<br />Godišnji prihodi: 54<br />Potrošački kod: 51<br />Cluster: 5","ID Kupca: 83<br />Spol: Muško<br />Dob: 67<br />Godišnji prihodi: 54<br />Potrošački kod: 41<br />Cluster: 5","ID Kupca: 84<br />Spol: Žensko<br />Dob: 46<br />Godišnji prihodi: 54<br />Potrošački kod: 44<br />Cluster: 5","ID Kupca: 86<br />Spol: Muško<br />Dob: 48<br />Godišnji prihodi: 54<br />Potrošački kod: 46<br />Cluster: 5","ID Kupca: 87<br />Spol: Žensko<br />Dob: 55<br />Godišnji prihodi: 57<br />Potrošački kod: 58<br />Cluster: 5","ID Kupca: 90<br />Spol: Žensko<br />Dob: 50<br />Godišnji prihodi: 58<br />Potrošački kod: 46<br />Cluster: 5","ID Kupca: 91<br />Spol: Žensko<br />Dob: 68<br />Godišnji prihodi: 59<br />Potrošački kod: 55<br />Cluster: 5","ID Kupca: 93<br />Spol: Muško<br />Dob: 48<br />Godišnji prihodi: 60<br />Potrošački kod: 49<br />Cluster: 5","ID Kupca: 97<br />Spol: Žensko<br />Dob: 47<br />Godišnji prihodi: 60<br />Potrošački kod: 47<br />Cluster: 5","ID Kupca: 99<br />Spol: Muško<br />Dob: 48<br />Godišnji prihodi: 61<br />Potrošački kod: 42<br />Cluster: 5","ID Kupca: 102<br />Spol: Žensko<br />Dob: 49<br />Godišnji prihodi: 62<br />Potrošački kod: 48<br />Cluster: 5","ID Kupca: 103<br />Spol: Muško<br />Dob: 67<br />Godišnji prihodi: 62<br />Potrošački kod: 59<br />Cluster: 5","ID Kupca: 105<br />Spol: Muško<br />Dob: 49<br />Godišnji prihodi: 62<br />Potrošački kod: 56<br />Cluster: 5","ID Kupca: 107<br />Spol: Žensko<br />Dob: 66<br />Godišnji prihodi: 63<br />Potrošački kod: 50<br />Cluster: 5","ID Kupca: 108<br />Spol: Muško<br />Dob: 54<br />Godišnji prihodi: 63<br />Potrošački kod: 46<br />Cluster: 5","ID Kupca: 109<br />Spol: Muško<br />Dob: 68<br />Godišnji prihodi: 63<br />Potrošački kod: 43<br />Cluster: 5","ID Kupca: 110<br />Spol: Muško<br />Dob: 66<br />Godišnji prihodi: 63<br />Potrošački kod: 48<br />Cluster: 5","ID Kupca: 111<br />Spol: Muško<br />Dob: 65<br />Godišnji prihodi: 63<br />Potrošački kod: 52<br />Cluster: 5","ID Kupca: 117<br />Spol: Žensko<br />Dob: 63<br />Godišnji prihodi: 65<br />Potrošački kod: 43<br />Cluster: 5","ID Kupca: 118<br />Spol: Žensko<br />Dob: 49<br />Godišnji prihodi: 65<br />Potrošački kod: 59<br />Cluster: 5","ID Kupca: 119<br />Spol: Žensko<br />Dob: 51<br />Godišnji prihodi: 67<br />Potrošački kod: 43<br />Cluster: 5","ID Kupca: 120<br />Spol: Žensko<br />Dob: 50<br />Godišnji prihodi: 67<br />Potrošački kod: 57<br />Cluster: 5","ID Kupca: 161<br />Spol: Žensko<br />Dob: 56<br />Godišnji prihodi: 79<br />Potrošački kod: 35<br />Cluster: 5"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(166,206,227,1)","opacity":1,"size":5.66929133858268,"symbol":"circle","line":{"width":1.88976377952756,"color":"rgba(166,206,227,1)"}},"hoveron":"points","name":"5","legendgroup":"5","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[2.7133585648864,2.72575871197927,2.6512763671355,3.07228358970695,2.72674411523687,2.51211297213713,1.65396886873102,1.68530036970206,1.71752515417882,2.50177995139219,1.42093097692322,-0.126305397584799,1.46085182846609,-0.0483222625997028,1.3518198331144,1.1727197668433,0.348385019207465,0.676861471166513,-0.176555662097687,0.202281611750885,1.05985630173063,1.12111235725701,0.862335360041789,0.570449366538346,1.09726030248867,0.642186858794749,1.14371985444449,-0.0798760425207719,0.835175512184688,0.696452126963659,0.847394203010016,1.10603166878556,-0.180846080468538,-0.00529202874014534,0.387320283134855,0.415689115041265,-0.0837774764347093,0.366757991964863],"y":[-0.495549473734861,-0.663902570944214,-0.748474720481309,-0.409063175205293,-0.475629475967042,-0.30028632275597,0.847868040973449,0.817028666603115,0.752978204460776,-0.502777844824582,0.435196176035004,-1.76335166393979,0.797931673933885,-1.51965063100131,0.697146710521784,0.831983196345752,-1.15425489826364,0.784526649278445,-1.8449083859244,-1.38760802218868,0.749537750220937,0.353843111738017,0.885018117565286,0.53925494730405,0.495290107234094,0.786330579410697,1.07111717428142,-1.72275454459257,0.941969660901174,0.846197043072019,1.01121804273683,0.424726910475039,-1.94979570453251,-1.36716472388978,0.322456459303847,0.358796462666167,-4.59050511817508,0.184630361863921],"text":["ID Kupca: 48<br />Spol: Žensko<br />Dob: 27<br />Godišnji prihodi: 40<br />Potrošački kod: 47<br />Cluster: 6","ID Kupca: 49<br />Spol: Žensko<br />Dob: 29<br />Godišnji prihodi: 40<br />Potrošački kod: 42<br />Cluster: 6","ID Kupca: 50<br />Spol: Žensko<br />Dob: 31<br />Godišnji prihodi: 40<br />Potrošački kod: 42<br />Cluster: 6","ID Kupca: 52<br />Spol: Muško<br />Dob: 33<br />Godišnji prihodi: 42<br />Potrošački kod: 60<br />Cluster: 6","ID Kupca: 53<br />Spol: Žensko<br />Dob: 31<br />Godišnji prihodi: 43<br />Potrošački kod: 54<br />Cluster: 6","ID Kupca: 59<br />Spol: Žensko<br />Dob: 27<br />Godišnji prihodi: 46<br />Potrošački kod: 51<br />Cluster: 6","ID Kupca: 62<br />Spol: Muško<br />Dob: 19<br />Godišnji prihodi: 46<br />Potrošački kod: 55<br />Cluster: 6","ID Kupca: 66<br />Spol: Muško<br />Dob: 18<br />Godišnji prihodi: 48<br />Potrošački kod: 59<br />Cluster: 6","ID Kupca: 69<br />Spol: Muško<br />Dob: 19<br />Godišnji prihodi: 48<br />Potrošački kod: 59<br />Cluster: 6","ID Kupca: 70<br />Spol: Žensko<br />Dob: 32<br />Godišnji prihodi: 48<br />Potrošački kod: 47<br />Cluster: 6","ID Kupca: 76<br />Spol: Muško<br />Dob: 26<br />Godišnji prihodi: 54<br />Potrošački kod: 54<br />Cluster: 6","ID Kupca: 78<br />Spol: Muško<br />Dob: 40<br />Godišnji prihodi: 54<br />Potrošački kod: 48<br />Cluster: 6","ID Kupca: 79<br />Spol: Žensko<br />Dob: 23<br />Godišnji prihodi: 54<br />Potrošački kod: 52<br />Cluster: 6","ID Kupca: 82<br />Spol: Muško<br />Dob: 38<br />Godišnji prihodi: 54<br />Potrošački kod: 55<br />Cluster: 6","ID Kupca: 85<br />Spol: Žensko<br />Dob: 21<br />Godišnji prihodi: 54<br />Potrošački kod: 57<br />Cluster: 6","ID Kupca: 88<br />Spol: Žensko<br />Dob: 22<br />Godišnji prihodi: 57<br />Potrošački kod: 55<br />Cluster: 6","ID Kupca: 89<br />Spol: Žensko<br />Dob: 34<br />Godišnji prihodi: 58<br />Potrošački kod: 60<br />Cluster: 6","ID Kupca: 92<br />Spol: Muško<br />Dob: 18<br />Godišnji prihodi: 59<br />Potrošački kod: 41<br />Cluster: 6","ID Kupca: 94<br />Spol: Žensko<br />Dob: 40<br />Godišnji prihodi: 60<br />Potrošački kod: 40<br />Cluster: 6","ID Kupca: 95<br />Spol: Žensko<br />Dob: 32<br />Godišnji prihodi: 60<br />Potrošački kod: 42<br />Cluster: 6","ID Kupca: 96<br />Spol: Muško<br />Dob: 24<br />Godišnji prihodi: 60<br />Potrošački kod: 52<br />Cluster: 6","ID Kupca: 98<br />Spol: Žensko<br />Dob: 27<br />Godišnji prihodi: 60<br />Potrošački kod: 50<br />Cluster: 6","ID Kupca: 100<br />Spol: Muško<br />Dob: 20<br />Godišnji prihodi: 61<br />Potrošački kod: 49<br />Cluster: 6","ID Kupca: 101<br />Spol: Žensko<br />Dob: 23<br />Godišnji prihodi: 62<br />Potrošački kod: 41<br />Cluster: 6","ID Kupca: 104<br />Spol: Muško<br />Dob: 26<br />Godišnji prihodi: 62<br />Potrošački kod: 55<br />Cluster: 6","ID Kupca: 106<br />Spol: Žensko<br />Dob: 21<br />Godišnji prihodi: 62<br />Potrošački kod: 42<br />Cluster: 6","ID Kupca: 112<br />Spol: Žensko<br />Dob: 19<br />Godišnji prihodi: 63<br />Potrošački kod: 54<br />Cluster: 6","ID Kupca: 113<br />Spol: Žensko<br />Dob: 38<br />Godišnji prihodi: 64<br />Potrošački kod: 42<br />Cluster: 6","ID Kupca: 114<br />Spol: Muško<br />Dob: 19<br />Godišnji prihodi: 64<br />Potrošački kod: 46<br />Cluster: 6","ID Kupca: 115<br />Spol: Žensko<br />Dob: 18<br />Godišnji prihodi: 65<br />Potrošački kod: 48<br />Cluster: 6","ID Kupca: 116<br />Spol: Žensko<br />Dob: 19<br />Godišnji prihodi: 65<br />Potrošački kod: 50<br />Cluster: 6","ID Kupca: 121<br />Spol: Muško<br />Dob: 27<br />Godišnji prihodi: 67<br />Potrošački kod: 56<br />Cluster: 6","ID Kupca: 122<br />Spol: Žensko<br />Dob: 38<br />Godišnji prihodi: 67<br />Potrošački kod: 40<br />Cluster: 6","ID Kupca: 123<br />Spol: Žensko<br />Dob: 40<br />Godišnji prihodi: 69<br />Potrošački kod: 58<br />Cluster: 6","ID Kupca: 125<br />Spol: Žensko<br />Dob: 23<br />Godišnji prihodi: 70<br />Potrošački kod: 29<br />Cluster: 6","ID Kupca: 133<br />Spol: Žensko<br />Dob: 25<br />Godišnji prihodi: 72<br />Potrošački kod: 34<br />Cluster: 6","ID Kupca: 139<br />Spol: Muško<br />Dob: 19<br />Godišnji prihodi: 74<br />Potrošački kod: 10<br />Cluster: 6","ID Kupca: 143<br />Spol: Žensko<br />Dob: 28<br />Godišnji prihodi: 76<br />Potrošački kod: 40<br />Cluster: 6"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(31,120,180,1)","opacity":1,"size":5.66929133858268,"symbol":"circle","line":{"width":1.88976377952756,"color":"rgba(31,120,180,1)"}},"hoveron":"points","name":"6","legendgroup":"6","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":40.1826484018265,"l":37.2602739726027},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187},"title":{"text":"Segmentacija kupaca: 2D Projection","font":{"color":"rgba(44,62,80,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-4.04079630437777,6.65550306596208],"tickmode":"array","ticktext":["-4","-2","0","2","4","6"],"tickvals":[-4,-2,0,2,4,6],"categoryorder":"array","categoryarray":["-4","-2","0","2","4","6"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0.22139200221392,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0.22139200221392,"zeroline":false,"anchor":"y","title":{"text":"x","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-6.78950652755833,9.52421575373817],"tickmode":"array","ticktext":["-5","0","5"],"tickvals":[-5,0,5],"categoryorder":"array","categoryarray":["-5","0","5"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0.22139200221392,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0.22139200221392,"zeroline":false,"anchor":"x","title":{"text":"y","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0.33208800332088,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895}},"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"source":"A","attrs":{"1221361c27675":{"x":{},"y":{},"colour":{},"text":{},"type":"scatter"}},"cur_data":"1221361c27675","visdat":{"1221361c27675":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>

### Interpretacija klastera

Nakon što smo vizualizirali dobivene grupe kupaca, interpretirat ćemo klastere:

- **Klaster 1**: grupa kupaca sa niskim godišnjim prihodima, niskim potrošačkim kodom, uglavnom srednjih godina
- **Klater 2**: kupci sa visokim prihodima, niskim potrošačkim kodom, uglavnom srednjih godina
- **Klaster 3**: kupci sa niskim godišnjim prihodima, visokim potrošačkim kodom, uglavnom mladi kupci
- **Klaster 4**: kupci sa visokim godišnjim prihodima, visokog potrošačkog koda, uglavnom srednjih godina
- **Klaster 5**: kupci sa srednjim godišnjim prihodima, srednjeg potrošačkog koda, uglavnom stariji kupaci
- **Klaster 6**: kupci sa srednjim prihodima, srednjeg potrošačkog koda, uglavnom srednjih godina

Interaktivna tablica kupaca sa pripadajućim klasterima.

``` r
datatable(kmeans_6_clusters, caption = htmltools::tags$caption(
    style = 'caption-side: top; text-align: center;',
    'Tablica 2: ', htmltools::em('Klasteri'))
    )
```

<div id="htmlwidget-3" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-3">{"x":{"filter":"none","vertical":false,"caption":"<caption style=\"caption-side: top; text-align: center;\">\n  Tablica 2: \n  <em>Klasteri<\/em>\n<\/caption>","data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31","32","33","34","35","36","37","38","39","40","41","42","43","44","45","46","47","48","49","50","51","52","53","54","55","56","57","58","59","60","61","62","63","64","65","66","67","68","69","70","71","72","73","74","75","76","77","78","79","80","81","82","83","84","85","86","87","88","89","90","91","92","93","94","95","96","97","98","99","100","101","102","103","104","105","106","107","108","109","110","111","112","113","114","115","116","117","118","119","120","121","122","123","124","125","126","127","128","129","130","131","132","133","134","135","136","137","138","139","140","141","142","143","144","145","146","147","148","149","150","151","152","153","154","155","156","157","158","159","160","161","162","163","164","165","166","167","168","169","170","171","172","173","174","175","176","177","178","179","180","181","182","183","184","185","186","187","188","189","190","191","192","193","194","195","196","197","198","199","200"],[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200],["3","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","1","3","5","3","1","3","1","3","5","6","6","6","5","6","6","5","5","5","5","5","6","5","5","6","5","5","5","6","5","5","6","6","5","5","5","5","5","6","5","6","6","5","5","6","5","5","6","5","5","6","6","5","5","6","5","6","6","6","5","6","5","6","6","5","5","6","5","6","5","5","5","5","5","6","6","6","6","6","5","5","5","5","6","6","6","4","6","4","2","4","2","4","2","4","6","4","2","4","2","4","6","4","2","4","6","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","5","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4","2","4"],[19,21,20,23,31,22,35,23,64,30,67,35,58,24,37,22,35,20,52,35,35,25,46,31,54,29,45,35,40,23,60,21,53,18,49,21,42,30,36,20,65,24,48,31,49,24,50,27,29,31,49,33,31,59,50,47,51,69,27,53,70,19,67,54,63,18,43,68,19,32,70,47,60,60,59,26,45,40,23,49,57,38,67,46,21,48,55,22,34,50,68,18,48,40,32,24,47,27,48,20,23,49,67,26,49,21,66,54,68,66,65,19,38,19,18,19,63,49,51,50,27,38,40,39,23,31,43,40,59,38,47,39,25,31,20,29,44,32,19,35,57,32,28,32,25,28,48,32,34,34,43,39,44,38,47,27,37,30,34,30,56,29,19,31,50,36,42,33,36,32,40,28,36,36,52,30,58,27,59,35,37,32,46,29,41,30,54,28,41,36,34,32,33,38,47,35,45,32,32,30],["Muško","Muško","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Žensko","Muško","Muško","Žensko","Muško","Muško","Žensko","Muško","Muško","Žensko","Muško","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Muško","Žensko","Muško","Muško","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Muško","Muško","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Muško","Žensko","Muško","Žensko","Muško","Muško","Muško","Žensko","Žensko","Muško","Muško","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Žensko","Muško","Muško","Žensko","Muško","Žensko","Žensko","Muško","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Žensko","Žensko","Žensko","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Muško","Žensko","Žensko","Muško","Muško","Muško","Žensko","Žensko","Muško","Muško","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Žensko","Žensko","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Muško","Muško","Muško","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Muško","Muško","Žensko","Žensko","Muško","Muško","Muško","Žensko","Žensko","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Muško","Muško","Muško","Muško","Muško","Žensko","Žensko","Muško","Muško","Muško","Muško","Žensko","Žensko","Muško","Žensko","Žensko","Muško","Žensko","Muško","Žensko","Žensko","Žensko","Žensko","Muško","Žensko","Žensko","Žensko","Žensko","Muško","Muško","Muško"],[15,15,16,16,17,17,18,18,19,19,19,19,20,20,20,20,21,21,23,23,24,24,25,25,28,28,28,28,29,29,30,30,33,33,33,33,34,34,37,37,38,38,39,39,39,39,40,40,40,40,42,42,43,43,43,43,44,44,46,46,46,46,47,47,48,48,48,48,48,48,49,49,50,50,54,54,54,54,54,54,54,54,54,54,54,54,57,57,58,58,59,59,60,60,60,60,60,60,61,61,62,62,62,62,62,62,63,63,63,63,63,63,64,64,65,65,65,65,67,67,67,67,69,69,70,70,71,71,71,71,71,71,72,72,73,73,73,73,74,74,75,75,76,76,77,77,77,77,78,78,78,78,78,78,78,78,78,78,78,78,79,79,81,81,85,85,86,86,87,87,87,87,87,87,88,88,88,88,93,93,97,97,98,98,99,99,101,101,103,103,103,103,113,113,120,120,126,126,137,137],[39,81,6,77,40,76,6,94,3,72,14,99,15,77,13,79,35,66,29,98,35,73,5,73,14,82,32,61,31,87,4,73,4,92,14,81,17,73,26,75,35,92,36,61,28,65,55,47,42,42,52,60,54,60,45,41,50,46,51,46,56,55,52,59,51,59,50,48,59,47,55,42,49,56,47,54,53,48,52,42,51,55,41,44,57,46,58,55,60,46,55,41,49,40,42,52,47,50,42,49,41,48,59,55,56,42,50,46,43,48,52,54,42,46,48,50,43,59,43,57,56,40,58,91,29,77,35,95,11,75,9,75,34,71,5,88,7,73,10,72,5,93,40,87,12,97,36,74,22,90,17,88,20,76,16,89,1,78,1,73,35,83,5,93,26,75,20,95,27,63,13,75,10,92,13,86,15,69,14,90,32,86,15,88,39,97,24,68,17,85,23,69,8,91,16,79,28,74,18,83]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>ID Kupca<\/th>\n      <th>.cluster<\/th>\n      <th>Dob<\/th>\n      <th>Spol<\/th>\n      <th>Godišnji Prihodi(1000$)<\/th>\n      <th>Potrošački kod(1-100)<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[1,3,5,6]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>

## EVALUACIJA

Uvid u klastere kupaca i njihovo bolje razumijevanje omogućuje organizaciji da donosi bolje i informirane odluke. Navedeno se odnosi na spoznaje o postojanju grupe kupaca koji imaju visoke godišnje prihode ali nisku ocjenu potrošnje. Strateški i ciljani marketinški pristup mogao bi podići njihov interes što bi moglo rezultirati većom potrošnjom. Također, istovremeno je potrebno održavati zadovoljstva “lojalnih kupaca” jer su ono najveći generator prihoda.

## IMPLEMENTACIJA

Provedena segmentacija kupaca implementirana je unutar `Shiny` web aplikacije koja omogućuje interaktivnu vizualizaciju i interpretaciju rezultata (Slika 1).
Pokreni aplikaciju

[^1]: https://www.researchgate.net/publication/356756320_Customer_Segmentation_Using_Machine_Learning

[^2]: https://www.youtube.com/watch?v=4b5d3muPQmA

[^3]: https://www.youtube.com/watch?v=eN0wFzBA4Sc

[^4]: https://www.ibm.com/docs/it/spss-modeler/saas?topic=dm-crisp-help-overview

[^5]: https://www.kaggle.com/datasets/vjchoudhary7/customer-segmentation-tutorial-in-python
