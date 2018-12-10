---
title: "Raport z analizy danych"
author: "Arutur Wesolowski"
date: \12 10,2018\
output: 
  html_document: 
    keep_md: yes
    number_sections: yes
    toc: yes
    toc_float: yes
---


#Wst�p

Poni�szy raport ma na celu podj�cie analizy danych pochodz�cych z bazy Protein Data Bank (PDB). Dane zawieraj� informacje na temat ligand�w. Zb�r danych zawiera mi�dzy innymi nazw� danej cz�steczki chemicznej, ilo�� atom�w oraz elektron�w oraz inne kolumny oparte o tr�jwymiarowy fragment g�sto�ci elektronowej struktury. Przy analizie pomini�te zosta�y kolumny utworzone przy pomocy warto�ci s�ownikowych. Ze wzgl�du na problem ze �rodowiskiem, zbi�r pocz�tkowy ograniczono do 400 000 wierszy.

#U�yte biblioteki

```r
library(EDAWR)
library(dplyr)
library(DT)
library(ggplot2)
library(plotly)
library(reshape2)
library(cowplot)
library(data.table)
library(qwraps2)
library(fastDummies)
library(reshape2)
library(caret)
library(kableExtra)
library(pROC)
```

#Powtarzalno�� wynik�w.


```r
set.seed(123)
```




#Wczytywanie danych z pliku.


```r
initial<-fread("all_summary.csv", nrows = 100)
colClass <- sapply(initial, class)
pdb_table<-fread("all_summary.csv", nrows = 400000, colClasses = colClass)
```

#Usuwanie wierszy z wybrana warto�ci� res_name.



#Przetwarzanie brakuj�cych danych.


```r
for(i in 1:ncol(pdb_clear_res_name_table)){
  pdb_clear_res_name_table[is.na(pdb_clear_res_name_table[,i]), i] <- mean(pdb_clear_res_name_table[,i], na.rm = TRUE)
}
```





Ze zbioru usuniete zostaly wiersze posiadajace wartosci zmiennej res_name rozne od: UNK, UNX, UNL, DUM, N, BLOB, ALA, ARG, ASN, ASP, CYS, GLN, GLU, GLY, HIS, ILE, LEU, LYS, MET, MSE, PHE, PRO, SEC, SER, THR, TRP, TYR, VAL, DA, DG, DT, DC, DU, A, G, T, C, U, HOH, H20, WAT.
Zbior ograniczono do kolumn opisanych na stronie projektu, nie uwzgledniono kolumn <font color='red'>nie wykorzystywanych do klasyfikacji</font> poza kolumnami res_name, local_res_atom_non_h_count, local_res_atom_non_h_count, dict_atom_non_h_count, dict_atom_non_h_electron_sum. Wartosci 'Na' zostaly zastapione srednia wartoscia dla danej kolumny.

# Podsumowanie zbioru.

Zbior przed wyczyszczeniem posiadał wymiar: 400000, 412 [wierszy, kolumn]. Po oczyszczeniu zbioru wymiary wynosza: 393259, 336 [wierszy, kolumn].

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;">   res_name </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Length:393259 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Class :character </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> Mode  :character </td>
  </tr>
</tbody>
</table>

