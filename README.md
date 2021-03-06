# demoRscripts
демонстрация прогнозирования спроса продукции для food retail на горизонт 8 дней в разрезе магазина и артикула товара

основные данные, что используются:
1. чеки с 01.01.2015 по 01.04.2020
2. остатки с 01.01.2015 по 01.04.2020
3. ассортиментая матрица (включая исторический период)
4. иерархия по магазинам

основные этапы процесса (код находиться в main.R):
loadDataFromBlobContainer() - загрузка данных с облако на локальный компьютер. при необходимости можно указать куда копировать. создаться папка: "data" куда сохраняться загруженные данные.
createOlapCube() - происходит выравнивание временного ряда и создается куб из разных таблиц. куб: shopId-skuId-date-stock-sales-salesRub - status (статус активный ассортимент или нет). выравнивание происходит в разрезе shopId-skuId, это необходимо, чтобы временной ряд не был разрывным.
preProcessTimeSeries() – когда ряд выровнены, необходимо, сделать что то с продажами когда он не продавался по причине отсутствия товара на полке. Происходит математическое восстановление спроса, при нулевых или отрицательных остатках, при нулевых продажах.
forecastOneShopIdSkuId() – расчет прогноза одного артикула на одном магазине, различными методами, такими как:
1.	fs.means – среднее значение за весь период
2.	fs.ses – экспоненциальное сглаживание
3.	fs.arima – авто АРИМА
4.	fs.croston – метод кростона
5.	fs.holt – Хольт
6.	fs.holtWinterAdditive -хольт с аддитивной сезонностью
7.	fs.holtWinterMult – хольт с мультипликативной сезонностью
8.	fs.naive – наивный метод
9.	fs.snaive – сезонный наивный метод
10.	fs.nnet – нейронная сеть
11.	fs.stlEts – экспоненциальное сглаживание с выделением тренда и сезонности
12.	fs.tbst – метод Tbats
не все методы могут быть рассчитаны, поэтому предусмотрено в функции отваливание методов, без ущерба общему расчету системы.
результаты расчётов сохраняются в файл csv forecastDT в корень проекта, в виде:
1.	shopId – айди магазина
2.	skuId – айди артикула
3.	method – метод прогнозирования
4.	V1…V8 -прогноз на 8 дней вперед

Все функции вынесены в файл function.R
Основные библиотеки:
library(AzureRMR)
library(AzureStor)
library(tidyverse)
library(data.table)
library(lubridate)
library(arrow)
library(forecast)
library(doParallel)
протестировано на:
R version 3.6.3 (2020-02-29) -- "Holding the Windsock"
Copyright (C) 2020 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)

"Machine:     " "AMD EPYC 7742 64-Core Processor"
"Num cores:   " "20 cores"                       
"Num threads: " "20 threads"                     
"RAM:         " "157 Gb"

время выполнения: 3.9 часа
