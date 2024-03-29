---
title: "Analiza potrošačke košarice"
author: "Marijana Andabaka"
date: "21.03.2023."
---




![](img/img1.jpg)

# UVOD  {.tabset .tabset-pills}

U današnjem svijetu, mali kao i veliki poslovni subjekti uvijek traže načine za povećanje svoje dobiti. Kako bi to učinili mnogi koriste tehniku koja se zove analiza potrošačke košarice. 

Analiza potrošačke kosarice (engl. *Market Basket Analysis*, *Affinity Analysis*) je proces razumijevanja obrasca ponašanja kupaca pri kupnji i povezanosti između različitih artikala. Cilj je pronaći artikle koji se pojavljuju zajedno u transakcijama što omogućuje primjenu ciljnih strategija poput raspodjele artikala unutar trgovina, davanje popusta na artikle koji se često prodaju zajedno te kreiranja preporuka proizvoda s ciljem privlačenja kupaca i navođenja na neplaniranu potrošnju. Postoji nekoliko metoda analize potrošačke košarice:

- __Kolaborativno filtriranje__ (engl. *Collaborative Filtering*)
- __Pravila pridruživanja__ (engl. *Association Rules*)
- __Popularni artikli__ (engl. *Item Popularity*)
- __Kontekstno bazirani sustavi za preporuke__ (engl. *Content-Based-Filtering*)
- __Hibridni modeli__

__Pritisnite donje kartice__ kako biste saznali nešto više o metodama koje će se  koristiti u ovom projektu. 

## Kolaborativno filtriranje

__Kolaborativno filtriranje__ temelji se na sličnostima korisnika ili proizvoda koje se računaju putem unaprijed definirane funkcije sličnosti (engl. *similarity function*). Funkcija sličnosti numerički određuje udaljenost (npr. euklidska udaljenost, Pearson korelacija, kosinus udaljenost) između dva korisnika ili proizvoda u svrhu definiranja onih koji su najbliži.

Sustavi orijentirani prema korisniku (engl. *user-based collaborative filtering*) pronalaze slične korisnike za određenog promatranog korisnika te se temelje na pretpostavci da slični korisnici preferiraju slične proizvode. 

Sustavi orijentirani prema proizvodima (engl. *item-based collaborative filtering*) pronalaze slične proizvode za određeni promatrani proizvod te se temelje na pretpostavci da slične proizvode preferiraju slični korisnici.

## Pravila pridruživanja

Metoda __pravila pridruživanja__ jedna je od najčešće korištenih metoda u analiziranju potrošačke košarice ili transakcijskih podataka. Cilj je otkriti specifične uzorke ili pravila unutar skupa promatranih podataka na temelju koncepta strogih pravila. Pravila između dva kupljena proizvoda nastaju pomoću tri veličine: 

- značaj (engl. *support*) je vjerojatnost da se dvije stavke istovremeno pojavljuju `$${P(stavka1,stavka2)}$$`
- pouzdanost (engl. *confidence*) predstavlja značaj podijeljen sa vjerojatnošću kupnje druge stavke `$$\frac{znacaj}{P(stavka2)}$$`
- poboljšanje (engl. *lift*) je pouzdanost podijeljenja sa vjerojatnošću kupnje prve stavke `$$\frac{pouzdanost}{P(stavka1)}$$`

Kad se ove veličine kombiniraju, stavke sa najvećom vjerojatnošću kupnje su one koje imaju najveće poboljšanje. Metoda se odlikuje brzinom i dobrim funkcioniranjem kod analize predmeta koji se najčešće kupuju. 

## Popularni artikli

Strategija primjene metode __popularni artikli__ je vrlo jednostavna. Artikli se razvrstavaju na temelju učestalosti kupnje (tj. popularnosti) u svrhu razumijevanja kupovnih navika. Sustav preporuka se temelji samo na najčešće kupljenim artiklima koji se trenutno ne kupuju. Loše strana ovog pristupa je njegova jednostavnost i nedostatak nekih temeljnih sličnosti unutar segmenata ili skupina kupaca. 

# ANALIZA

U ovom projektu napravit ćemo nekoliko modela pomoću kolaborativnog filtriranja, pravila pridruživanja i popularnosti artikala unutar `recommenderlab` paketa.
 `recommenderlab` paket omogućuje procjenu i usporedbu različitih algoritama te brzu procjenu modela sa najboljim performansama.

## Učitavanje paketa

Za provedbu projekta učitani su potrebni paketi: `recommenderlab`, `tidyverse`, `tidyquant`, `fs`, `knitr`, `glue`.


```r
library(recommenderlab) 
library(tidyverse) 
library(tidyquant)
library(knitr)
library(glue)
library(DT) 
```