<div style="border: 1px solid #ddd; padding: 5px; overflow-y: scroll; height:400px; overflow-x: scroll; width:100%; "><table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> mean </th>
   <th style="text-align:right;"> sd </th>
   <th style="text-align:right;"> median </th>
   <th style="text-align:right;"> min </th>
   <th style="text-align:right;"> max </th>
   <th style="text-align:left;"> class </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> local_res_atom_non_h_count </td>
   <td style="text-align:right;"> 1.353000e+01 </td>
   <td style="text-align:right;"> 1.514000e+01 </td>
   <td style="text-align:right;"> 6.000000e+00 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 1.060000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> local_res_atom_non_h_electron_sum </td>
   <td style="text-align:right;"> 1.003800e+02 </td>
   <td style="text-align:right;"> 1.019700e+02 </td>
   <td style="text-align:right;"> 4.800000e+01 </td>
   <td style="text-align:right;"> 3.00 </td>
   <td style="text-align:right;"> 1.848000e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dict_atom_non_h_count </td>
   <td style="text-align:right;"> 1.387000e+01 </td>
   <td style="text-align:right;"> 1.561000e+01 </td>
   <td style="text-align:right;"> 6.000000e+00 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 1.260000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dict_atom_non_h_electron_sum </td>
   <td style="text-align:right;"> 1.029000e+02 </td>
   <td style="text-align:right;"> 1.045600e+02 </td>
   <td style="text-align:right;"> 5.200000e+01 </td>
   <td style="text-align:right;"> 3.00 </td>
   <td style="text-align:right;"> 1.223000e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> local_volume </td>
   <td style="text-align:right;"> 8.564600e+02 </td>
   <td style="text-align:right;"> 1.467780e+03 </td>
   <td style="text-align:right;"> 3.438200e+02 </td>
   <td style="text-align:right;"> 49.25 </td>
   <td style="text-align:right;"> 9.095251e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> local_electrons </td>
   <td style="text-align:right;"> 1.769000e+01 </td>
   <td style="text-align:right;"> 2.526000e+01 </td>
   <td style="text-align:right;"> 7.770000e+00 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 4.424400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> local_mean </td>
   <td style="text-align:right;"> 2.000000e-02 </td>
   <td style="text-align:right;"> 2.000000e-02 </td>
   <td style="text-align:right;"> 2.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.700000e-01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> local_std </td>
   <td style="text-align:right;"> 1.200000e-01 </td>
   <td style="text-align:right;"> 1.000000e-01 </td>
   <td style="text-align:right;"> 1.000000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.960000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> local_min </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> local_max </td>
   <td style="text-align:right;"> 1.350000e+00 </td>
   <td style="text-align:right;"> 1.590000e+00 </td>
   <td style="text-align:right;"> 8.900000e-01 </td>
   <td style="text-align:right;"> 0.03 </td>
   <td style="text-align:right;"> 4.463000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> local_skewness </td>
   <td style="text-align:right;"> 2.200000e-01 </td>
   <td style="text-align:right;"> 1.900000e-01 </td>
   <td style="text-align:right;"> 1.700000e-01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 4.040000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_segments_count </td>
   <td style="text-align:right;"> 3.388500e+02 </td>
   <td style="text-align:right;"> 1.110370e+03 </td>
   <td style="text-align:right;"> 2.500000e+01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.145770e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_segments_count </td>
   <td style="text-align:right;"> 3.388500e+02 </td>
   <td style="text-align:right;"> 1.110370e+03 </td>
   <td style="text-align:right;"> 2.500000e+01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.145770e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_volume </td>
   <td style="text-align:right;"> 3.287000e+01 </td>
   <td style="text-align:right;"> 5.051000e+01 </td>
   <td style="text-align:right;"> 1.422000e+01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.427940e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_electrons </td>
   <td style="text-align:right;"> 1.749000e+01 </td>
   <td style="text-align:right;"> 2.511000e+01 </td>
   <td style="text-align:right;"> 7.710000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 4.411400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_mean </td>
   <td style="text-align:right;"> 6.000000e-01 </td>
   <td style="text-align:right;"> 4.000000e-01 </td>
   <td style="text-align:right;"> 5.100000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 8.600000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_std </td>
   <td style="text-align:right;"> 2.100000e-01 </td>
   <td style="text-align:right;"> 3.000000e-01 </td>
   <td style="text-align:right;"> 1.200000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 8.010000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_max </td>
   <td style="text-align:right;"> 1.350000e+00 </td>
   <td style="text-align:right;"> 1.590000e+00 </td>
   <td style="text-align:right;"> 8.900000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 4.463000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_max_over_std </td>
   <td style="text-align:right;"> 9.740000e+00 </td>
   <td style="text-align:right;"> 7.600000e+00 </td>
   <td style="text-align:right;"> 7.220000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.732500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_skewness </td>
   <td style="text-align:right;"> 2.100000e-01 </td>
   <td style="text-align:right;"> 3.400000e-01 </td>
   <td style="text-align:right;"> 1.100000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.051000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_parts </td>
   <td style="text-align:right;"> 1.070000e+00 </td>
   <td style="text-align:right;"> 3.800000e-01 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.800000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_O3 </td>
   <td style="text-align:right;"> 1.695558e+06 </td>
   <td style="text-align:right;"> 9.850960e+06 </td>
   <td style="text-align:right;"> 9.601052e+04 </td>
   <td style="text-align:right;"> 121.39 </td>
   <td style="text-align:right;"> 2.293302e+09 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_O4 </td>
   <td style="text-align:right;"> 1.116615e+13 </td>
   <td style="text-align:right;"> 9.992375e+14 </td>
   <td style="text-align:right;"> 2.274156e+09 </td>
   <td style="text-align:right;"> 3807.05 </td>
   <td style="text-align:right;"> 4.027551e+17 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_O5 </td>
   <td style="text-align:right;"> 1.916881e+20 </td>
   <td style="text-align:right;"> 5.147942e+22 </td>
   <td style="text-align:right;"> 1.481914e+13 </td>
   <td style="text-align:right;"> 33777.03 </td>
   <td style="text-align:right;"> 2.908412e+25 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_FL </td>
   <td style="text-align:right;"> 6.023041e+16 </td>
   <td style="text-align:right;"> 1.439510e+19 </td>
   <td style="text-align:right;"> 3.770154e+10 </td>
   <td style="text-align:right;"> 75.70 </td>
   <td style="text-align:right;"> 5.852600e+21 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_O3_norm </td>
   <td style="text-align:right;"> 4.900000e-01 </td>
   <td style="text-align:right;"> 3.300000e-01 </td>
   <td style="text-align:right;"> 3.800000e-01 </td>
   <td style="text-align:right;"> 0.23 </td>
   <td style="text-align:right;"> 3.965000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_O4_norm </td>
   <td style="text-align:right;"> 6.000000e-02 </td>
   <td style="text-align:right;"> 8.000000e-02 </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 6.010000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_O5_norm </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 4.100000e-01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_FL_norm </td>
   <td style="text-align:right;"> 6.000000e-02 </td>
   <td style="text-align:right;"> 5.900000e-01 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.893500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I1 </td>
   <td style="text-align:right;"> 3.252631e+09 </td>
   <td style="text-align:right;"> 3.464861e+11 </td>
   <td style="text-align:right;"> 7.712680e+06 </td>
   <td style="text-align:right;"> 528.82 </td>
   <td style="text-align:right;"> 1.633222e+14 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I2 </td>
   <td style="text-align:right;"> 2.919856e+20 </td>
   <td style="text-align:right;"> 7.877431e+22 </td>
   <td style="text-align:right;"> 8.701642e+12 </td>
   <td style="text-align:right;"> 56939.71 </td>
   <td style="text-align:right;"> 3.482558e+25 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I3 </td>
   <td style="text-align:right;"> 1.178734e+23 </td>
   <td style="text-align:right;"> 4.974270e+25 </td>
   <td style="text-align:right;"> 2.430589e+13 </td>
   <td style="text-align:right;"> 121692.16 </td>
   <td style="text-align:right;"> 2.665214e+28 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I4 </td>
   <td style="text-align:right;"> 3.480335e+16 </td>
   <td style="text-align:right;"> 7.655498e+18 </td>
   <td style="text-align:right;"> 2.063800e+10 </td>
   <td style="text-align:right;"> 42.47 </td>
   <td style="text-align:right;"> 2.867703e+21 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I5 </td>
   <td style="text-align:right;"> 1.785198e+16 </td>
   <td style="text-align:right;"> 3.389322e+18 </td>
   <td style="text-align:right;"> 6.199766e+09 </td>
   <td style="text-align:right;"> 6.19 </td>
   <td style="text-align:right;"> 1.540703e+21 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I6 </td>
   <td style="text-align:right;"> 2.049683e+18 </td>
   <td style="text-align:right;"> 7.101063e+20 </td>
   <td style="text-align:right;"> 3.367402e+11 </td>
   <td style="text-align:right;"> 28626.18 </td>
   <td style="text-align:right;"> 3.743294e+23 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I1_norm </td>
   <td style="text-align:right;"> 5.700000e-01 </td>
   <td style="text-align:right;"> 6.750000e+00 </td>
   <td style="text-align:right;"> 2.300000e-01 </td>
   <td style="text-align:right;"> 0.06 </td>
   <td style="text-align:right;"> 2.760570e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I2_norm </td>
   <td style="text-align:right;"> 9.000000e-02 </td>
   <td style="text-align:right;"> 1.100000e+00 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.039100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I3_norm </td>
   <td style="text-align:right;"> 4.553000e+01 </td>
   <td style="text-align:right;"> 1.461936e+04 </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 7.617375e+06 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I4_norm </td>
   <td style="text-align:right;"> 4.000000e-02 </td>
   <td style="text-align:right;"> 5.800000e-01 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.890700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I5_norm </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 5.800000e-01 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.888900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_I6_norm </td>
   <td style="text-align:right;"> 1.120000e+00 </td>
   <td style="text-align:right;"> 2.293800e+02 </td>
   <td style="text-align:right;"> 4.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.094147e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_M000 </td>
   <td style="text-align:right;"> 4.109220e+03 </td>
   <td style="text-align:right;"> 6.313310e+03 </td>
   <td style="text-align:right;"> 1.778000e+03 </td>
   <td style="text-align:right;"> 38.00 </td>
   <td style="text-align:right;"> 3.034930e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_CI </td>
   <td style="text-align:right;"> 4.000000e-02 </td>
   <td style="text-align:right;"> 4.120000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> -129.45 </td>
   <td style="text-align:right;"> 7.004000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_E3_E1 </td>
   <td style="text-align:right;"> 2.400000e-01 </td>
   <td style="text-align:right;"> 2.000000e-01 </td>
   <td style="text-align:right;"> 1.700000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 9.900000e-01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_E2_E1 </td>
   <td style="text-align:right;"> 4.200000e-01 </td>
   <td style="text-align:right;"> 2.400000e-01 </td>
   <td style="text-align:right;"> 3.800000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_E3_E2 </td>
   <td style="text-align:right;"> 5.500000e-01 </td>
   <td style="text-align:right;"> 2.300000e-01 </td>
   <td style="text-align:right;"> 5.700000e-01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_sqrt_E1 </td>
   <td style="text-align:right;"> 8.030000e+00 </td>
   <td style="text-align:right;"> 5.950000e+00 </td>
   <td style="text-align:right;"> 5.870000e+00 </td>
   <td style="text-align:right;"> 1.24 </td>
   <td style="text-align:right;"> 2.027600e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_sqrt_E2 </td>
   <td style="text-align:right;"> 4.420000e+00 </td>
   <td style="text-align:right;"> 2.730000e+00 </td>
   <td style="text-align:right;"> 3.510000e+00 </td>
   <td style="text-align:right;"> 0.74 </td>
   <td style="text-align:right;"> 3.452000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_sqrt_E3 </td>
   <td style="text-align:right;"> 2.940000e+00 </td>
   <td style="text-align:right;"> 1.420000e+00 </td>
   <td style="text-align:right;"> 2.580000e+00 </td>
   <td style="text-align:right;"> 0.60 </td>
   <td style="text-align:right;"> 1.993000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_O3 </td>
   <td style="text-align:right;"> 7.961914e+05 </td>
   <td style="text-align:right;"> 2.894272e+06 </td>
   <td style="text-align:right;"> 4.686123e+04 </td>
   <td style="text-align:right;"> 9.71 </td>
   <td style="text-align:right;"> 4.315815e+08 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_O4 </td>
   <td style="text-align:right;"> 1.360434e+12 </td>
   <td style="text-align:right;"> 3.255212e+13 </td>
   <td style="text-align:right;"> 5.451315e+08 </td>
   <td style="text-align:right;"> 24.38 </td>
   <td style="text-align:right;"> 1.207528e+16 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_O5 </td>
   <td style="text-align:right;"> 1.589070e+18 </td>
   <td style="text-align:right;"> 1.517293e+20 </td>
   <td style="text-align:right;"> 1.722258e+12 </td>
   <td style="text-align:right;"> 17.32 </td>
   <td style="text-align:right;"> 5.920612e+22 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_FL </td>
   <td style="text-align:right;"> 3.306074e+15 </td>
   <td style="text-align:right;"> 6.223221e+17 </td>
   <td style="text-align:right;"> 8.229923e+09 </td>
   <td style="text-align:right;"> -3.01 </td>
   <td style="text-align:right;"> 3.058302e+20 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_O3_norm </td>
   <td style="text-align:right;"> 7.500000e-01 </td>
   <td style="text-align:right;"> 1.070000e+00 </td>
   <td style="text-align:right;"> 6.100000e-01 </td>
   <td style="text-align:right;"> 0.04 </td>
   <td style="text-align:right;"> 4.123300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_O4_norm </td>
   <td style="text-align:right;"> 1.500000e-01 </td>
   <td style="text-align:right;"> 2.200000e-01 </td>
   <td style="text-align:right;"> 9.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.293000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_O5_norm </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 2.000000e-02 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 4.430000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_FL_norm </td>
   <td style="text-align:right;"> 3.800000e-01 </td>
   <td style="text-align:right;"> 4.728000e+01 </td>
   <td style="text-align:right;"> 2.000000e-02 </td>
   <td style="text-align:right;"> -0.03 </td>
   <td style="text-align:right;"> 2.927105e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I1 </td>
   <td style="text-align:right;"> 1.058597e+09 </td>
   <td style="text-align:right;"> 3.301963e+10 </td>
   <td style="text-align:right;"> 3.414976e+06 </td>
   <td style="text-align:right;"> 42.22 </td>
   <td style="text-align:right;"> 1.282083e+13 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I2 </td>
   <td style="text-align:right;"> 1.219865e+19 </td>
   <td style="text-align:right;"> 2.907814e+21 </td>
   <td style="text-align:right;"> 1.725746e+12 </td>
   <td style="text-align:right;"> 363.10 </td>
   <td style="text-align:right;"> 1.380658e+24 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I3 </td>
   <td style="text-align:right;"> 1.003384e+21 </td>
   <td style="text-align:right;"> 3.220425e+23 </td>
   <td style="text-align:right;"> 4.769437e+12 </td>
   <td style="text-align:right;"> 775.23 </td>
   <td style="text-align:right;"> 1.642551e+26 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I4 </td>
   <td style="text-align:right;"> 1.996255e+15 </td>
   <td style="text-align:right;"> 3.413131e+17 </td>
   <td style="text-align:right;"> 4.943933e+09 </td>
   <td style="text-align:right;"> -1.01 </td>
   <td style="text-align:right;"> 1.713566e+20 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I5 </td>
   <td style="text-align:right;"> 1.123042e+15 </td>
   <td style="text-align:right;"> 1.563157e+17 </td>
   <td style="text-align:right;"> 1.945365e+09 </td>
   <td style="text-align:right;"> 0.18 </td>
   <td style="text-align:right;"> 8.170753e+19 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I6 </td>
   <td style="text-align:right;"> 3.604626e+16 </td>
   <td style="text-align:right;"> 7.467917e+18 </td>
   <td style="text-align:right;"> 7.076168e+10 </td>
   <td style="text-align:right;"> 182.74 </td>
   <td style="text-align:right;"> 2.905409e+21 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I1_norm </td>
   <td style="text-align:right;"> 2.940000e+00 </td>
   <td style="text-align:right;"> 5.121100e+02 </td>
   <td style="text-align:right;"> 5.800000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.985910e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I2_norm </td>
   <td style="text-align:right;"> 7.900000e-01 </td>
   <td style="text-align:right;"> 2.053000e+01 </td>
   <td style="text-align:right;"> 5.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 7.616410e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I3_norm </td>
   <td style="text-align:right;"> 2.621333e+05 </td>
   <td style="text-align:right;"> 1.425401e+08 </td>
   <td style="text-align:right;"> 1.500000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 8.911718e+10 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I4_norm </td>
   <td style="text-align:right;"> 3.100000e-01 </td>
   <td style="text-align:right;"> 4.721000e+01 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> -0.01 </td>
   <td style="text-align:right;"> 2.923185e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I5_norm </td>
   <td style="text-align:right;"> 2.700000e-01 </td>
   <td style="text-align:right;"> 4.716000e+01 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.920571e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_I6_norm </td>
   <td style="text-align:right;"> 4.339200e+02 </td>
   <td style="text-align:right;"> 1.991169e+05 </td>
   <td style="text-align:right;"> 1.700000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.230801e+08 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_M000 </td>
   <td style="text-align:right;"> 2.186050e+03 </td>
   <td style="text-align:right;"> 3.139180e+03 </td>
   <td style="text-align:right;"> 9.634300e+02 </td>
   <td style="text-align:right;"> 3.05 </td>
   <td style="text-align:right;"> 5.514218e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_CI </td>
   <td style="text-align:right;"> 4.000000e-02 </td>
   <td style="text-align:right;"> 4.700000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> -155.70 </td>
   <td style="text-align:right;"> 8.996000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_E3_E1 </td>
   <td style="text-align:right;"> 2.500000e-01 </td>
   <td style="text-align:right;"> 2.000000e-01 </td>
   <td style="text-align:right;"> 1.700000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_E2_E1 </td>
   <td style="text-align:right;"> 4.200000e-01 </td>
   <td style="text-align:right;"> 2.500000e-01 </td>
   <td style="text-align:right;"> 3.800000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_E3_E2 </td>
   <td style="text-align:right;"> 5.600000e-01 </td>
   <td style="text-align:right;"> 2.300000e-01 </td>
   <td style="text-align:right;"> 5.800000e-01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_sqrt_E1 </td>
   <td style="text-align:right;"> 7.720000e+00 </td>
   <td style="text-align:right;"> 5.840000e+00 </td>
   <td style="text-align:right;"> 5.550000e+00 </td>
   <td style="text-align:right;"> 1.24 </td>
   <td style="text-align:right;"> 2.024800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_sqrt_E2 </td>
   <td style="text-align:right;"> 4.200000e+00 </td>
   <td style="text-align:right;"> 2.630000e+00 </td>
   <td style="text-align:right;"> 3.270000e+00 </td>
   <td style="text-align:right;"> 0.74 </td>
   <td style="text-align:right;"> 3.280000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_sqrt_E3 </td>
   <td style="text-align:right;"> 2.780000e+00 </td>
   <td style="text-align:right;"> 1.340000e+00 </td>
   <td style="text-align:right;"> 2.430000e+00 </td>
   <td style="text-align:right;"> 0.60 </td>
   <td style="text-align:right;"> 1.938000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_7_3 </td>
   <td style="text-align:right;"> 4.093000e+01 </td>
   <td style="text-align:right;"> 3.645000e+01 </td>
   <td style="text-align:right;"> 2.658000e+01 </td>
   <td style="text-align:right;"> 6.30 </td>
   <td style="text-align:right;"> 5.587100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_0_0 </td>
   <td style="text-align:right;"> 2.615000e+01 </td>
   <td style="text-align:right;"> 1.723000e+01 </td>
   <td style="text-align:right;"> 2.060000e+01 </td>
   <td style="text-align:right;"> 3.01 </td>
   <td style="text-align:right;"> 2.691700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_7_0 </td>
   <td style="text-align:right;"> 1.735000e+01 </td>
   <td style="text-align:right;"> 1.671000e+01 </td>
   <td style="text-align:right;"> 1.015000e+01 </td>
   <td style="text-align:right;"> 0.85 </td>
   <td style="text-align:right;"> 3.669900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_7_1 </td>
   <td style="text-align:right;"> 2.813000e+01 </td>
   <td style="text-align:right;"> 2.593000e+01 </td>
   <td style="text-align:right;"> 1.757000e+01 </td>
   <td style="text-align:right;"> 3.66 </td>
   <td style="text-align:right;"> 4.461400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_3_0 </td>
   <td style="text-align:right;"> 1.505000e+01 </td>
   <td style="text-align:right;"> 1.296000e+01 </td>
   <td style="text-align:right;"> 1.066000e+01 </td>
   <td style="text-align:right;"> 0.50 </td>
   <td style="text-align:right;"> 2.081300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_5_2 </td>
   <td style="text-align:right;"> 3.493000e+01 </td>
   <td style="text-align:right;"> 2.908000e+01 </td>
   <td style="text-align:right;"> 2.470000e+01 </td>
   <td style="text-align:right;"> 4.58 </td>
   <td style="text-align:right;"> 4.551000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_6_1 </td>
   <td style="text-align:right;"> 3.166000e+01 </td>
   <td style="text-align:right;"> 2.848000e+01 </td>
   <td style="text-align:right;"> 2.085000e+01 </td>
   <td style="text-align:right;"> 1.81 </td>
   <td style="text-align:right;"> 4.762100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_3_1 </td>
   <td style="text-align:right;"> 2.437000e+01 </td>
   <td style="text-align:right;"> 1.902000e+01 </td>
   <td style="text-align:right;"> 1.807000e+01 </td>
   <td style="text-align:right;"> 2.51 </td>
   <td style="text-align:right;"> 2.972800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_6_0 </td>
   <td style="text-align:right;"> 1.479000e+01 </td>
   <td style="text-align:right;"> 1.402000e+01 </td>
   <td style="text-align:right;"> 9.890000e+00 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 2.990100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_2_1 </td>
   <td style="text-align:right;"> 3.825000e+01 </td>
   <td style="text-align:right;"> 2.770000e+01 </td>
   <td style="text-align:right;"> 2.870000e+01 </td>
   <td style="text-align:right;"> 2.75 </td>
   <td style="text-align:right;"> 4.208100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_6_3 </td>
   <td style="text-align:right;"> 4.648000e+01 </td>
   <td style="text-align:right;"> 4.064000e+01 </td>
   <td style="text-align:right;"> 3.112000e+01 </td>
   <td style="text-align:right;"> 4.11 </td>
   <td style="text-align:right;"> 6.084300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_2_0 </td>
   <td style="text-align:right;"> 2.809000e+01 </td>
   <td style="text-align:right;"> 1.984000e+01 </td>
   <td style="text-align:right;"> 2.179000e+01 </td>
   <td style="text-align:right;"> 1.32 </td>
   <td style="text-align:right;"> 3.265300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_6_2 </td>
   <td style="text-align:right;"> 4.197000e+01 </td>
   <td style="text-align:right;"> 3.728000e+01 </td>
   <td style="text-align:right;"> 2.791000e+01 </td>
   <td style="text-align:right;"> 2.94 </td>
   <td style="text-align:right;"> 5.622000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_5_0 </td>
   <td style="text-align:right;"> 1.827000e+01 </td>
   <td style="text-align:right;"> 1.712000e+01 </td>
   <td style="text-align:right;"> 1.218000e+01 </td>
   <td style="text-align:right;"> 0.88 </td>
   <td style="text-align:right;"> 3.155700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_5_1 </td>
   <td style="text-align:right;"> 2.887000e+01 </td>
   <td style="text-align:right;"> 2.459000e+01 </td>
   <td style="text-align:right;"> 2.043000e+01 </td>
   <td style="text-align:right;"> 3.46 </td>
   <td style="text-align:right;"> 4.074900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_4_2 </td>
   <td style="text-align:right;"> 4.374000e+01 </td>
   <td style="text-align:right;"> 3.569000e+01 </td>
   <td style="text-align:right;"> 3.133000e+01 </td>
   <td style="text-align:right;"> 3.58 </td>
   <td style="text-align:right;"> 5.344700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_1_0 </td>
   <td style="text-align:right;"> 1.430000e+00 </td>
   <td style="text-align:right;"> 2.100000e-01 </td>
   <td style="text-align:right;"> 1.410000e+00 </td>
   <td style="text-align:right;"> 0.74 </td>
   <td style="text-align:right;"> 2.400000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_4_1 </td>
   <td style="text-align:right;"> 3.763000e+01 </td>
   <td style="text-align:right;"> 3.120000e+01 </td>
   <td style="text-align:right;"> 2.698000e+01 </td>
   <td style="text-align:right;"> 1.95 </td>
   <td style="text-align:right;"> 4.656000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_7_2 </td>
   <td style="text-align:right;"> 3.638000e+01 </td>
   <td style="text-align:right;"> 3.305000e+01 </td>
   <td style="text-align:right;"> 2.330000e+01 </td>
   <td style="text-align:right;"> 5.96 </td>
   <td style="text-align:right;"> 5.303200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_shape_Z_4_0 </td>
   <td style="text-align:right;"> 2.046000e+01 </td>
   <td style="text-align:right;"> 1.753000e+01 </td>
   <td style="text-align:right;"> 1.485000e+01 </td>
   <td style="text-align:right;"> 0.03 </td>
   <td style="text-align:right;"> 3.130900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_7_3 </td>
   <td style="text-align:right;"> 3.030000e+01 </td>
   <td style="text-align:right;"> 2.754000e+01 </td>
   <td style="text-align:right;"> 1.942000e+01 </td>
   <td style="text-align:right;"> 2.89 </td>
   <td style="text-align:right;"> 2.059200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_0_0 </td>
   <td style="text-align:right;"> 1.904000e+01 </td>
   <td style="text-align:right;"> 1.262000e+01 </td>
   <td style="text-align:right;"> 1.517000e+01 </td>
   <td style="text-align:right;"> 0.85 </td>
   <td style="text-align:right;"> 1.147400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_7_0 </td>
   <td style="text-align:right;"> 1.486000e+01 </td>
   <td style="text-align:right;"> 1.374000e+01 </td>
   <td style="text-align:right;"> 8.520000e+00 </td>
   <td style="text-align:right;"> 0.98 </td>
   <td style="text-align:right;"> 1.269200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_7_1 </td>
   <td style="text-align:right;"> 2.230000e+01 </td>
   <td style="text-align:right;"> 2.042000e+01 </td>
   <td style="text-align:right;"> 1.391000e+01 </td>
   <td style="text-align:right;"> 2.88 </td>
   <td style="text-align:right;"> 1.599200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_3_0 </td>
   <td style="text-align:right;"> 1.129000e+01 </td>
   <td style="text-align:right;"> 9.710000e+00 </td>
   <td style="text-align:right;"> 7.760000e+00 </td>
   <td style="text-align:right;"> 0.42 </td>
   <td style="text-align:right;"> 8.854000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_5_2 </td>
   <td style="text-align:right;"> 2.552000e+01 </td>
   <td style="text-align:right;"> 2.161000e+01 </td>
   <td style="text-align:right;"> 1.772000e+01 </td>
   <td style="text-align:right;"> 2.15 </td>
   <td style="text-align:right;"> 1.821800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_6_1 </td>
   <td style="text-align:right;"> 2.469000e+01 </td>
   <td style="text-align:right;"> 2.254000e+01 </td>
   <td style="text-align:right;"> 1.713000e+01 </td>
   <td style="text-align:right;"> 0.51 </td>
   <td style="text-align:right;"> 1.746700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_3_1 </td>
   <td style="text-align:right;"> 1.729000e+01 </td>
   <td style="text-align:right;"> 1.388000e+01 </td>
   <td style="text-align:right;"> 1.244000e+01 </td>
   <td style="text-align:right;"> 1.44 </td>
   <td style="text-align:right;"> 1.208600e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_6_0 </td>
   <td style="text-align:right;"> 1.253000e+01 </td>
   <td style="text-align:right;"> 1.254000e+01 </td>
   <td style="text-align:right;"> 8.040000e+00 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.224900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_2_1 </td>
   <td style="text-align:right;"> 2.801000e+01 </td>
   <td style="text-align:right;"> 2.003000e+01 </td>
   <td style="text-align:right;"> 2.147000e+01 </td>
   <td style="text-align:right;"> 0.91 </td>
   <td style="text-align:right;"> 1.762500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_6_3 </td>
   <td style="text-align:right;"> 3.422000e+01 </td>
   <td style="text-align:right;"> 3.068000e+01 </td>
   <td style="text-align:right;"> 2.352000e+01 </td>
   <td style="text-align:right;"> 1.16 </td>
   <td style="text-align:right;"> 2.600800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_2_0 </td>
   <td style="text-align:right;"> 2.156000e+01 </td>
   <td style="text-align:right;"> 1.486000e+01 </td>
   <td style="text-align:right;"> 1.696000e+01 </td>
   <td style="text-align:right;"> 0.51 </td>
   <td style="text-align:right;"> 1.352800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_6_2 </td>
   <td style="text-align:right;"> 3.148000e+01 </td>
   <td style="text-align:right;"> 2.846000e+01 </td>
   <td style="text-align:right;"> 2.167000e+01 </td>
   <td style="text-align:right;"> 0.85 </td>
   <td style="text-align:right;"> 2.355200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_5_0 </td>
   <td style="text-align:right;"> 1.489000e+01 </td>
   <td style="text-align:right;"> 1.344000e+01 </td>
   <td style="text-align:right;"> 9.730000e+00 </td>
   <td style="text-align:right;"> 0.87 </td>
   <td style="text-align:right;"> 1.182300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_5_1 </td>
   <td style="text-align:right;"> 2.185000e+01 </td>
   <td style="text-align:right;"> 1.860000e+01 </td>
   <td style="text-align:right;"> 1.519000e+01 </td>
   <td style="text-align:right;"> 2.14 </td>
   <td style="text-align:right;"> 1.670800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_4_2 </td>
   <td style="text-align:right;"> 3.225000e+01 </td>
   <td style="text-align:right;"> 2.614000e+01 </td>
   <td style="text-align:right;"> 2.349000e+01 </td>
   <td style="text-align:right;"> 1.01 </td>
   <td style="text-align:right;"> 2.284400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_1_0 </td>
   <td style="text-align:right;"> 1.420000e+00 </td>
   <td style="text-align:right;"> 2.200000e-01 </td>
   <td style="text-align:right;"> 1.390000e+00 </td>
   <td style="text-align:right;"> 0.68 </td>
   <td style="text-align:right;"> 2.400000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_4_1 </td>
   <td style="text-align:right;"> 2.851000e+01 </td>
   <td style="text-align:right;"> 2.309000e+01 </td>
   <td style="text-align:right;"> 2.093000e+01 </td>
   <td style="text-align:right;"> 0.76 </td>
   <td style="text-align:right;"> 1.810600e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_7_2 </td>
   <td style="text-align:right;"> 2.760000e+01 </td>
   <td style="text-align:right;"> 2.528000e+01 </td>
   <td style="text-align:right;"> 1.751000e+01 </td>
   <td style="text-align:right;"> 2.89 </td>
   <td style="text-align:right;"> 1.951200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_00_density_Z_4_0 </td>
   <td style="text-align:right;"> 1.700000e+01 </td>
   <td style="text-align:right;"> 1.404000e+01 </td>
   <td style="text-align:right;"> 1.273000e+01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.196500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_segments_count </td>
   <td style="text-align:right;"> 2.830100e+02 </td>
   <td style="text-align:right;"> 9.662200e+02 </td>
   <td style="text-align:right;"> 1.500000e+01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 6.920200e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_segments_count </td>
   <td style="text-align:right;"> 2.830100e+02 </td>
   <td style="text-align:right;"> 9.662200e+02 </td>
   <td style="text-align:right;"> 1.500000e+01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 6.920200e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_volume </td>
   <td style="text-align:right;"> 2.525000e+01 </td>
   <td style="text-align:right;"> 4.106000e+01 </td>
   <td style="text-align:right;"> 1.031000e+01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.996250e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_electrons </td>
   <td style="text-align:right;"> 1.504000e+01 </td>
   <td style="text-align:right;"> 2.287000e+01 </td>
   <td style="text-align:right;"> 6.120000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.957000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_mean </td>
   <td style="text-align:right;"> 6.500000e-01 </td>
   <td style="text-align:right;"> 4.300000e-01 </td>
   <td style="text-align:right;"> 5.600000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 8.860000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_std </td>
   <td style="text-align:right;"> 2.000000e-01 </td>
   <td style="text-align:right;"> 3.000000e-01 </td>
   <td style="text-align:right;"> 1.100000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 8.080000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_max </td>
   <td style="text-align:right;"> 1.350000e+00 </td>
   <td style="text-align:right;"> 1.590000e+00 </td>
   <td style="text-align:right;"> 8.900000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 4.463000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_max_over_std </td>
   <td style="text-align:right;"> 9.710000e+00 </td>
   <td style="text-align:right;"> 7.640000e+00 </td>
   <td style="text-align:right;"> 7.220000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.732500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_skewness </td>
   <td style="text-align:right;"> 2.000000e-01 </td>
   <td style="text-align:right;"> 3.400000e-01 </td>
   <td style="text-align:right;"> 1.000000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.077000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_parts </td>
   <td style="text-align:right;"> 1.270000e+00 </td>
   <td style="text-align:right;"> 7.000000e-01 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.400000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_O3 </td>
   <td style="text-align:right;"> 1.276332e+06 </td>
   <td style="text-align:right;"> 7.453268e+06 </td>
   <td style="text-align:right;"> 5.937043e+04 </td>
   <td style="text-align:right;"> 74.84 </td>
   <td style="text-align:right;"> 1.837172e+09 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_O4 </td>
   <td style="text-align:right;"> 5.888376e+12 </td>
   <td style="text-align:right;"> 4.679577e+14 </td>
   <td style="text-align:right;"> 8.720523e+08 </td>
   <td style="text-align:right;"> 1818.62 </td>
   <td style="text-align:right;"> 1.680769e+17 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_O5 </td>
   <td style="text-align:right;"> 5.793748e+19 </td>
   <td style="text-align:right;"> 1.314826e+22 </td>
   <td style="text-align:right;"> 3.512181e+12 </td>
   <td style="text-align:right;"> 13532.12 </td>
   <td style="text-align:right;"> 5.890566e+24 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_FL </td>
   <td style="text-align:right;"> 3.084952e+16 </td>
   <td style="text-align:right;"> 7.231544e+18 </td>
   <td style="text-align:right;"> 1.046845e+10 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.872585e+21 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_O3_norm </td>
   <td style="text-align:right;"> 5.300000e-01 </td>
   <td style="text-align:right;"> 4.300000e-01 </td>
   <td style="text-align:right;"> 3.700000e-01 </td>
   <td style="text-align:right;"> 0.23 </td>
   <td style="text-align:right;"> 4.375000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_O4_norm </td>
   <td style="text-align:right;"> 7.000000e-02 </td>
   <td style="text-align:right;"> 1.100000e-01 </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 1.175000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_O5_norm </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.230000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_FL_norm </td>
   <td style="text-align:right;"> 1.400000e-01 </td>
   <td style="text-align:right;"> 2.230000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 5.526100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I1 </td>
   <td style="text-align:right;"> 2.410654e+09 </td>
   <td style="text-align:right;"> 2.754567e+11 </td>
   <td style="text-align:right;"> 3.905652e+06 </td>
   <td style="text-align:right;"> 210.50 </td>
   <td style="text-align:right;"> 1.307399e+14 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I2 </td>
   <td style="text-align:right;"> 1.474496e+20 </td>
   <td style="text-align:right;"> 4.067568e+22 </td>
   <td style="text-align:right;"> 2.238828e+12 </td>
   <td style="text-align:right;"> 10919.68 </td>
   <td style="text-align:right;"> 1.923967e+25 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I3 </td>
   <td style="text-align:right;"> 7.544764e+22 </td>
   <td style="text-align:right;"> 3.200528e+25 </td>
   <td style="text-align:right;"> 6.116160e+12 </td>
   <td style="text-align:right;"> 9454.58 </td>
   <td style="text-align:right;"> 1.708172e+28 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I4 </td>
   <td style="text-align:right;"> 1.783789e+16 </td>
   <td style="text-align:right;"> 3.751691e+18 </td>
   <td style="text-align:right;"> 5.856015e+09 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.407712e+21 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I5 </td>
   <td style="text-align:right;"> 9.163466e+15 </td>
   <td style="text-align:right;"> 1.520986e+18 </td>
   <td style="text-align:right;"> 1.619440e+09 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 4.931548e+20 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I6 </td>
   <td style="text-align:right;"> 1.273856e+18 </td>
   <td style="text-align:right;"> 4.557894e+20 </td>
   <td style="text-align:right;"> 1.045825e+11 </td>
   <td style="text-align:right;"> 5359.46 </td>
   <td style="text-align:right;"> 2.400810e+23 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I1_norm </td>
   <td style="text-align:right;"> 7.700000e-01 </td>
   <td style="text-align:right;"> 8.740000e+00 </td>
   <td style="text-align:right;"> 2.200000e-01 </td>
   <td style="text-align:right;"> 0.06 </td>
   <td style="text-align:right;"> 3.426140e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I2_norm </td>
   <td style="text-align:right;"> 2.400000e-01 </td>
   <td style="text-align:right;"> 1.197000e+01 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 6.729610e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I3_norm </td>
   <td style="text-align:right;"> 7.693000e+01 </td>
   <td style="text-align:right;"> 2.330541e+04 </td>
   <td style="text-align:right;"> 2.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.173424e+07 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I4_norm </td>
   <td style="text-align:right;"> 1.100000e-01 </td>
   <td style="text-align:right;"> 2.270000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 6.056100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I5_norm </td>
   <td style="text-align:right;"> 1.000000e-01 </td>
   <td style="text-align:right;"> 2.310000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 6.409500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_I6_norm </td>
   <td style="text-align:right;"> 1.790000e+00 </td>
   <td style="text-align:right;"> 3.274700e+02 </td>
   <td style="text-align:right;"> 4.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.498618e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_M000 </td>
   <td style="text-align:right;"> 3.186450e+03 </td>
   <td style="text-align:right;"> 5.123700e+03 </td>
   <td style="text-align:right;"> 1.328000e+03 </td>
   <td style="text-align:right;"> 32.00 </td>
   <td style="text-align:right;"> 2.495310e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_CI </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 4.530000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> -142.64 </td>
   <td style="text-align:right;"> 7.127000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_E3_E1 </td>
   <td style="text-align:right;"> 2.500000e-01 </td>
   <td style="text-align:right;"> 2.100000e-01 </td>
   <td style="text-align:right;"> 1.800000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 9.900000e-01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_E2_E1 </td>
   <td style="text-align:right;"> 4.300000e-01 </td>
   <td style="text-align:right;"> 2.500000e-01 </td>
   <td style="text-align:right;"> 4.000000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_E3_E2 </td>
   <td style="text-align:right;"> 5.700000e-01 </td>
   <td style="text-align:right;"> 2.300000e-01 </td>
   <td style="text-align:right;"> 5.900000e-01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_sqrt_E1 </td>
   <td style="text-align:right;"> 7.450000e+00 </td>
   <td style="text-align:right;"> 5.990000e+00 </td>
   <td style="text-align:right;"> 5.270000e+00 </td>
   <td style="text-align:right;"> 0.93 </td>
   <td style="text-align:right;"> 2.023700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_sqrt_E2 </td>
   <td style="text-align:right;"> 4.030000e+00 </td>
   <td style="text-align:right;"> 2.730000e+00 </td>
   <td style="text-align:right;"> 3.160000e+00 </td>
   <td style="text-align:right;"> 0.53 </td>
   <td style="text-align:right;"> 3.206000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_sqrt_E3 </td>
   <td style="text-align:right;"> 2.660000e+00 </td>
   <td style="text-align:right;"> 1.410000e+00 </td>
   <td style="text-align:right;"> 2.350000e+00 </td>
   <td style="text-align:right;"> 0.37 </td>
   <td style="text-align:right;"> 1.906000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_O3 </td>
   <td style="text-align:right;"> 6.719290e+05 </td>
   <td style="text-align:right;"> 2.504688e+06 </td>
   <td style="text-align:right;"> 3.307433e+04 </td>
   <td style="text-align:right;"> 5.07 </td>
   <td style="text-align:right;"> 3.763261e+08 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_O4 </td>
   <td style="text-align:right;"> 9.962197e+11 </td>
   <td style="text-align:right;"> 2.296292e+13 </td>
   <td style="text-align:right;"> 2.639872e+08 </td>
   <td style="text-align:right;"> 6.52 </td>
   <td style="text-align:right;"> 8.774435e+15 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_O5 </td>
   <td style="text-align:right;"> 9.393053e+17 </td>
   <td style="text-align:right;"> 8.694761e+19 </td>
   <td style="text-align:right;"> 5.760985e+11 </td>
   <td style="text-align:right;"> 1.74 </td>
   <td style="text-align:right;"> 3.619558e+22 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_FL </td>
   <td style="text-align:right;"> 2.372681e+15 </td>
   <td style="text-align:right;"> 4.490618e+17 </td>
   <td style="text-align:right;"> 2.842058e+09 </td>
   <td style="text-align:right;"> -11.69 </td>
   <td style="text-align:right;"> 2.206107e+20 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_O3_norm </td>
   <td style="text-align:right;"> 7.700000e-01 </td>
   <td style="text-align:right;"> 1.090000e+00 </td>
   <td style="text-align:right;"> 5.700000e-01 </td>
   <td style="text-align:right;"> 0.04 </td>
   <td style="text-align:right;"> 3.097900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_O4_norm </td>
   <td style="text-align:right;"> 1.600000e-01 </td>
   <td style="text-align:right;"> 3.000000e-01 </td>
   <td style="text-align:right;"> 8.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.095000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_O5_norm </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.740000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_FL_norm </td>
   <td style="text-align:right;"> 7.900000e-01 </td>
   <td style="text-align:right;"> 2.885000e+01 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> -0.04 </td>
   <td style="text-align:right;"> 1.474174e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I1 </td>
   <td style="text-align:right;"> 8.851343e+08 </td>
   <td style="text-align:right;"> 2.810244e+10 </td>
   <td style="text-align:right;"> 2.011426e+06 </td>
   <td style="text-align:right;"> 23.86 </td>
   <td style="text-align:right;"> 1.098793e+13 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I2 </td>
   <td style="text-align:right;"> 8.447497e+18 </td>
   <td style="text-align:right;"> 2.080879e+21 </td>
   <td style="text-align:right;"> 5.998141e+11 </td>
   <td style="text-align:right;"> 81.51 </td>
   <td style="text-align:right;"> 9.956763e+23 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I3 </td>
   <td style="text-align:right;"> 7.378980e+20 </td>
   <td style="text-align:right;"> 2.363363e+23 </td>
   <td style="text-align:right;"> 1.584537e+12 </td>
   <td style="text-align:right;"> 186.52 </td>
   <td style="text-align:right;"> 1.206633e+26 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I4 </td>
   <td style="text-align:right;"> 1.469681e+15 </td>
   <td style="text-align:right;"> 2.535361e+17 </td>
   <td style="text-align:right;"> 1.703236e+09 </td>
   <td style="text-align:right;"> -3.02 </td>
   <td style="text-align:right;"> 1.278331e+20 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I5 </td>
   <td style="text-align:right;"> 8.676808e+14 </td>
   <td style="text-align:right;"> 1.250201e+17 </td>
   <td style="text-align:right;"> 6.184899e+08 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 6.598137e+19 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I6 </td>
   <td style="text-align:right;"> 2.664503e+16 </td>
   <td style="text-align:right;"> 5.527029e+18 </td>
   <td style="text-align:right;"> 2.976817e+10 </td>
   <td style="text-align:right;"> 63.49 </td>
   <td style="text-align:right;"> 2.140061e+21 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I1_norm </td>
   <td style="text-align:right;"> 3.090000e+00 </td>
   <td style="text-align:right;"> 3.784700e+02 </td>
   <td style="text-align:right;"> 5.100000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.711294e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I2_norm </td>
   <td style="text-align:right;"> 2.250000e+00 </td>
   <td style="text-align:right;"> 1.563800e+02 </td>
   <td style="text-align:right;"> 4.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 7.025807e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I3_norm </td>
   <td style="text-align:right;"> 1.445226e+05 </td>
   <td style="text-align:right;"> 5.385298e+07 </td>
   <td style="text-align:right;"> 1.100000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.926873e+10 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I4_norm </td>
   <td style="text-align:right;"> 7.200000e-01 </td>
   <td style="text-align:right;"> 2.927000e+01 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> -0.02 </td>
   <td style="text-align:right;"> 1.478973e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I5_norm </td>
   <td style="text-align:right;"> 6.700000e-01 </td>
   <td style="text-align:right;"> 2.965000e+01 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.482173e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_I6_norm </td>
   <td style="text-align:right;"> 3.171300e+02 </td>
   <td style="text-align:right;"> 1.042647e+05 </td>
   <td style="text-align:right;"> 1.300000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 5.299373e+07 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_M000 </td>
   <td style="text-align:right;"> 1.897890e+03 </td>
   <td style="text-align:right;"> 2.852510e+03 </td>
   <td style="text-align:right;"> 7.915200e+02 </td>
   <td style="text-align:right;"> 1.44 </td>
   <td style="text-align:right;"> 4.946192e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_CI </td>
   <td style="text-align:right;"> 4.000000e-02 </td>
   <td style="text-align:right;"> 5.090000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> -162.58 </td>
   <td style="text-align:right;"> 1.049000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_E3_E1 </td>
   <td style="text-align:right;"> 2.600000e-01 </td>
   <td style="text-align:right;"> 2.100000e-01 </td>
   <td style="text-align:right;"> 1.800000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_E2_E1 </td>
   <td style="text-align:right;"> 4.300000e-01 </td>
   <td style="text-align:right;"> 2.500000e-01 </td>
   <td style="text-align:right;"> 4.000000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_E3_E2 </td>
   <td style="text-align:right;"> 5.700000e-01 </td>
   <td style="text-align:right;"> 2.400000e-01 </td>
   <td style="text-align:right;"> 5.900000e-01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_sqrt_E1 </td>
   <td style="text-align:right;"> 7.190000e+00 </td>
   <td style="text-align:right;"> 5.870000e+00 </td>
   <td style="text-align:right;"> 4.960000e+00 </td>
   <td style="text-align:right;"> 0.93 </td>
   <td style="text-align:right;"> 2.021700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_sqrt_E2 </td>
   <td style="text-align:right;"> 3.840000e+00 </td>
   <td style="text-align:right;"> 2.640000e+00 </td>
   <td style="text-align:right;"> 2.970000e+00 </td>
   <td style="text-align:right;"> 0.53 </td>
   <td style="text-align:right;"> 3.077000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_sqrt_E3 </td>
   <td style="text-align:right;"> 2.530000e+00 </td>
   <td style="text-align:right;"> 1.340000e+00 </td>
   <td style="text-align:right;"> 2.230000e+00 </td>
   <td style="text-align:right;"> 0.37 </td>
   <td style="text-align:right;"> 1.869000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_7_3 </td>
   <td style="text-align:right;"> 3.578000e+01 </td>
   <td style="text-align:right;"> 3.373000e+01 </td>
   <td style="text-align:right;"> 2.195000e+01 </td>
   <td style="text-align:right;"> 4.61 </td>
   <td style="text-align:right;"> 4.702800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_0_0 </td>
   <td style="text-align:right;"> 2.243000e+01 </td>
   <td style="text-align:right;"> 1.598000e+01 </td>
   <td style="text-align:right;"> 1.781000e+01 </td>
   <td style="text-align:right;"> 2.76 </td>
   <td style="text-align:right;"> 2.440700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_7_0 </td>
   <td style="text-align:right;"> 1.597000e+01 </td>
   <td style="text-align:right;"> 1.540000e+01 </td>
   <td style="text-align:right;"> 8.700000e+00 </td>
   <td style="text-align:right;"> 0.71 </td>
   <td style="text-align:right;"> 2.915600e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_7_1 </td>
   <td style="text-align:right;"> 2.494000e+01 </td>
   <td style="text-align:right;"> 2.397000e+01 </td>
   <td style="text-align:right;"> 1.439000e+01 </td>
   <td style="text-align:right;"> 3.42 </td>
   <td style="text-align:right;"> 3.709900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_3_0 </td>
   <td style="text-align:right;"> 1.341000e+01 </td>
   <td style="text-align:right;"> 1.216000e+01 </td>
   <td style="text-align:right;"> 9.020000e+00 </td>
   <td style="text-align:right;"> 0.63 </td>
   <td style="text-align:right;"> 1.917600e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_5_2 </td>
   <td style="text-align:right;"> 3.023000e+01 </td>
   <td style="text-align:right;"> 2.696000e+01 </td>
   <td style="text-align:right;"> 2.051000e+01 </td>
   <td style="text-align:right;"> 3.10 </td>
   <td style="text-align:right;"> 4.033000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_6_1 </td>
   <td style="text-align:right;"> 2.730000e+01 </td>
   <td style="text-align:right;"> 2.665000e+01 </td>
   <td style="text-align:right;"> 1.695000e+01 </td>
   <td style="text-align:right;"> 0.79 </td>
   <td style="text-align:right;"> 3.717000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_3_1 </td>
   <td style="text-align:right;"> 2.127000e+01 </td>
   <td style="text-align:right;"> 1.774000e+01 </td>
   <td style="text-align:right;"> 1.547000e+01 </td>
   <td style="text-align:right;"> 2.42 </td>
   <td style="text-align:right;"> 2.606700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_6_0 </td>
   <td style="text-align:right;"> 1.306000e+01 </td>
   <td style="text-align:right;"> 1.346000e+01 </td>
   <td style="text-align:right;"> 8.210000e+00 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 2.636200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_2_1 </td>
   <td style="text-align:right;"> 3.264000e+01 </td>
   <td style="text-align:right;"> 2.569000e+01 </td>
   <td style="text-align:right;"> 2.422000e+01 </td>
   <td style="text-align:right;"> 1.71 </td>
   <td style="text-align:right;"> 3.463800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_6_3 </td>
   <td style="text-align:right;"> 4.009000e+01 </td>
   <td style="text-align:right;"> 3.784000e+01 </td>
   <td style="text-align:right;"> 2.567000e+01 </td>
   <td style="text-align:right;"> 3.40 </td>
   <td style="text-align:right;"> 5.043900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_2_0 </td>
   <td style="text-align:right;"> 2.378000e+01 </td>
   <td style="text-align:right;"> 1.850000e+01 </td>
   <td style="text-align:right;"> 1.816000e+01 </td>
   <td style="text-align:right;"> 0.05 </td>
   <td style="text-align:right;"> 2.846000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_6_2 </td>
   <td style="text-align:right;"> 3.602000e+01 </td>
   <td style="text-align:right;"> 3.467000e+01 </td>
   <td style="text-align:right;"> 2.267000e+01 </td>
   <td style="text-align:right;"> 2.46 </td>
   <td style="text-align:right;"> 4.666900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_5_0 </td>
   <td style="text-align:right;"> 1.638000e+01 </td>
   <td style="text-align:right;"> 1.587000e+01 </td>
   <td style="text-align:right;"> 9.780000e+00 </td>
   <td style="text-align:right;"> 0.77 </td>
   <td style="text-align:right;"> 2.952300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_5_1 </td>
   <td style="text-align:right;"> 2.492000e+01 </td>
   <td style="text-align:right;"> 2.270000e+01 </td>
   <td style="text-align:right;"> 1.669000e+01 </td>
   <td style="text-align:right;"> 2.41 </td>
   <td style="text-align:right;"> 3.681300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_4_2 </td>
   <td style="text-align:right;"> 3.730000e+01 </td>
   <td style="text-align:right;"> 3.316000e+01 </td>
   <td style="text-align:right;"> 2.569000e+01 </td>
   <td style="text-align:right;"> 2.22 </td>
   <td style="text-align:right;"> 4.224900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_1_0 </td>
   <td style="text-align:right;"> 1.540000e+00 </td>
   <td style="text-align:right;"> 3.100000e-01 </td>
   <td style="text-align:right;"> 1.500000e+00 </td>
   <td style="text-align:right;"> 0.70 </td>
   <td style="text-align:right;"> 4.280000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_4_1 </td>
   <td style="text-align:right;"> 3.183000e+01 </td>
   <td style="text-align:right;"> 2.898000e+01 </td>
   <td style="text-align:right;"> 2.177000e+01 </td>
   <td style="text-align:right;"> 1.14 </td>
   <td style="text-align:right;"> 3.746300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_7_2 </td>
   <td style="text-align:right;"> 3.171000e+01 </td>
   <td style="text-align:right;"> 3.046000e+01 </td>
   <td style="text-align:right;"> 1.890000e+01 </td>
   <td style="text-align:right;"> 4.02 </td>
   <td style="text-align:right;"> 4.472700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_shape_Z_4_0 </td>
   <td style="text-align:right;"> 1.736000e+01 </td>
   <td style="text-align:right;"> 1.655000e+01 </td>
   <td style="text-align:right;"> 1.171000e+01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 2.751700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_7_3 </td>
   <td style="text-align:right;"> 2.800000e+01 </td>
   <td style="text-align:right;"> 2.657000e+01 </td>
   <td style="text-align:right;"> 1.657000e+01 </td>
   <td style="text-align:right;"> 2.60 </td>
   <td style="text-align:right;"> 2.030500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_0_0 </td>
   <td style="text-align:right;"> 1.727000e+01 </td>
   <td style="text-align:right;"> 1.238000e+01 </td>
   <td style="text-align:right;"> 1.375000e+01 </td>
   <td style="text-align:right;"> 0.59 </td>
   <td style="text-align:right;"> 1.086700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_7_0 </td>
   <td style="text-align:right;"> 1.425000e+01 </td>
   <td style="text-align:right;"> 1.315000e+01 </td>
   <td style="text-align:right;"> 7.810000e+00 </td>
   <td style="text-align:right;"> 1.06 </td>
   <td style="text-align:right;"> 1.251700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_7_1 </td>
   <td style="text-align:right;"> 2.077000e+01 </td>
   <td style="text-align:right;"> 1.963000e+01 </td>
   <td style="text-align:right;"> 1.162000e+01 </td>
   <td style="text-align:right;"> 1.91 </td>
   <td style="text-align:right;"> 1.556800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_3_0 </td>
   <td style="text-align:right;"> 1.067000e+01 </td>
   <td style="text-align:right;"> 9.470000e+00 </td>
   <td style="text-align:right;"> 6.910000e+00 </td>
   <td style="text-align:right;"> 0.44 </td>
   <td style="text-align:right;"> 8.632000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_5_2 </td>
   <td style="text-align:right;"> 2.343000e+01 </td>
   <td style="text-align:right;"> 2.094000e+01 </td>
   <td style="text-align:right;"> 1.539000e+01 </td>
   <td style="text-align:right;"> 2.07 </td>
   <td style="text-align:right;"> 1.804900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_6_1 </td>
   <td style="text-align:right;"> 2.208000e+01 </td>
   <td style="text-align:right;"> 2.202000e+01 </td>
   <td style="text-align:right;"> 1.397000e+01 </td>
   <td style="text-align:right;"> 0.37 </td>
   <td style="text-align:right;"> 1.685900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_3_1 </td>
   <td style="text-align:right;"> 1.606000e+01 </td>
   <td style="text-align:right;"> 1.353000e+01 </td>
   <td style="text-align:right;"> 1.119000e+01 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 1.174500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_6_0 </td>
   <td style="text-align:right;"> 1.133000e+01 </td>
   <td style="text-align:right;"> 1.240000e+01 </td>
   <td style="text-align:right;"> 6.270000e+00 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.174500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_2_1 </td>
   <td style="text-align:right;"> 2.527000e+01 </td>
   <td style="text-align:right;"> 1.959000e+01 </td>
   <td style="text-align:right;"> 1.919000e+01 </td>
   <td style="text-align:right;"> 0.71 </td>
   <td style="text-align:right;"> 1.709200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_6_3 </td>
   <td style="text-align:right;"> 3.089000e+01 </td>
   <td style="text-align:right;"> 2.989000e+01 </td>
   <td style="text-align:right;"> 1.986000e+01 </td>
   <td style="text-align:right;"> 1.05 </td>
   <td style="text-align:right;"> 2.519900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_2_0 </td>
   <td style="text-align:right;"> 1.929000e+01 </td>
   <td style="text-align:right;"> 1.465000e+01 </td>
   <td style="text-align:right;"> 1.493000e+01 </td>
   <td style="text-align:right;"> 0.06 </td>
   <td style="text-align:right;"> 1.281300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_6_2 </td>
   <td style="text-align:right;"> 2.821000e+01 </td>
   <td style="text-align:right;"> 2.768000e+01 </td>
   <td style="text-align:right;"> 1.804000e+01 </td>
   <td style="text-align:right;"> 0.88 </td>
   <td style="text-align:right;"> 2.259600e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_5_0 </td>
   <td style="text-align:right;"> 1.400000e+01 </td>
   <td style="text-align:right;"> 1.296000e+01 </td>
   <td style="text-align:right;"> 8.280000e+00 </td>
   <td style="text-align:right;"> 0.79 </td>
   <td style="text-align:right;"> 1.139400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_5_1 </td>
   <td style="text-align:right;"> 1.998000e+01 </td>
   <td style="text-align:right;"> 1.795000e+01 </td>
   <td style="text-align:right;"> 1.294000e+01 </td>
   <td style="text-align:right;"> 1.93 </td>
   <td style="text-align:right;"> 1.648700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_4_2 </td>
   <td style="text-align:right;"> 2.887000e+01 </td>
   <td style="text-align:right;"> 2.558000e+01 </td>
   <td style="text-align:right;"> 2.038000e+01 </td>
   <td style="text-align:right;"> 0.84 </td>
   <td style="text-align:right;"> 2.217200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_1_0 </td>
   <td style="text-align:right;"> 1.530000e+00 </td>
   <td style="text-align:right;"> 3.100000e-01 </td>
   <td style="text-align:right;"> 1.490000e+00 </td>
   <td style="text-align:right;"> 0.62 </td>
   <td style="text-align:right;"> 4.290000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_4_1 </td>
   <td style="text-align:right;"> 2.527000e+01 </td>
   <td style="text-align:right;"> 2.262000e+01 </td>
   <td style="text-align:right;"> 1.793000e+01 </td>
   <td style="text-align:right;"> 0.47 </td>
   <td style="text-align:right;"> 1.753100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_7_2 </td>
   <td style="text-align:right;"> 2.541000e+01 </td>
   <td style="text-align:right;"> 2.429000e+01 </td>
   <td style="text-align:right;"> 1.469000e+01 </td>
   <td style="text-align:right;"> 2.26 </td>
   <td style="text-align:right;"> 1.917000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_01_density_Z_4_0 </td>
   <td style="text-align:right;"> 1.495000e+01 </td>
   <td style="text-align:right;"> 1.397000e+01 </td>
   <td style="text-align:right;"> 1.057000e+01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.188000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_segments_count </td>
   <td style="text-align:right;"> 2.365800e+02 </td>
   <td style="text-align:right;"> 8.451900e+02 </td>
   <td style="text-align:right;"> 9.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 4.556400e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_segments_count </td>
   <td style="text-align:right;"> 2.365800e+02 </td>
   <td style="text-align:right;"> 8.451900e+02 </td>
   <td style="text-align:right;"> 9.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 4.556400e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_volume </td>
   <td style="text-align:right;"> 1.947000e+01 </td>
   <td style="text-align:right;"> 3.372000e+01 </td>
   <td style="text-align:right;"> 7.350000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.632540e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_electrons </td>
   <td style="text-align:right;"> 1.284000e+01 </td>
   <td style="text-align:right;"> 2.064000e+01 </td>
   <td style="text-align:right;"> 4.730000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.511900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_mean </td>
   <td style="text-align:right;"> 6.800000e-01 </td>
   <td style="text-align:right;"> 4.800000e-01 </td>
   <td style="text-align:right;"> 6.000000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 9.760000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_std </td>
   <td style="text-align:right;"> 1.900000e-01 </td>
   <td style="text-align:right;"> 3.100000e-01 </td>
   <td style="text-align:right;"> 9.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 8.260000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_max </td>
   <td style="text-align:right;"> 1.320000e+00 </td>
   <td style="text-align:right;"> 1.600000e+00 </td>
   <td style="text-align:right;"> 8.800000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 4.463000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_max_over_std </td>
   <td style="text-align:right;"> 9.480000e+00 </td>
   <td style="text-align:right;"> 7.860000e+00 </td>
   <td style="text-align:right;"> 7.220000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.732500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_skewness </td>
   <td style="text-align:right;"> 1.900000e-01 </td>
   <td style="text-align:right;"> 3.400000e-01 </td>
   <td style="text-align:right;"> 8.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.089000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_parts </td>
   <td style="text-align:right;"> 1.300000e+00 </td>
   <td style="text-align:right;"> 9.500000e-01 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.600000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_O3 </td>
   <td style="text-align:right;"> 1.014612e+06 </td>
   <td style="text-align:right;"> 5.732035e+06 </td>
   <td style="text-align:right;"> 4.992546e+04 </td>
   <td style="text-align:right;"> 72.00 </td>
   <td style="text-align:right;"> 1.487394e+09 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_O4 </td>
   <td style="text-align:right;"> 3.440276e+12 </td>
   <td style="text-align:right;"> 2.387292e+14 </td>
   <td style="text-align:right;"> 6.119596e+08 </td>
   <td style="text-align:right;"> 1728.00 </td>
   <td style="text-align:right;"> 8.781097e+16 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_O5 </td>
   <td style="text-align:right;"> 2.085633e+19 </td>
   <td style="text-align:right;"> 4.284804e+21 </td>
   <td style="text-align:right;"> 2.073642e+12 </td>
   <td style="text-align:right;"> 12288.00 </td>
   <td style="text-align:right;"> 2.033831e+24 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_FL </td>
   <td style="text-align:right;"> 1.662951e+16 </td>
   <td style="text-align:right;"> 3.744737e+18 </td>
   <td style="text-align:right;"> 6.558381e+09 </td>
   <td style="text-align:right;"> -61.16 </td>
   <td style="text-align:right;"> 1.394306e+21 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_O3_norm </td>
   <td style="text-align:right;"> 5.800000e-01 </td>
   <td style="text-align:right;"> 5.400000e-01 </td>
   <td style="text-align:right;"> 3.700000e-01 </td>
   <td style="text-align:right;"> 0.22 </td>
   <td style="text-align:right;"> 6.888000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_O4_norm </td>
   <td style="text-align:right;"> 9.000000e-02 </td>
   <td style="text-align:right;"> 1.700000e-01 </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 2.169000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_O5_norm </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 2.000000e-02 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 6.420000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_FL_norm </td>
   <td style="text-align:right;"> 3.600000e-01 </td>
   <td style="text-align:right;"> 9.310000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.374840e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I1 </td>
   <td style="text-align:right;"> 1.885939e+09 </td>
   <td style="text-align:right;"> 2.150612e+11 </td>
   <td style="text-align:right;"> 3.103373e+06 </td>
   <td style="text-align:right;"> 186.00 </td>
   <td style="text-align:right;"> 1.057686e+14 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I2 </td>
   <td style="text-align:right;"> 8.098471e+19 </td>
   <td style="text-align:right;"> 2.193735e+22 </td>
   <td style="text-align:right;"> 1.384962e+12 </td>
   <td style="text-align:right;"> 9312.00 </td>
   <td style="text-align:right;"> 1.018994e+25 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I3 </td>
   <td style="text-align:right;"> 4.906230e+22 </td>
   <td style="text-align:right;"> 2.008981e+25 </td>
   <td style="text-align:right;"> 3.838129e+12 </td>
   <td style="text-align:right;"> 7092.00 </td>
   <td style="text-align:right;"> 1.118111e+28 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I4 </td>
   <td style="text-align:right;"> 9.749810e+15 </td>
   <td style="text-align:right;"> 1.969465e+18 </td>
   <td style="text-align:right;"> 3.736084e+09 </td>
   <td style="text-align:right;"> -21.56 </td>
   <td style="text-align:right;"> 7.546862e+20 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I5 </td>
   <td style="text-align:right;"> 5.163344e+15 </td>
   <td style="text-align:right;"> 8.306804e+17 </td>
   <td style="text-align:right;"> 1.020489e+09 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.282730e+20 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I6 </td>
   <td style="text-align:right;"> 8.204262e+17 </td>
   <td style="text-align:right;"> 2.860271e+20 </td>
   <td style="text-align:right;"> 6.931835e+10 </td>
   <td style="text-align:right;"> 4464.00 </td>
   <td style="text-align:right;"> 1.572613e+23 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I1_norm </td>
   <td style="text-align:right;"> 1.060000e+00 </td>
   <td style="text-align:right;"> 1.552000e+01 </td>
   <td style="text-align:right;"> 2.300000e-01 </td>
   <td style="text-align:right;"> 0.06 </td>
   <td style="text-align:right;"> 8.356550e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I2_norm </td>
   <td style="text-align:right;"> 9.200000e-01 </td>
   <td style="text-align:right;"> 1.095600e+02 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 4.807317e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I3_norm </td>
   <td style="text-align:right;"> 2.581300e+02 </td>
   <td style="text-align:right;"> 1.139011e+05 </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 6.981922e+07 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I4_norm </td>
   <td style="text-align:right;"> 3.300000e-01 </td>
   <td style="text-align:right;"> 9.590000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.367000e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I5_norm </td>
   <td style="text-align:right;"> 3.100000e-01 </td>
   <td style="text-align:right;"> 9.840000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.361780e+03 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_I6_norm </td>
   <td style="text-align:right;"> 3.780000e+00 </td>
   <td style="text-align:right;"> 9.674900e+02 </td>
   <td style="text-align:right;"> 4.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 5.755581e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_M000 </td>
   <td style="text-align:right;"> 2.613920e+03 </td>
   <td style="text-align:right;"> 4.162100e+03 </td>
   <td style="text-align:right;"> 1.185000e+03 </td>
   <td style="text-align:right;"> 32.00 </td>
   <td style="text-align:right;"> 2.040670e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_CI </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 4.710000e+00 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> -153.75 </td>
   <td style="text-align:right;"> 1.470400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_E3_E1 </td>
   <td style="text-align:right;"> 2.700000e-01 </td>
   <td style="text-align:right;"> 2.100000e-01 </td>
   <td style="text-align:right;"> 2.300000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_E2_E1 </td>
   <td style="text-align:right;"> 4.400000e-01 </td>
   <td style="text-align:right;"> 2.500000e-01 </td>
   <td style="text-align:right;"> 4.400000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_E3_E2 </td>
   <td style="text-align:right;"> 5.800000e-01 </td>
   <td style="text-align:right;"> 2.300000e-01 </td>
   <td style="text-align:right;"> 5.900000e-01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_sqrt_E1 </td>
   <td style="text-align:right;"> 7.110000e+00 </td>
   <td style="text-align:right;"> 5.900000e+00 </td>
   <td style="text-align:right;"> 5.110000e+00 </td>
   <td style="text-align:right;"> 0.87 </td>
   <td style="text-align:right;"> 2.018100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_sqrt_E2 </td>
   <td style="text-align:right;"> 3.790000e+00 </td>
   <td style="text-align:right;"> 2.650000e+00 </td>
   <td style="text-align:right;"> 3.060000e+00 </td>
   <td style="text-align:right;"> 0.57 </td>
   <td style="text-align:right;"> 2.965000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_sqrt_E3 </td>
   <td style="text-align:right;"> 2.500000e+00 </td>
   <td style="text-align:right;"> 1.350000e+00 </td>
   <td style="text-align:right;"> 2.300000e+00 </td>
   <td style="text-align:right;"> 0.42 </td>
   <td style="text-align:right;"> 1.822000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_O3 </td>
   <td style="text-align:right;"> 5.898117e+05 </td>
   <td style="text-align:right;"> 2.142951e+06 </td>
   <td style="text-align:right;"> 3.142096e+04 </td>
   <td style="text-align:right;"> 9.54 </td>
   <td style="text-align:right;"> 3.232489e+08 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_O4 </td>
   <td style="text-align:right;"> 7.561160e+11 </td>
   <td style="text-align:right;"> 1.618552e+13 </td>
   <td style="text-align:right;"> 2.389454e+08 </td>
   <td style="text-align:right;"> 26.47 </td>
   <td style="text-align:right;"> 6.115114e+15 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_O5 </td>
   <td style="text-align:right;"> 5.906779e+17 </td>
   <td style="text-align:right;"> 5.133060e+19 </td>
   <td style="text-align:right;"> 4.901528e+11 </td>
   <td style="text-align:right;"> 19.58 </td>
   <td style="text-align:right;"> 2.094861e+22 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_FL </td>
   <td style="text-align:right;"> 1.765939e+15 </td>
   <td style="text-align:right;"> 3.193372e+17 </td>
   <td style="text-align:right;"> 2.240732e+09 </td>
   <td style="text-align:right;"> -23.33 </td>
   <td style="text-align:right;"> 1.536992e+20 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_O3_norm </td>
   <td style="text-align:right;"> 7.800000e-01 </td>
   <td style="text-align:right;"> 1.120000e+00 </td>
   <td style="text-align:right;"> 5.500000e-01 </td>
   <td style="text-align:right;"> 0.03 </td>
   <td style="text-align:right;"> 3.829800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_O4_norm </td>
   <td style="text-align:right;"> 1.600000e-01 </td>
   <td style="text-align:right;"> 4.400000e-01 </td>
   <td style="text-align:right;"> 7.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.155500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_O5_norm </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 1.900000e-01 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.130500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_FL_norm </td>
   <td style="text-align:right;"> 2.510000e+00 </td>
   <td style="text-align:right;"> 3.340100e+02 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.979128e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I1 </td>
   <td style="text-align:right;"> 7.687959e+08 </td>
   <td style="text-align:right;"> 2.338538e+10 </td>
   <td style="text-align:right;"> 1.806941e+06 </td>
   <td style="text-align:right;"> 26.90 </td>
   <td style="text-align:right;"> 9.365256e+12 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I2 </td>
   <td style="text-align:right;"> 6.088224e+18 </td>
   <td style="text-align:right;"> 1.457944e+21 </td>
   <td style="text-align:right;"> 4.777123e+11 </td>
   <td style="text-align:right;"> 184.28 </td>
   <td style="text-align:right;"> 6.907748e+23 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I3 </td>
   <td style="text-align:right;"> 5.462508e+20 </td>
   <td style="text-align:right;"> 1.665822e+23 </td>
   <td style="text-align:right;"> 1.252898e+12 </td>
   <td style="text-align:right;"> 162.25 </td>
   <td style="text-align:right;"> 8.766542e+25 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I4 </td>
   <td style="text-align:right;"> 1.131192e+15 </td>
   <td style="text-align:right;"> 1.880488e+17 </td>
   <td style="text-align:right;"> 1.353432e+09 </td>
   <td style="text-align:right;"> -5.77 </td>
   <td style="text-align:right;"> 9.375584e+19 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I5 </td>
   <td style="text-align:right;"> 7.080263e+14 </td>
   <td style="text-align:right;"> 1.020142e+17 </td>
   <td style="text-align:right;"> 4.826866e+08 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 5.379362e+19 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I6 </td>
   <td style="text-align:right;"> 2.018508e+16 </td>
   <td style="text-align:right;"> 3.972588e+18 </td>
   <td style="text-align:right;"> 2.538124e+10 </td>
   <td style="text-align:right;"> 88.32 </td>
   <td style="text-align:right;"> 1.523784e+21 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I1_norm </td>
   <td style="text-align:right;"> 3.410000e+00 </td>
   <td style="text-align:right;"> 4.507300e+02 </td>
   <td style="text-align:right;"> 4.700000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.583039e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I2_norm </td>
   <td style="text-align:right;"> 1.456000e+01 </td>
   <td style="text-align:right;"> 3.643240e+03 </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.154056e+06 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I3_norm </td>
   <td style="text-align:right;"> 2.182020e+05 </td>
   <td style="text-align:right;"> 1.076679e+08 </td>
   <td style="text-align:right;"> 1.000000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 6.670879e+10 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I4_norm </td>
   <td style="text-align:right;"> 2.640000e+00 </td>
   <td style="text-align:right;"> 4.366500e+02 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 2.648333e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I5_norm </td>
   <td style="text-align:right;"> 2.720000e+00 </td>
   <td style="text-align:right;"> 5.061100e+02 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 3.094470e+05 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_I6_norm </td>
   <td style="text-align:right;"> 3.696800e+02 </td>
   <td style="text-align:right;"> 1.629474e+05 </td>
   <td style="text-align:right;"> 1.200000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 9.891203e+07 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_M000 </td>
   <td style="text-align:right;"> 1.724400e+03 </td>
   <td style="text-align:right;"> 2.543160e+03 </td>
   <td style="text-align:right;"> 7.908900e+02 </td>
   <td style="text-align:right;"> 2.54 </td>
   <td style="text-align:right;"> 4.389833e+04 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_CI </td>
   <td style="text-align:right;"> 3.000000e-02 </td>
   <td style="text-align:right;"> 5.250000e+00 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> -166.26 </td>
   <td style="text-align:right;"> 1.675400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_E3_E1 </td>
   <td style="text-align:right;"> 2.700000e-01 </td>
   <td style="text-align:right;"> 2.200000e-01 </td>
   <td style="text-align:right;"> 2.300000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_E2_E1 </td>
   <td style="text-align:right;"> 4.400000e-01 </td>
   <td style="text-align:right;"> 2.500000e-01 </td>
   <td style="text-align:right;"> 4.400000e-01 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_E3_E2 </td>
   <td style="text-align:right;"> 5.800000e-01 </td>
   <td style="text-align:right;"> 2.300000e-01 </td>
   <td style="text-align:right;"> 5.900000e-01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_sqrt_E1 </td>
   <td style="text-align:right;"> 6.870000e+00 </td>
   <td style="text-align:right;"> 5.780000e+00 </td>
   <td style="text-align:right;"> 4.810000e+00 </td>
   <td style="text-align:right;"> 0.87 </td>
   <td style="text-align:right;"> 2.017100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_sqrt_E2 </td>
   <td style="text-align:right;"> 3.630000e+00 </td>
   <td style="text-align:right;"> 2.560000e+00 </td>
   <td style="text-align:right;"> 2.870000e+00 </td>
   <td style="text-align:right;"> 0.57 </td>
   <td style="text-align:right;"> 2.875000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_sqrt_E3 </td>
   <td style="text-align:right;"> 2.390000e+00 </td>
   <td style="text-align:right;"> 1.290000e+00 </td>
   <td style="text-align:right;"> 2.190000e+00 </td>
   <td style="text-align:right;"> 0.42 </td>
   <td style="text-align:right;"> 1.768000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_7_3 </td>
   <td style="text-align:right;"> 3.261000e+01 </td>
   <td style="text-align:right;"> 3.051000e+01 </td>
   <td style="text-align:right;"> 2.041000e+01 </td>
   <td style="text-align:right;"> 5.84 </td>
   <td style="text-align:right;"> 4.144200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_0_0 </td>
   <td style="text-align:right;"> 2.004000e+01 </td>
   <td style="text-align:right;"> 1.438000e+01 </td>
   <td style="text-align:right;"> 1.682000e+01 </td>
   <td style="text-align:right;"> 2.76 </td>
   <td style="text-align:right;"> 2.207200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_7_0 </td>
   <td style="text-align:right;"> 1.521000e+01 </td>
   <td style="text-align:right;"> 1.395000e+01 </td>
   <td style="text-align:right;"> 8.970000e+00 </td>
   <td style="text-align:right;"> 0.91 </td>
   <td style="text-align:right;"> 2.288400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_7_1 </td>
   <td style="text-align:right;"> 2.306000e+01 </td>
   <td style="text-align:right;"> 2.168000e+01 </td>
   <td style="text-align:right;"> 1.346000e+01 </td>
   <td style="text-align:right;"> 3.76 </td>
   <td style="text-align:right;"> 3.197400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_3_0 </td>
   <td style="text-align:right;"> 1.239000e+01 </td>
   <td style="text-align:right;"> 1.114000e+01 </td>
   <td style="text-align:right;"> 8.500000e+00 </td>
   <td style="text-align:right;"> 0.66 </td>
   <td style="text-align:right;"> 1.951600e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_5_2 </td>
   <td style="text-align:right;"> 2.730000e+01 </td>
   <td style="text-align:right;"> 2.438000e+01 </td>
   <td style="text-align:right;"> 1.902000e+01 </td>
   <td style="text-align:right;"> 3.98 </td>
   <td style="text-align:right;"> 3.446100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_6_1 </td>
   <td style="text-align:right;"> 2.438000e+01 </td>
   <td style="text-align:right;"> 2.434000e+01 </td>
   <td style="text-align:right;"> 1.569000e+01 </td>
   <td style="text-align:right;"> 0.97 </td>
   <td style="text-align:right;"> 3.223400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_3_1 </td>
   <td style="text-align:right;"> 1.930000e+01 </td>
   <td style="text-align:right;"> 1.613000e+01 </td>
   <td style="text-align:right;"> 1.458000e+01 </td>
   <td style="text-align:right;"> 2.62 </td>
   <td style="text-align:right;"> 2.619600e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_6_0 </td>
   <td style="text-align:right;"> 1.186000e+01 </td>
   <td style="text-align:right;"> 1.257000e+01 </td>
   <td style="text-align:right;"> 7.690000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 1.878500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_2_1 </td>
   <td style="text-align:right;"> 2.896000e+01 </td>
   <td style="text-align:right;"> 2.320000e+01 </td>
   <td style="text-align:right;"> 2.248000e+01 </td>
   <td style="text-align:right;"> 1.59 </td>
   <td style="text-align:right;"> 3.159100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_6_3 </td>
   <td style="text-align:right;"> 3.586000e+01 </td>
   <td style="text-align:right;"> 3.442000e+01 </td>
   <td style="text-align:right;"> 2.391000e+01 </td>
   <td style="text-align:right;"> 3.18 </td>
   <td style="text-align:right;"> 4.571400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_2_0 </td>
   <td style="text-align:right;"> 2.096000e+01 </td>
   <td style="text-align:right;"> 1.676000e+01 </td>
   <td style="text-align:right;"> 1.673000e+01 </td>
   <td style="text-align:right;"> 0.06 </td>
   <td style="text-align:right;"> 2.567500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_6_2 </td>
   <td style="text-align:right;"> 3.207000e+01 </td>
   <td style="text-align:right;"> 3.148000e+01 </td>
   <td style="text-align:right;"> 2.094000e+01 </td>
   <td style="text-align:right;"> 2.17 </td>
   <td style="text-align:right;"> 4.218400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_5_0 </td>
   <td style="text-align:right;"> 1.525000e+01 </td>
   <td style="text-align:right;"> 1.442000e+01 </td>
   <td style="text-align:right;"> 9.070000e+00 </td>
   <td style="text-align:right;"> 0.88 </td>
   <td style="text-align:right;"> 2.418300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_5_1 </td>
   <td style="text-align:right;"> 2.251000e+01 </td>
   <td style="text-align:right;"> 2.045000e+01 </td>
   <td style="text-align:right;"> 1.537000e+01 </td>
   <td style="text-align:right;"> 2.71 </td>
   <td style="text-align:right;"> 3.003900e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_4_2 </td>
   <td style="text-align:right;"> 3.304000e+01 </td>
   <td style="text-align:right;"> 3.008000e+01 </td>
   <td style="text-align:right;"> 2.360000e+01 </td>
   <td style="text-align:right;"> 2.23 </td>
   <td style="text-align:right;"> 3.798200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_1_0 </td>
   <td style="text-align:right;"> 1.650000e+00 </td>
   <td style="text-align:right;"> 4.000000e-01 </td>
   <td style="text-align:right;"> 1.630000e+00 </td>
   <td style="text-align:right;"> 0.67 </td>
   <td style="text-align:right;"> 4.980000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_4_1 </td>
   <td style="text-align:right;"> 2.800000e+01 </td>
   <td style="text-align:right;"> 2.626000e+01 </td>
   <td style="text-align:right;"> 1.975000e+01 </td>
   <td style="text-align:right;"> 0.88 </td>
   <td style="text-align:right;"> 3.450400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_7_2 </td>
   <td style="text-align:right;"> 2.887000e+01 </td>
   <td style="text-align:right;"> 2.744000e+01 </td>
   <td style="text-align:right;"> 1.746000e+01 </td>
   <td style="text-align:right;"> 4.53 </td>
   <td style="text-align:right;"> 3.775500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_shape_Z_4_0 </td>
   <td style="text-align:right;"> 1.535000e+01 </td>
   <td style="text-align:right;"> 1.518000e+01 </td>
   <td style="text-align:right;"> 1.055000e+01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 2.135400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_7_3 </td>
   <td style="text-align:right;"> 2.684000e+01 </td>
   <td style="text-align:right;"> 2.500000e+01 </td>
   <td style="text-align:right;"> 1.618000e+01 </td>
   <td style="text-align:right;"> 3.21 </td>
   <td style="text-align:right;"> 2.001800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_0_0 </td>
   <td style="text-align:right;"> 1.628000e+01 </td>
   <td style="text-align:right;"> 1.168000e+01 </td>
   <td style="text-align:right;"> 1.374000e+01 </td>
   <td style="text-align:right;"> 0.78 </td>
   <td style="text-align:right;"> 1.023700e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_7_0 </td>
   <td style="text-align:right;"> 1.410000e+01 </td>
   <td style="text-align:right;"> 1.240000e+01 </td>
   <td style="text-align:right;"> 8.520000e+00 </td>
   <td style="text-align:right;"> 1.21 </td>
   <td style="text-align:right;"> 1.229800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_7_1 </td>
   <td style="text-align:right;"> 2.008000e+01 </td>
   <td style="text-align:right;"> 1.846000e+01 </td>
   <td style="text-align:right;"> 1.143000e+01 </td>
   <td style="text-align:right;"> 1.99 </td>
   <td style="text-align:right;"> 1.516200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_3_0 </td>
   <td style="text-align:right;"> 1.039000e+01 </td>
   <td style="text-align:right;"> 9.020000e+00 </td>
   <td style="text-align:right;"> 6.870000e+00 </td>
   <td style="text-align:right;"> 0.53 </td>
   <td style="text-align:right;"> 8.338000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_5_2 </td>
   <td style="text-align:right;"> 2.231000e+01 </td>
   <td style="text-align:right;"> 1.975000e+01 </td>
   <td style="text-align:right;"> 1.507000e+01 </td>
   <td style="text-align:right;"> 2.33 </td>
   <td style="text-align:right;"> 1.784800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_6_1 </td>
   <td style="text-align:right;"> 2.051000e+01 </td>
   <td style="text-align:right;"> 2.094000e+01 </td>
   <td style="text-align:right;"> 1.312000e+01 </td>
   <td style="text-align:right;"> 0.46 </td>
   <td style="text-align:right;"> 1.621400e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_3_1 </td>
   <td style="text-align:right;"> 1.541000e+01 </td>
   <td style="text-align:right;"> 1.282000e+01 </td>
   <td style="text-align:right;"> 1.119000e+01 </td>
   <td style="text-align:right;"> 1.96 </td>
   <td style="text-align:right;"> 1.138300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_6_0 </td>
   <td style="text-align:right;"> 1.065000e+01 </td>
   <td style="text-align:right;"> 1.200000e+01 </td>
   <td style="text-align:right;"> 5.880000e+00 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.151500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_2_1 </td>
   <td style="text-align:right;"> 2.364000e+01 </td>
   <td style="text-align:right;"> 1.850000e+01 </td>
   <td style="text-align:right;"> 1.891000e+01 </td>
   <td style="text-align:right;"> 0.78 </td>
   <td style="text-align:right;"> 1.652800e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_6_3 </td>
   <td style="text-align:right;"> 2.891000e+01 </td>
   <td style="text-align:right;"> 2.831000e+01 </td>
   <td style="text-align:right;"> 1.914000e+01 </td>
   <td style="text-align:right;"> 0.98 </td>
   <td style="text-align:right;"> 2.429600e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_2_0 </td>
   <td style="text-align:right;"> 1.793000e+01 </td>
   <td style="text-align:right;"> 1.391000e+01 </td>
   <td style="text-align:right;"> 1.461000e+01 </td>
   <td style="text-align:right;"> 0.03 </td>
   <td style="text-align:right;"> 1.205600e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_6_2 </td>
   <td style="text-align:right;"> 2.624000e+01 </td>
   <td style="text-align:right;"> 2.618000e+01 </td>
   <td style="text-align:right;"> 1.718000e+01 </td>
   <td style="text-align:right;"> 0.77 </td>
   <td style="text-align:right;"> 2.156300e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_5_0 </td>
   <td style="text-align:right;"> 1.361000e+01 </td>
   <td style="text-align:right;"> 1.226000e+01 </td>
   <td style="text-align:right;"> 8.200000e+00 </td>
   <td style="text-align:right;"> 0.64 </td>
   <td style="text-align:right;"> 1.106500e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_5_1 </td>
   <td style="text-align:right;"> 1.900000e+01 </td>
   <td style="text-align:right;"> 1.687000e+01 </td>
   <td style="text-align:right;"> 1.257000e+01 </td>
   <td style="text-align:right;"> 1.68 </td>
   <td style="text-align:right;"> 1.624200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_4_2 </td>
   <td style="text-align:right;"> 2.679000e+01 </td>
   <td style="text-align:right;"> 2.426000e+01 </td>
   <td style="text-align:right;"> 1.968000e+01 </td>
   <td style="text-align:right;"> 0.78 </td>
   <td style="text-align:right;"> 2.142100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_1_0 </td>
   <td style="text-align:right;"> 1.650000e+00 </td>
   <td style="text-align:right;"> 4.000000e-01 </td>
   <td style="text-align:right;"> 1.620000e+00 </td>
   <td style="text-align:right;"> 0.61 </td>
   <td style="text-align:right;"> 4.960000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_4_1 </td>
   <td style="text-align:right;"> 2.324000e+01 </td>
   <td style="text-align:right;"> 2.147000e+01 </td>
   <td style="text-align:right;"> 1.712000e+01 </td>
   <td style="text-align:right;"> 0.40 </td>
   <td style="text-align:right;"> 1.689100e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_7_2 </td>
   <td style="text-align:right;"> 2.429000e+01 </td>
   <td style="text-align:right;"> 2.277000e+01 </td>
   <td style="text-align:right;"> 1.425000e+01 </td>
   <td style="text-align:right;"> 2.61 </td>
   <td style="text-align:right;"> 1.884000e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> part_02_density_Z_4_0 </td>
   <td style="text-align:right;"> 1.371000e+01 </td>
   <td style="text-align:right;"> 1.346000e+01 </td>
   <td style="text-align:right;"> 9.920000e+00 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 1.186200e+02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> resolution </td>
   <td style="text-align:right;"> 2.150000e+00 </td>
   <td style="text-align:right;"> 5.400000e-01 </td>
   <td style="text-align:right;"> 2.070000e+00 </td>
   <td style="text-align:right;"> 0.48 </td>
   <td style="text-align:right;"> 8.200000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FoFc_mean </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 0.000000e+00 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FoFc_std </td>
   <td style="text-align:right;"> 1.300000e-01 </td>
   <td style="text-align:right;"> 5.000000e-02 </td>
   <td style="text-align:right;"> 1.200000e-01 </td>
   <td style="text-align:right;"> 0.01 </td>
   <td style="text-align:right;"> 9.400000e-01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FoFc_square_std </td>
   <td style="text-align:right;"> 2.000000e-02 </td>
   <td style="text-align:right;"> 2.000000e-02 </td>
   <td style="text-align:right;"> 1.000000e-02 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 8.900000e-01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FoFc_min </td>
   <td style="text-align:right;"> -7.000000e-01 </td>
   <td style="text-align:right;"> 3.000000e-01 </td>
   <td style="text-align:right;"> -6.600000e-01 </td>
   <td style="text-align:right;"> -7.55 </td>
   <td style="text-align:right;"> -4.000000e-02 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FoFc_max </td>
   <td style="text-align:right;"> 2.600000e+00 </td>
   <td style="text-align:right;"> 2.540000e+00 </td>
   <td style="text-align:right;"> 1.840000e+00 </td>
   <td style="text-align:right;"> 0.04 </td>
   <td style="text-align:right;"> 4.526000e+01 </td>
   <td style="text-align:left;"> numeric </td>
  </tr>
</tbody>
</table></div>

#Ograniczenie zbioru do 50 najpopularniejszych warto�ci res_name.



# 50 najpopularniejszych wartosci res_name oraz ilo�� przyk�ad�w.

<div style="border: 1px solid #ddd; padding: 5px; overflow-y: scroll; height:400px; "><table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> res_name </th>
   <th style="text-align:right;"> Przyklady </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> SO4 </td>
   <td style="text-align:right;"> 38757 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> GOL </td>
   <td style="text-align:right;"> 27615 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> EDO </td>
   <td style="text-align:right;"> 21169 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NAG </td>
   <td style="text-align:right;"> 17941 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CL </td>
   <td style="text-align:right;"> 15627 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CA </td>
   <td style="text-align:right;"> 14377 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ZN </td>
   <td style="text-align:right;"> 13568 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MG </td>
   <td style="text-align:right;"> 9960 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> HEM </td>
   <td style="text-align:right;"> 7397 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PO4 </td>
   <td style="text-align:right;"> 7336 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ACT </td>
   <td style="text-align:right;"> 5430 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> DMS </td>
   <td style="text-align:right;"> 4601 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> IOD </td>
   <td style="text-align:right;"> 4367 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PEG </td>
   <td style="text-align:right;"> 3455 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NAD </td>
   <td style="text-align:right;"> 3278 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> K </td>
   <td style="text-align:right;"> 3220 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FAD </td>
   <td style="text-align:right;"> 3100 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MN </td>
   <td style="text-align:right;"> 2824 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CLA </td>
   <td style="text-align:right;"> 2654 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ADP </td>
   <td style="text-align:right;"> 2574 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MLY </td>
   <td style="text-align:right;"> 2413 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NAP </td>
   <td style="text-align:right;"> 2327 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CD </td>
   <td style="text-align:right;"> 2264 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> UNX </td>
   <td style="text-align:right;"> 2171 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MPD </td>
   <td style="text-align:right;"> 2165 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PG4 </td>
   <td style="text-align:right;"> 2098 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MAN </td>
   <td style="text-align:right;"> 1955 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FMT </td>
   <td style="text-align:right;"> 1951 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MES </td>
   <td style="text-align:right;"> 1852 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1PE </td>
   <td style="text-align:right;"> 1543 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ATP </td>
   <td style="text-align:right;"> 1523 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CU </td>
   <td style="text-align:right;"> 1518 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> COA </td>
   <td style="text-align:right;"> 1497 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> BR </td>
   <td style="text-align:right;"> 1454 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FMN </td>
   <td style="text-align:right;"> 1439 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> EPE </td>
   <td style="text-align:right;"> 1374 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NDP </td>
   <td style="text-align:right;"> 1312 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PGE </td>
   <td style="text-align:right;"> 1261 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> HEC </td>
   <td style="text-align:right;"> 1236 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NI </td>
   <td style="text-align:right;"> 1167 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> TRS </td>
   <td style="text-align:right;"> 1152 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NO3 </td>
   <td style="text-align:right;"> 1144 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ACY </td>
   <td style="text-align:right;"> 1138 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> SF4 </td>
   <td style="text-align:right;"> 1132 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FE </td>
   <td style="text-align:right;"> 1085 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> SAH </td>
   <td style="text-align:right;"> 1084 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PLP </td>
   <td style="text-align:right;"> 1067 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> GDP </td>
   <td style="text-align:right;"> 1062 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> UNK </td>
   <td style="text-align:right;"> 1032 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> C8E </td>
   <td style="text-align:right;"> 1020 </td>
  </tr>
</tbody>
</table></div>


# Wykresy rozkladow:

## Liczby atomow.

<!--html_preserve--><div id="htmlwidget-49c968e2f00b87e48d74" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-49c968e2f00b87e48d74">{"x":{"data":[{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[73618,2033,80822,31328,4852,7483,19214,2814,80,41,490,121,3817,1060,3090,10,98,22,83,8660,3184,111,4764,136,3099,141,22,74,67,1809],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,5,18,96,106,352,119,847,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     5<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:    18<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:    96<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:   106<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:   352<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:   119<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:   847<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: 1PE"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(248,118,109,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"1PE","legendgroup":"1PE","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[73618,2022,75403,31328,4852,7483,19214,2814,80,41,490,121,3817,1060,3090,10,98,22,83,8660,3184,111,4764,136,3099,141,22,74,67,1809],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,11,5419,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:    11<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:  5419<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACT"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(243,123,89,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"ACT","legendgroup":"ACT","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[73618,2012,74275,31328,4852,7483,19214,2814,80,41,490,121,3817,1060,3090,10,98,22,83,8660,3184,111,4764,136,3099,141,22,74,67,1809],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,10,1128,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:    10<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:  1128<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ACY"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(237,129,65,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"ACY","legendgroup":"ACY","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[73618,2012,74275,31328,4848,7483,19214,2814,75,41,490,119,1254,1060,3090,10,98,22,83,8660,3184,111,4764,136,3099,141,22,74,67,1809],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,4,0,0,0,5,0,0,2,2563,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     4<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     5<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:  2563<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ADP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(231,134,27,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"ADP","legendgroup":"ADP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[73618,2012,74275,31328,4846,7481,19198,2814,75,39,485,119,1240,1057,1611,10,98,22,83,8660,3184,111,4764,136,3099,141,22,74,67,1809],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,2,2,16,0,0,2,5,0,14,3,1479,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:    16<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     5<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:    14<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     3<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:  1479<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: ATP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(224,139,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"ATP","legendgroup":"ATP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[72164,2012,74275,31328,4846,7481,19198,2814,75,39,485,119,1240,1057,1611,10,98,22,83,8660,3184,111,4764,136,3099,141,22,74,67,1809],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[1454,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  1454<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: BR"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(216,144,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"BR","legendgroup":"BR","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[72164,2004,74238,31251,4752,7340,19138,2747,26,8,29,119,1240,1057,1611,10,98,22,83,8660,3184,111,4764,136,3099,141,22,74,67,1809],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,8,37,77,94,141,60,67,49,31,456,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     8<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:    37<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:    77<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:    94<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:   141<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:    60<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:    67<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:    49<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:    31<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:   456<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: C8E"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(207,148,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"C8E","legendgroup":"C8E","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[57787,2004,74238,31251,4752,7340,19138,2747,26,8,29,119,1240,1057,1611,10,98,22,83,8660,3184,111,4764,136,3099,141,22,74,67,1809],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[14377,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count: 14377<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CA"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(197,153,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"CA","legendgroup":"CA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[55523,2004,74238,31251,4752,7340,19138,2747,26,8,29,119,1240,1057,1611,10,98,22,83,8660,3184,111,4764,136,3099,141,22,74,67,1809],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[2264,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  2264<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CD"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(187,157,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"CD","legendgroup":"CD","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[39896,2004,74238,31251,4752,7340,19138,2747,26,8,29,119,1240,1057,1611,10,98,22,83,8660,3184,111,4764,136,3099,141,22,74,67,1809],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[15627,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count: 15627<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CL"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(175,161,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"CL","legendgroup":"CL","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[39896,2004,74238,31251,4752,7340,19138,2747,26,8,29,7,1239,1057,1611,10,76,20,82,8644,3135,4,4721,0,3047,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,0,0,0,0,0,0,112,1,0,0,0,22,2,1,16,49,107,43,136,52,141,22,74,67,1809],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:   112<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:    22<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:    16<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:    49<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:   107<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:    43<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:   136<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:    52<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:   141<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:    22<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:    74<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:    67<br />local_res_atom_non_h_count: NA<br />res_name: CLA","count:  1809<br />local_res_atom_non_h_count: NA<br />res_name: CLA"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(163,165,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"CLA","legendgroup":"CLA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[39896,2004,74238,31251,4745,7340,19138,2747,5,7,17,7,1228,1057,1578,8,74,11,75,8623,3133,1,3355,0,3047,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,7,0,0,0,21,1,12,0,11,0,33,2,2,9,7,21,2,3,1366,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     7<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:    21<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:    12<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:    11<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:    33<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     9<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     7<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:    21<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     3<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:  1366<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: COA"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(149,169,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"COA","legendgroup":"COA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[38378,2004,74238,31251,4745,7340,19138,2747,5,7,17,7,1228,1057,1578,8,74,11,75,8623,3133,1,3355,0,3047,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[1518,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  1518<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: CU"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(133,173,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"CU","legendgroup":"CU","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[38378,2001,69640,31251,4745,7340,19138,2747,5,7,17,7,1228,1057,1578,8,74,11,75,8623,3133,1,3355,0,3047,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,3,4598,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     3<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:  4598<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: DMS"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(114,176,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"DMS","legendgroup":"DMS","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[38378,1985,48487,31251,4745,7340,19138,2747,5,7,17,7,1228,1057,1578,8,74,11,75,8623,3133,1,3355,0,3047,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,16,21153,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:    16<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count: 21153<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EDO"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(91,179,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"EDO","legendgroup":"EDO","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[38378,1985,48452,31238,4738,7294,19122,1490,5,7,17,7,1228,1057,1578,8,74,11,75,8623,3133,1,3355,0,3047,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,35,13,7,46,16,1257,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:    35<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:    13<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     7<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:    46<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:    16<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:  1257<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: EPE"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(57,182,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"EPE","legendgroup":"EPE","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[38378,1985,48452,31238,4738,7294,19122,1490,5,7,17,7,1208,1057,1573,6,52,9,75,8621,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,0,0,0,0,0,0,0,20,0,5,2,22,2,0,2,0,0,0,0,3047,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:    20<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     5<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:    22<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:  3047<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FAD"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,184,31,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"FAD","legendgroup":"FAD","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[37293,1985,48452,31238,4738,7294,19122,1490,5,7,17,7,1208,1057,1573,6,52,9,75,8621,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[1085,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  1085<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: FE"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,186,66,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"FE","legendgroup":"FE","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[37293,1985,48452,31238,4738,7294,19122,1490,5,1,17,5,1208,1057,142,6,52,9,75,8621,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,0,0,0,0,6,0,2,0,0,1431,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     6<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:  1431<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMN"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,188,89,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"FMN","legendgroup":"FMN","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[37293,34,48452,31238,4738,7294,19122,1490,5,1,17,5,1208,1057,142,6,52,9,75,8621,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,1951,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:  1951<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: FMT"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,190,108,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"FMT","legendgroup":"FMT","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[37293,34,48452,31238,4738,7293,19122,1490,2,1,17,3,1208,1,142,6,52,9,75,8621,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,1,0,0,3,0,0,2,0,1056,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     3<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:  1056<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GDP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,191,125,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"GDP","legendgroup":"GDP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[37293,33,48400,3676,4738,7293,19122,1490,2,1,17,3,1208,1,142,6,52,9,75,8621,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,1,52,27562,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:    52<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count: 27562<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: GOL"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,192,141,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"GOL","legendgroup":"GOL","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[37293,33,48400,3676,4738,7293,19122,1490,2,1,17,3,1208,1,142,6,52,0,66,7403,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,9,9,1218,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     9<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     9<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:  1218<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEC"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,193,156,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"HEC","legendgroup":"HEC","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[37293,33,48400,3676,4738,7293,19122,1490,2,1,17,3,1208,1,142,5,52,0,65,8,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,1,7395,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:  7395<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: HEM"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,193,170,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"HEM","legendgroup":"HEM","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[32926,33,48400,3676,4738,7293,19122,1490,2,1,17,3,1208,1,142,5,52,0,65,8,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[4367,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  4367<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: IOD"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,192,184,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"IOD","legendgroup":"IOD","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[29706,33,48400,3676,4738,7293,19122,1490,2,1,17,3,1208,1,142,5,52,0,65,8,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[3220,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  3220<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: K"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,191,196,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"K","legendgroup":"K","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[29706,33,48400,3676,4736,5340,19122,1490,2,1,17,3,1208,1,142,5,52,0,65,8,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,2,1953,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:  1953<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MAN"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,189,208,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MAN","legendgroup":"MAN","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[29706,33,48396,3667,4731,3506,19122,1490,2,1,17,3,1208,1,142,5,52,0,65,8,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,4,9,5,1834,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     4<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     9<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     5<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:  1834<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MES"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,187,219,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MES","legendgroup":"MES","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[19746,33,48396,3667,4731,3506,19122,1490,2,1,17,3,1208,1,142,5,52,0,65,8,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[9960,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  9960<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MG"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,184,229,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MG","legendgroup":"MG","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[19746,33,48357,3641,4480,1409,19122,1490,2,1,17,3,1208,1,142,5,52,0,65,8,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,39,26,251,2097,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:    39<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:    26<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:   251<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:  2097<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MLY"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,180,239,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MLY","legendgroup":"MLY","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[16922,33,48357,3641,4480,1409,19122,1490,2,1,17,3,1208,1,142,5,52,0,65,8,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[2824,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  2824<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: MN"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,176,246,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MN","legendgroup":"MN","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[16922,33,48349,3636,2328,1409,19122,1490,2,1,17,3,1208,1,142,5,52,0,65,8,3133,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,8,5,2152,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     8<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     5<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:  2152<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: MPD"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,171,253,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MPD","legendgroup":"MPD","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[16922,33,48349,3636,2328,1402,19122,1490,2,1,10,3,1128,0,142,5,0,0,65,0,10,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,7,0,0,0,0,7,0,80,1,0,0,52,0,0,8,3123,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     7<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     7<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:    80<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:    52<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     8<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:  3123<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAD"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,165,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NAD","legendgroup":"NAD","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[16921,33,48349,3636,2327,1398,1621,1056,2,1,10,3,1128,0,142,5,0,0,65,0,10,1,3355,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[1,0,0,0,1,4,17501,434,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     1<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     4<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count: 17501<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:   434<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAG"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(82,158,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NAG","legendgroup":"NAG","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[16921,33,48349,3636,2327,1398,1621,1056,0,1,3,3,1109,0,22,3,0,0,11,0,0,1,1242,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,0,0,0,2,0,7,0,19,0,120,2,0,0,54,0,10,0,2113,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     7<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:    19<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:   120<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:    54<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:    10<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:  2113<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NAP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(121,151,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NAP","legendgroup":"NAP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[16921,33,48349,3636,2327,1398,1621,1056,0,1,0,0,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,0,0,0,0,0,3,3,27,0,22,3,0,0,11,0,0,1,1242,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     3<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     3<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:    27<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:    22<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     3<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:    11<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:  1242<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: NDP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(149,144,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NDP","legendgroup":"NDP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15754,33,48349,3636,2327,1398,1621,1056,0,1,0,0,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[1167,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  1167<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: NI"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(172,136,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NI","legendgroup":"NI","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15754,33,47205,3636,2327,1398,1621,1056,0,1,0,0,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,1144,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:  1144<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3","count:     0<br />local_res_atom_non_h_count:  4<br />res_name: NO3"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(191,128,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NO3","legendgroup":"NO3","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15754,31,47152,236,2327,1398,1621,1056,0,1,0,0,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,2,53,3400,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     2<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:    53<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:  3400<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PEG"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(207,120,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"PEG","legendgroup":"PEG","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15754,31,47118,70,2272,1164,12,1056,0,1,0,0,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,34,166,55,234,1609,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:    34<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:   166<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:    55<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:   234<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:  1609<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PG4"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(220,113,250,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"PG4","legendgroup":"PG4","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15754,31,47066,32,2265,0,12,1056,0,1,0,0,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,52,38,7,1164,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:    52<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:    38<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     7<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:  1164<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PGE"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(231,107,243,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"PGE","legendgroup":"PGE","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15754,31,47066,32,2265,0,0,1,0,1,0,0,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,0,12,1055,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:    12<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:  1055<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PLP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(240,102,234,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"PLP","legendgroup":"PLP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15752,17,39746,32,2265,0,0,1,0,1,0,0,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[2,14,7320,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     2<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:    14<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:  7320<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: PO4"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(247,99,224,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"PO4","legendgroup":"PO4","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15752,17,39746,32,2265,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,0,0,0,0,0,1,0,1,0,0,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:  1082<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SAH"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(252,97,213,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"SAH","legendgroup":"SAH","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15752,17,39745,29,1137,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,1,3,1128,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     1<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     3<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:  1128<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SF4"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(255,97,201,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"SF4","legendgroup":"SF4","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15746,0,1011,29,1137,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[6,17,38734,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     6<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:    17<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count: 38734<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: SO4"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(255,98,188,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"SO4","legendgroup":"SO4","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15746,0,1004,21,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[0,0,7,8,1137,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     7<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     8<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:  1137<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: TRS"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(255,101,174,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"TRS","legendgroup":"TRS","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[15739,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[7,0,1004,21,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     7<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:  1004<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:    21<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_count: NA<br />res_name: UNK"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(255,104,159,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"UNK","legendgroup":"UNK","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[13568,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[2171,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  2171<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: UNX"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(255,108,144,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"UNX","legendgroup":"UNX","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172413,2.20689655172413,2.20689655172414,2.20689655172416],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,2.20689655172414,4.41379310344828,6.62068965517241,8.82758620689655,11.0344827586207,13.2413793103448,15.448275862069,17.6551724137931,19.8620689655172,22.0689655172414,24.2758620689655,26.4827586206897,28.6896551724138,30.8965517241379,33.1034482758621,35.3103448275862,37.5172413793103,39.7241379310345,41.9310344827586,44.1379310344828,46.3448275862069,48.551724137931,50.7586206896552,52.9655172413793,55.1724137931034,57.3793103448276,59.5862068965517,61.7931034482759,64],"y":[13568,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count: 13568<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN","count:     0<br />local_res_atom_non_h_count:  1<br />res_name: ZN"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(252,113,127,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"ZN","legendgroup":"ZN","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":26.2283105022831,"r":7.30593607305936,"b":40.1826484018265,"l":54.7945205479452},"plot_bgcolor":"rgba(235,235,235,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-4.41379310344828,68.4137931034483],"tickmode":"array","ticktext":["0","20","40","60"],"tickvals":[0,20,40,60],"categoryorder":"array","categoryarray":["0","20","40","60"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":"local_res_atom_non_h_count","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-4042,84882],"tickmode":"array","ticktext":["0","20000","40000","60000","80000"],"tickvals":[0,20000,40000,60000,80000],"categoryorder":"array","categoryarray":["0","20000","40000","60000","80000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":"count","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"y":0.93503937007874},"annotations":[{"text":"res_name","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"cloud":false},"source":"A","attrs":{"290c6bcd5be0":{"x":{},"fill":{},"type":"bar"}},"cur_data":"290c6bcd5be0","visdat":{"290c6bcd5be0":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

## Liczby elektronow.

<!--html_preserve--><div id="htmlwidget-f474ad921ae63e0a53f4" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-f474ad921ae63e0a53f4">{"x":{"data":[{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,43243,52278,85740,8098,5344,1983,19835,2375,76,466,119,1149,1107,14,3780,1452,17,1658,105,33,8757,84,3317,22,164,4770,3108,67,1809],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,5,18,96,106,265,186,35,832,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     5<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:    18<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:    96<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:   106<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:   265<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:   186<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:    35<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:   832<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: 1PE"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(248,118,109,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"1PE","legendgroup":"1PE","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,43236,46855,85740,8098,5344,1983,19835,2375,76,466,119,1149,1107,14,3780,1452,17,1658,105,33,8757,84,3317,22,164,4770,3108,67,1809],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,7,5423,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     7<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:  5423<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACT"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(243,123,89,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"ACT","legendgroup":"ACT","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,43226,45727,85740,8098,5344,1983,19835,2375,76,466,119,1149,1107,14,3780,1452,17,1658,105,33,8757,84,3317,22,164,4770,3108,67,1809],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,10,1128,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:    10<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:  1128<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ACY"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(237,129,65,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"ACY","legendgroup":"ACY","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,43226,45727,85740,8098,5344,1979,19835,2372,76,465,118,1149,1107,12,1217,1452,17,1658,105,33,8757,84,3317,22,164,4770,3108,67,1809],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,4,0,3,0,1,1,0,0,2,2563,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     4<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     3<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:  2563<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ADP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(231,134,27,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"ADP","legendgroup":"ADP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,43226,45727,85740,8098,5343,1977,19835,2371,58,465,118,1144,1107,12,1203,1449,2,194,105,33,8757,84,3317,22,164,4770,3108,67,1809],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,1,2,0,1,18,0,0,5,0,0,14,3,15,1464,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:    18<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     5<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:    14<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     3<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:    15<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:  1464<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: ATP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(224,139,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"ATP","legendgroup":"ATP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,43226,45727,84286,8098,5343,1977,19835,2371,58,465,118,1144,1107,12,1203,1449,2,194,105,33,8757,84,3317,22,164,4770,3108,67,1809],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,1454,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:  1454<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR","count:     0<br />local_res_atom_non_h_electron_sum: 35<br />res_name: BR"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(216,144,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"BR","legendgroup":"BR","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,43218,45690,84189,7997,5242,1915,19766,2313,27,9,118,1144,1107,12,1203,1449,2,194,105,33,8757,84,3317,22,164,4770,3108,67,1809],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,8,37,97,101,101,62,69,58,31,456,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     8<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:    37<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:    97<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:   101<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:   101<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:    62<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:    69<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:    58<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:    31<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:   456<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: C8E"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(207,148,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"C8E","legendgroup":"C8E","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,28841,45690,84189,7997,5242,1915,19766,2313,27,9,118,1144,1107,12,1203,1449,2,194,105,33,8757,84,3317,22,164,4770,3108,67,1809],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,14377,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count: 14377<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA","count:     0<br />local_res_atom_non_h_electron_sum: 20<br />res_name: CA"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(197,153,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"CA","legendgroup":"CA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,28841,45690,81925,7997,5242,1915,19766,2313,27,9,118,1144,1107,12,1203,1449,2,194,105,33,8757,84,3317,22,164,4770,3108,67,1809],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,2264,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:  2264<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD","count:     0<br />local_res_atom_non_h_electron_sum: 48<br />res_name: CD"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(187,157,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"CD","legendgroup":"CD","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13214,45690,81925,7997,5242,1915,19766,2313,27,9,118,1144,1107,12,1203,1449,2,194,105,33,8757,84,3317,22,164,4770,3108,67,1809],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,15627,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count: 15627<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL","count:     0<br />local_res_atom_non_h_electron_sum: 17<br />res_name: CL"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(175,161,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"CL","legendgroup":"CL","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13214,45690,81925,7997,5242,1915,19766,2313,27,9,6,1143,1107,12,1203,1427,0,193,89,29,8650,33,3144,4,0,4737,3034,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,0,0,0,0,0,112,1,0,0,0,22,2,1,16,4,107,51,173,18,164,33,74,67,1809],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:   112<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:    22<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:    16<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:     4<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:   107<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:    51<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:   173<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:    18<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:   164<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:    33<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:    74<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:    67<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA","count:  1809<br />local_res_atom_non_h_electron_sum: NA<br />res_name: CLA"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(163,165,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"CLA","legendgroup":"CLA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13214,45690,81925,7997,5235,1915,19766,2313,6,9,3,1143,1097,12,1192,1427,0,158,87,29,8634,24,3132,2,0,3368,3034,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,7,0,0,0,21,0,3,0,10,0,11,0,0,35,2,0,16,9,12,2,0,1369,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     7<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:    21<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     3<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:    10<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:    11<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:    35<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:    16<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     9<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:    12<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:  1369<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: COA"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(149,169,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"COA","legendgroup":"COA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13214,44172,81925,7997,5235,1915,19766,2313,6,9,3,1143,1097,12,1192,1427,0,158,87,29,8634,24,3132,2,0,3368,3034,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,1518,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:  1518<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU","count:     0<br />local_res_atom_non_h_electron_sum: 29<br />res_name: CU"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(133,173,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"CU","legendgroup":"CU","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13214,44169,77327,7997,5235,1915,19766,2313,6,9,3,1143,1097,12,1192,1427,0,158,87,29,8634,24,3132,2,0,3368,3034,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,3,4598,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     3<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:  4598<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: DMS"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(114,176,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"DMS","legendgroup":"DMS","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13198,23016,77327,7997,5235,1915,19766,2313,6,9,3,1143,1097,12,1192,1427,0,158,87,29,8634,24,3132,2,0,3368,3034,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,16,21153,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:    16<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count: 21153<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EDO"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(91,179,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"EDO","legendgroup":"EDO","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13198,23016,77290,7984,5216,1883,19750,1056,6,9,3,1143,1097,12,1192,1427,0,158,87,29,8634,24,3132,2,0,3368,3034,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,37,13,19,32,16,1257,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:    37<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:    13<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:    19<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:    32<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:    16<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:  1257<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: EPE"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(57,182,0,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"EPE","legendgroup":"EPE","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13198,23016,77290,7984,5216,1883,19750,1056,6,9,3,1143,1085,12,1184,1422,0,154,65,29,8634,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,0,0,0,0,0,0,0,12,0,8,5,0,4,22,0,0,2,0,0,0,13,3034,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:    12<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     8<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     5<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     4<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:    22<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:    13<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:  3034<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FAD"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,184,31,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"FAD","legendgroup":"FAD","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13198,21931,77290,7984,5216,1883,19750,1056,6,9,3,1143,1085,12,1184,1422,0,154,65,29,8634,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,1085,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:  1085<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE","count:     0<br />local_res_atom_non_h_electron_sum: 26<br />res_name: FE"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,186,66,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"FE","legendgroup":"FE","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13198,21931,77290,7984,5216,1883,19750,1056,0,9,3,1141,1085,12,1174,1,0,154,65,29,8634,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,0,0,0,6,0,0,2,0,0,10,1421,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     6<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:    10<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:  1421<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMN"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,188,89,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"FMN","legendgroup":"FMN","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13197,19981,77290,7984,5216,1883,19750,1056,0,9,3,1141,1085,12,1174,1,0,154,65,29,8634,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,1,1950,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:  1950<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: FMT"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,190,108,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"FMT","legendgroup":"FMT","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13197,19981,77290,7984,5216,1883,19749,1056,0,6,3,1139,1085,12,118,1,0,154,65,29,8634,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,0,1,0,0,3,0,2,0,0,1056,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     3<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:  1056<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GDP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,191,125,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"GDP","legendgroup":"GDP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13196,19932,49725,7984,5216,1883,19749,1056,0,6,3,1139,1085,12,118,1,0,154,65,29,8634,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,1,49,27565,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:    49<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count: 27565<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: GOL"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,192,141,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"GOL","legendgroup":"GOL","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13196,19932,49725,7984,5216,1883,19749,1056,0,6,3,1139,1085,12,118,1,0,145,56,29,7416,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,9,9,0,1218,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     9<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     9<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:  1218<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEC"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,193,156,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"HEC","legendgroup":"HEC","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13196,19932,49725,7984,5216,1883,19749,1056,0,6,3,1139,1085,12,118,0,0,145,54,0,51,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,2,29,7365,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:    29<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:  7365<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: HEM"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,193,170,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"HEM","legendgroup":"HEM","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,13196,19932,49725,3617,5216,1883,19749,1056,0,6,3,1139,1085,12,118,0,0,145,54,0,51,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,4367,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:  4367<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD","count:     0<br />local_res_atom_non_h_electron_sum: 53<br />res_name: IOD"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,192,184,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"IOD","legendgroup":"IOD","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,9976,19932,49725,3617,5216,1883,19749,1056,0,6,3,1139,1085,12,118,0,0,145,54,0,51,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,3220,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:  3220<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K","count:     0<br />local_res_atom_non_h_electron_sum: 19<br />res_name: K"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,191,196,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"K","legendgroup":"K","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,9976,19932,49725,3615,3455,1691,19749,1056,0,6,3,1139,1085,12,118,0,0,145,54,0,51,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,2,1761,192,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:  1761<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:   192<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MAN"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,189,208,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MAN","legendgroup":"MAN","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,9976,19932,49717,3605,3455,1685,17921,1056,0,6,3,1139,1085,12,118,0,0,145,54,0,51,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,8,10,0,6,1828,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     8<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:    10<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     6<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:  1828<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MES"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,187,219,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MES","legendgroup":"MES","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,16,19932,49717,3605,3455,1685,17921,1056,0,6,3,1139,1085,12,118,0,0,145,54,0,51,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,9960,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:  9960<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG","count:     0<br />local_res_atom_non_h_electron_sum: 12<br />res_name: MG"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,184,229,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MG","legendgroup":"MG","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,16,19893,49691,3352,1378,1667,17921,1056,0,6,3,1139,1085,12,118,0,0,145,54,0,51,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,39,26,253,2077,18,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:    39<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:    26<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:   253<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:  2077<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:    18<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MLY"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,180,239,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MLY","legendgroup":"MLY","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,16,17069,49691,3352,1378,1667,17921,1056,0,6,3,1139,1085,12,118,0,0,145,54,0,51,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,2824,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:  2824<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN","count:     0<br />local_res_atom_non_h_electron_sum: 25<br />res_name: MN"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,176,246,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MN","legendgroup":"MN","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,16,17061,49686,1200,1378,1667,17921,1056,0,6,3,1139,1085,12,118,0,0,145,54,0,51,22,3132,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,8,5,2152,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     8<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     5<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:  2152<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: MPD"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,171,253,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"MPD","legendgroup":"MPD","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2173,16,17061,49686,1200,1371,1667,17921,1056,0,6,3,1132,1085,7,42,0,0,145,2,0,51,14,9,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,7,0,0,0,0,0,0,7,0,5,76,0,0,0,52,0,0,8,3123,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     7<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     7<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     5<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:    76<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:    52<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     8<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:  3123<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAD"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(0,165,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NAD","legendgroup":"NAD","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,16,17061,49686,1199,1369,1639,12,1056,0,6,3,1132,1085,7,42,0,0,145,2,0,51,14,9,2,0,3355,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[1,0,0,0,1,2,28,17909,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:    28<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count: 17909<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAG"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(82,158,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NAG","legendgroup":"NAG","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,16,17061,49686,1199,1369,1639,12,1056,0,4,0,1128,1085,3,27,0,0,25,0,0,10,1,0,1,0,1242,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,0,0,0,0,2,3,4,0,4,15,0,0,120,2,0,41,13,9,1,0,2113,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     3<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     4<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     4<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:    15<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:   120<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:    41<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:    13<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     9<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:  2113<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NAP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(121,151,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NAP","legendgroup":"NAP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,16,17061,49686,1199,1369,1639,12,1056,0,4,0,1128,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,0,0,0,0,0,0,0,3,3,27,0,0,25,0,0,10,1,0,1,0,1242,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     3<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     3<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:    27<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:    25<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:    10<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:  1242<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: NDP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(149,144,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NDP","legendgroup":"NDP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,16,15894,49686,1199,1369,1639,12,1056,0,4,0,1128,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,1167,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:  1167<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI","count:     0<br />local_res_atom_non_h_electron_sum: 28<br />res_name: NI"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(172,136,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NI","legendgroup":"NI","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,16,14750,49686,1199,1369,1639,12,1056,0,4,0,1128,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,1144,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:  1144<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3","count:     0<br />local_res_atom_non_h_electron_sum: 31<br />res_name: NO3"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(191,128,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"NO3","legendgroup":"NO3","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,14,14697,46286,1199,1369,1639,12,1056,0,4,0,1128,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,2,53,3400,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:    53<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:  3400<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PEG"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(207,120,255,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"PEG","legendgroup":"PEG","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,14,14663,46120,1144,1164,1,12,1056,0,4,0,1128,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,34,166,55,205,1638,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:    34<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:   166<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:    55<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:   205<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:  1638<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PG4"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(220,113,250,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"PG4","legendgroup":"PG4","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,14,14611,46082,1137,0,1,12,1056,0,4,0,1128,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,52,38,7,1164,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:    52<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:    38<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     7<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:  1164<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PGE"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(231,107,243,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"PGE","legendgroup":"PGE","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,14,14611,46082,1137,0,1,0,1,0,4,0,1128,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,0,12,1055,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:    12<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:  1055<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PLP"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(240,102,234,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"PLP","legendgroup":"PLP","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,12,14595,38764,1137,0,1,0,1,0,4,0,1128,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,2,16,7318,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     2<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:    16<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:  7318<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: PO4"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(247,99,224,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"PO4","legendgroup":"PO4","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,12,14595,38764,1137,0,1,0,0,0,3,0,1128,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,0,0,1,0,1,0,0,1082,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:  1082<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SAH"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(252,97,213,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"SAH","legendgroup":"SAH","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,12,14595,38764,1137,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,0,0,0,0,1,0,0,0,3,0,1128,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     3<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:  1128<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SF4"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(255,97,201,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"SF4","legendgroup":"SF4","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,6,14578,30,1137,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,6,17,38734,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     6<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:    17<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count: 38734<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: SO4"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(255,98,188,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"SO4","legendgroup":"SO4","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2172,6,14571,22,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,7,8,1137,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     7<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     8<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:  1137<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: TRS"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(255,101,174,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"TRS","legendgroup":"TRS","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[2171,0,13568,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[1,6,1003,22,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     1<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     6<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:  1003<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:    22<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK","count:     0<br />local_res_atom_non_h_electron_sum: NA<br />res_name: UNK"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(255,104,159,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"UNK","legendgroup":"UNK","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[0,0,13568,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[2171,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:  2171<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX","count:     0<br />local_res_atom_non_h_electron_sum:  6<br />res_name: UNX"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(255,108,144,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"UNX","legendgroup":"UNX","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"orientation":"v","width":[13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827587,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586,13.9310344827586],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[0,13.9310344827586,27.8620689655172,41.7931034482759,55.7241379310345,69.6551724137931,83.5862068965517,97.5172413793103,111.448275862069,125.379310344828,139.310344827586,153.241379310345,167.172413793103,181.103448275862,195.034482758621,208.965517241379,222.896551724138,236.827586206897,250.758620689655,264.689655172414,278.620689655172,292.551724137931,306.48275862069,320.413793103448,334.344827586207,348.275862068966,362.206896551724,376.137931034483,390.068965517241,404],"y":[0,0,13568,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"text":["count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count: 13568<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN","count:     0<br />local_res_atom_non_h_electron_sum: 30<br />res_name: ZN"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(252,113,127,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"ZN","legendgroup":"ZN","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":26.2283105022831,"r":7.30593607305936,"b":40.1826484018265,"l":54.7945205479452},"plot_bgcolor":"rgba(235,235,235,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-27.8620689655172,431.862068965517],"tickmode":"array","ticktext":["0","100","200","300","400"],"tickvals":[0,100,200,300,400],"categoryorder":"array","categoryarray":["0","100","200","300","400"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":"local_res_atom_non_h_electron_sum","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-4291.8,90127.8],"tickmode":"array","ticktext":["0","25000","50000","75000"],"tickvals":[0,25000,50000,75000],"categoryorder":"array","categoryarray":["0","25000","50000","75000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":"count","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"y":0.93503937007874},"annotations":[{"text":"res_name","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"cloud":false},"source":"A","attrs":{"290c3c8c1f4e":{"x":{},"fill":{},"type":"bar"}},"cur_data":"290c3c8c1f4e","visdat":{"290c3c8c1f4e":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

#  Korelacje miedzy zmiennymi.

Tabela przedstawia korelacj� pomi�dzy poszczeg�lnymi liczbowymi kolumnami zbioru przy pomocy funkcji cor(), wy�wietlaj�c wyniki gdzie warto�� bezwzgl�dna jest wi�ksza od 0,6.

##Tabela korelacji. 

<!--html_preserve--><div id="htmlwidget-e3b1a3fea7b58b984a25" style="width:100%;height:auto;" class="datatables html-widget"></div>

##Mapa ciep�a dla wybranych kolumn.

![](projekt_files/figure-html/HeatMap-1.png)<!-- -->



# 10 klas z najwieksza niezgodnoscia:

Niezgodnosc obliczona za pomoca zsumowanej ilo�ci wierszy w kt�rych wyst�puje r�nica pomi�dzy warto�ciami.

##  Liczby atom�w.

<div style="border: 1px solid #ddd; padding: 5px; overflow-y: scroll; height:400px; "><table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> res_name </th>
   <th style="text-align:right;"> Niezgodnosc </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> NAG </td>
   <td style="text-align:right;"> 17507 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MLY </td>
   <td style="text-align:right;"> 2395 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MAN </td>
   <td style="text-align:right;"> 1763 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> UNK </td>
   <td style="text-align:right;"> 1032 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PLP </td>
   <td style="text-align:right;"> 933 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CLA </td>
   <td style="text-align:right;"> 847 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1PE </td>
   <td style="text-align:right;"> 711 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> C8E </td>
   <td style="text-align:right;"> 564 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PG4 </td>
   <td style="text-align:right;"> 489 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NAP </td>
   <td style="text-align:right;"> 214 </td>
  </tr>
</tbody>
</table></div>

## Liczby elektronow.

<div style="border: 1px solid #ddd; padding: 5px; overflow-y: scroll; height:400px; "><table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> res_name </th>
   <th style="text-align:right;"> Niezgodnosc </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> NAG </td>
   <td style="text-align:right;"> 17507 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MLY </td>
   <td style="text-align:right;"> 2395 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MAN </td>
   <td style="text-align:right;"> 1763 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> UNK </td>
   <td style="text-align:right;"> 1032 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PLP </td>
   <td style="text-align:right;"> 933 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CLA </td>
   <td style="text-align:right;"> 847 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1PE </td>
   <td style="text-align:right;"> 711 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> C8E </td>
   <td style="text-align:right;"> 564 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PG4 </td>
   <td style="text-align:right;"> 489 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NAP </td>
   <td style="text-align:right;"> 214 </td>
  </tr>
</tbody>
</table></div>

#  Rozklad wartosci kolumn part_01.

![](projekt_files/figure-html/Part_01-1.png)<!-- -->![](projekt_files/figure-html/Part_01-2.png)<!-- -->![](projekt_files/figure-html/Part_01-3.png)<!-- -->![](projekt_files/figure-html/Part_01-4.png)<!-- -->![](projekt_files/figure-html/Part_01-5.png)<!-- -->![](projekt_files/figure-html/Part_01-6.png)<!-- -->![](projekt_files/figure-html/Part_01-7.png)<!-- -->![](projekt_files/figure-html/Part_01-8.png)<!-- -->![](projekt_files/figure-html/Part_01-9.png)<!-- -->![](projekt_files/figure-html/Part_01-10.png)<!-- -->![](projekt_files/figure-html/Part_01-11.png)<!-- -->![](projekt_files/figure-html/Part_01-12.png)<!-- -->![](projekt_files/figure-html/Part_01-13.png)<!-- -->![](projekt_files/figure-html/Part_01-14.png)<!-- -->![](projekt_files/figure-html/Part_01-15.png)<!-- -->![](projekt_files/figure-html/Part_01-16.png)<!-- -->![](projekt_files/figure-html/Part_01-17.png)<!-- -->![](projekt_files/figure-html/Part_01-18.png)<!-- -->![](projekt_files/figure-html/Part_01-19.png)<!-- -->![](projekt_files/figure-html/Part_01-20.png)<!-- -->![](projekt_files/figure-html/Part_01-21.png)<!-- -->![](projekt_files/figure-html/Part_01-22.png)<!-- -->![](projekt_files/figure-html/Part_01-23.png)<!-- -->![](projekt_files/figure-html/Part_01-24.png)<!-- -->![](projekt_files/figure-html/Part_01-25.png)<!-- -->![](projekt_files/figure-html/Part_01-26.png)<!-- -->![](projekt_files/figure-html/Part_01-27.png)<!-- -->![](projekt_files/figure-html/Part_01-28.png)<!-- -->![](projekt_files/figure-html/Part_01-29.png)<!-- -->![](projekt_files/figure-html/Part_01-30.png)<!-- -->![](projekt_files/figure-html/Part_01-31.png)<!-- -->![](projekt_files/figure-html/Part_01-32.png)<!-- -->![](projekt_files/figure-html/Part_01-33.png)<!-- -->![](projekt_files/figure-html/Part_01-34.png)<!-- -->![](projekt_files/figure-html/Part_01-35.png)<!-- -->![](projekt_files/figure-html/Part_01-36.png)<!-- -->![](projekt_files/figure-html/Part_01-37.png)<!-- -->![](projekt_files/figure-html/Part_01-38.png)<!-- -->![](projekt_files/figure-html/Part_01-39.png)<!-- -->![](projekt_files/figure-html/Part_01-40.png)<!-- -->![](projekt_files/figure-html/Part_01-41.png)<!-- -->![](projekt_files/figure-html/Part_01-42.png)<!-- -->![](projekt_files/figure-html/Part_01-43.png)<!-- -->![](projekt_files/figure-html/Part_01-44.png)<!-- -->![](projekt_files/figure-html/Part_01-45.png)<!-- -->![](projekt_files/figure-html/Part_01-46.png)<!-- -->![](projekt_files/figure-html/Part_01-47.png)<!-- -->![](projekt_files/figure-html/Part_01-48.png)<!-- -->![](projekt_files/figure-html/Part_01-49.png)<!-- -->![](projekt_files/figure-html/Part_01-50.png)<!-- -->![](projekt_files/figure-html/Part_01-51.png)<!-- -->![](projekt_files/figure-html/Part_01-52.png)<!-- -->![](projekt_files/figure-html/Part_01-53.png)<!-- -->![](projekt_files/figure-html/Part_01-54.png)<!-- -->![](projekt_files/figure-html/Part_01-55.png)<!-- -->![](projekt_files/figure-html/Part_01-56.png)<!-- -->![](projekt_files/figure-html/Part_01-57.png)<!-- -->![](projekt_files/figure-html/Part_01-58.png)<!-- -->![](projekt_files/figure-html/Part_01-59.png)<!-- -->![](projekt_files/figure-html/Part_01-60.png)<!-- -->![](projekt_files/figure-html/Part_01-61.png)<!-- -->![](projekt_files/figure-html/Part_01-62.png)<!-- -->![](projekt_files/figure-html/Part_01-63.png)<!-- -->![](projekt_files/figure-html/Part_01-64.png)<!-- -->![](projekt_files/figure-html/Part_01-65.png)<!-- -->![](projekt_files/figure-html/Part_01-66.png)<!-- -->![](projekt_files/figure-html/Part_01-67.png)<!-- -->![](projekt_files/figure-html/Part_01-68.png)<!-- -->![](projekt_files/figure-html/Part_01-69.png)<!-- -->![](projekt_files/figure-html/Part_01-70.png)<!-- -->![](projekt_files/figure-html/Part_01-71.png)<!-- -->![](projekt_files/figure-html/Part_01-72.png)<!-- -->![](projekt_files/figure-html/Part_01-73.png)<!-- -->![](projekt_files/figure-html/Part_01-74.png)<!-- -->![](projekt_files/figure-html/Part_01-75.png)<!-- -->![](projekt_files/figure-html/Part_01-76.png)<!-- -->![](projekt_files/figure-html/Part_01-77.png)<!-- -->![](projekt_files/figure-html/Part_01-78.png)<!-- -->![](projekt_files/figure-html/Part_01-79.png)<!-- -->![](projekt_files/figure-html/Part_01-80.png)<!-- -->![](projekt_files/figure-html/Part_01-81.png)<!-- -->![](projekt_files/figure-html/Part_01-82.png)<!-- -->![](projekt_files/figure-html/Part_01-83.png)<!-- -->![](projekt_files/figure-html/Part_01-84.png)<!-- -->![](projekt_files/figure-html/Part_01-85.png)<!-- -->![](projekt_files/figure-html/Part_01-86.png)<!-- -->![](projekt_files/figure-html/Part_01-87.png)<!-- -->![](projekt_files/figure-html/Part_01-88.png)<!-- -->![](projekt_files/figure-html/Part_01-89.png)<!-- -->![](projekt_files/figure-html/Part_01-90.png)<!-- -->![](projekt_files/figure-html/Part_01-91.png)<!-- -->![](projekt_files/figure-html/Part_01-92.png)<!-- -->![](projekt_files/figure-html/Part_01-93.png)<!-- -->![](projekt_files/figure-html/Part_01-94.png)<!-- -->![](projekt_files/figure-html/Part_01-95.png)<!-- -->![](projekt_files/figure-html/Part_01-96.png)<!-- -->

#Regresja liniowa:

##Dla liczby atom�w.


```r
set.seed(123)

reg_at<-cor_pdb%>%filter(X2=="local_res_atom_non_h_count", X1!=c("dict_atom_non_h_count", "dict_atom_non_h_electron_sum"))
reg_at_names <- reg_at[,1]

regresja_at <- pdb_clear_last%>%
  select(reg_at_names, local_res_atom_non_h_count)

idx_at <- createDataPartition(pdb_clear_last$local_res_atom_non_h_count,
                           p=0.7, list=F)
training_at <- pdb_clear_last[idx_at,]
testing_at <- pdb_clear_last[-idx_at,]

control <- trainControl(method="repeatedcv", number=2, repeats = 5)

fit_at <- train(local_res_atom_non_h_count ~ ., data=training_at, method="glm", metric="RMSE", trControl=control)
predAt<- predict(fit_at, newdata=testing_at)
postResample(predAt,testing_at$local_res_atom_non_h_count)
```

```
##       RMSE   Rsquared        MAE 
## 0.13184190 0.99990067 0.02184218
```

Warto�� RMSE zbli�ona do 0. Dla miary R^2 r�wnie� uzyskano zadowalaj�cy wynik zbli�ony do 1.

##Dla liczby elektron�w.


```r
set.seed(123)

reg_el<-cor_pdb%>%filter(X2=="local_res_atom_non_h_electron_sum", X1!=c("dict_atom_non_h_count", "dict_atom_non_h_electron_sum", "local_res_atom_non_h_electron_sum"))
reg_el_names <- reg_at[,1]

regresja_el <- pdb_clear_last%>%
  select(reg_el_names, local_res_atom_non_h_count)


idx_el <- createDataPartition(pdb_clear_last$local_res_atom_non_h_electron_sum,
                           p=0.7, list=F)
training_el <- pdb_clear_last[idx_el,]
testing_el <- pdb_clear_last[-idx_el,]

control <- trainControl(method="repeatedcv", number=2, repeats = 5)

fit_el <- train(local_res_atom_non_h_electron_sum ~ ., data=regresja_el, method="glm", metric="RMSE", trControl=control)
predEl<- predict(fit_el, newdata=testing_el)
postResample(predEl,testing_el$local_res_atom_non_h_electron_sum)
```

```
##       RMSE   Rsquared        MAE 
## 12.9018846  0.9791096  8.8296217
```

Warto�� miary RMSE niezadowalaj�ca, nie uda�o si� uzyska� dobrego wyniku dla liczby elektron�w.

#Klasyfikator Random Forest dla warto�ci res_name.

##Dla ntree = 5.

Ze wzgl�du na bardzo d�ugi czas przetwarzania danych, rozmiary zbioru zosta�y zmniejszone, a parametr ntree zosta� ograniczony do 5.


```r
pdb_clear_50_rf <- pdb_clear_50%>%select(-dict_atom_non_h_electron_sum, -dict_atom_non_h_count,-part_00_density_segments_count, -part_01_density_segments_count,-part_02_density_segments_count, -local_res_atom_non_h_electron_sum,-local_res_atom_non_h_count)


ogranicz<-createDataPartition(pdb_clear_50_rf$res_name,
                           p=0.9, list=F)


pdb_clear_50_rf<-pdb_clear_50_rf[-ogranicz,]

idx_kl <- createDataPartition(pdb_clear_50_rf$res_name,
                           p=0.7, list=F)

training_kl <- pdb_clear_50_rf[idx_kl,]

testing_kl <- pdb_clear_50_rf[-idx_kl,]

control <- trainControl(method="repeatedcv", number=2, repeats = 5)

set.seed(123)

fit_kl <- train(as.factor(res_name) ~ .,
             data = training_kl,
             method = "rf",
             trControl = control,
             ntree = 5
          )

rfRes <- predict(fit_kl, newdata = testing_kl)

cm_1<-confusionMatrix(data = rfRes, 
                factor(testing_kl[,1]))

cm_1$overall
```

```
##       Accuracy          Kappa  AccuracyLower  AccuracyUpper   AccuracyNull 
##      0.3403444      0.2878040      0.3296938      0.3511175      0.1527540 
## AccuracyPValue  McnemarPValue 
##      0.0000000            NaN
```