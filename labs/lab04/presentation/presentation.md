---
## Front matter
lang: ru-RU
title: "Лабораторная работа №4"
subtitle: "Задание для самостоятельного выполнения"
author: 
  - Астраханцева А. А.
institute:
  - Российский университет дружбы народов, Москва, Россия
date: 28 февраля 2025

## i18n babel
babel-lang: russian
babel-otherlangs: english

## Formatting pdf
toc: false
toc-title: Содержание
slide_level: 2
aspectratio: 169
section-titles: true
theme: metropolis
header-includes:
 - \metroset{progressbar=frametitle,sectionpage=progressbar,numbering=fraction}
---

# Информация

## Докладчик

:::::::::::::: {.columns align=center}
::: {.column width="70%"}

  * Астраханцева Анастасия Александровна
  * НФИбд-01-22, 1132226437
  * Российский университет дружбы народов
  * [1132226437@pfur.ru](mailto:1132226437@pfur.ru)
  * <https://github.com/aaastrakhantseva>

:::
::: {.column width="30%"}

![](./image/nastya.jpg)

:::
::::::::::::::

# Вводная часть

## Цели лабораторной работы

Закрепление навыков моделирования сетей с помощью средства имитационного моделирования NS-2, а также построения графиков с помощью GNUplot. 

## Задачи

1. Для приведённой схемы разработать имитационную модель в пакете NS-2.
2. Построить график изменения размера окна TCP (в Xgraph и в GNUPlot);
3. Построить график изменения длины очереди и средней длины очереди на первом
маршрутизаторе.
4. Оформить отчёт о выполненной работе

# Выполнение ЛР

## Описание моделируемой сети

- сеть состоит из N TCP-источников, N TCP-приёмников, двух маршрутизаторов R1 и R2 между источниками и приёмниками (N — не менее 20);
- между TCP-источниками и первым маршрутизатором установлены дуплексные соединения с пропускной способностью 100 Мбит/с и задержкой 20 мс очередью типа DropTail;
- между TCP-приёмниками и вторым маршрутизатором установлены дуплексные соединения с пропускной способностью 100 Мбит/с и задержкой 20 мс очередью типа DropTail;

## Описание моделируемой сети

- между маршрутизаторами установлено симплексное соединение (R1–R2) с пропускной способностью 20 Мбит/с и задержкой 15 мс очередью типа RED, размером буфера 300 пакетов; в обратную сторону — симплексное соединенние (R2–R1) с пропускной способностью 15 Мбит/с и задержкой 20 мс очередью типа DropTail;
- данные передаются по протоколу FTP поверх TCPReno;
- параметры алгоритма RED: qmin = 75, qmax = 150, qw = 0, 002, pmax = 0.1;
- максимальный размер TCP-окна 32; размер передаваемого пакета 500 байт; время моделирования — не менее 20 единиц модельного времени.

## Описание скрипта

```
# создание объекта Simulator
set ns [new Simulator]

# открытие на запись файла out.nam для визуализатора nam
set nf [open out.nam w]

# все результаты моделирования будут записаны в переменную nf
$ns namtrace-all $nf

# открытие на запись файла трассировки out.tr
# для регистрации всех событий
set f [open out.tr w]
# все регистрируемые события будут записаны в переменную f
$ns trace-all $f
```

## Описание скрипта

```
# для регистрации всех событий
set f [open out.tr w]
# все регистрируемые события будут записаны в переменную f
$ns trace-all $f
Agent/TCP set window_ 32
Agent/TCP set pktSize_ 500

```


## Описание скрипта

```
# процедура finish
proc finish {} {
	global tchan_
	# подключение кода AWK:
	set awkCode {
	{
		if ($1 == "Q" && NF>2) {
			print $2, $3 >> "temp.q";
			set end $2
	}
		else if ($1 == "a" && NF>2)
			print $2, $3 >> "temp.a";
	}
}
```

## Описание скрипта

```
exec rm -f temp.q temp.a
exec touch temp.a temp.q

#set f [open temp.q w]
#close $f

#set f [open temp.a w]
#close $f

exec awk $awkCode all.q
```

## Описание скрипта

```
# Запуск xgraph с графиками окна TCP и очереди:
# Изменение размера окна на одном источнике 
exec xgraph -bb -tk -x time -t "TCPRenoCWND" WindowSizeOne &
# Изменение размера окна на всех источниках 
exec xgraph -bb -tk -x time -t "TCPRenoCWND" WindowSizeAll &
# Изменение длины очереди на первом марштуртизаторе источниках 
exec xgraph -bb -tk -x time -y queue temp.q &
# Изменение средней длины очереди на первом марштуртизаторе источниках 
exec xgraph -bb -tk -x time -y queue temp.a &
exec nam out.nam &
exit 0
}
```

## Описание скрипта

```
# Формирование файла с данными о размере окна TCP:
proc plotWindow {tcpSource file} {
	global ns
	set time 0.01
	set now [$ns now]
	set cwnd [$tcpSource set cwnd_]
	puts $file "$now $cwnd"
	$ns at [expr $now+$time] "plotWindow $tcpSource $file"
}
set r1 [$ns node]
set r2 [$ns node]

```

## Описание скрипта

```
$ns simplex-link $r1 $r2 20Mb 15ms RED
$ns simplex-link $r2 $r1 15Mb 20ms DropTail
$ns queue-limit $r1 $r2 300

set N 25
for {set i 0} {$i < $N} {incr i} {
	set n1($i) [$ns node]
	$ns duplex-link $n1($i) $r1 100Mb 20ms DropTail
	set n2($i) [$ns node]
	$ns duplex-link $n2($i) $r2 100Mb 20ms DropTail

	set tcp($i) [$ns create-connection TCP/Reno $n1($i) TCPSink $n2($i) $i]
	set ftp($i) [$tcp($i) attach-source FTP]
}


```

## Описание скрипта

```
# Мониторинг размера окна TCP:
set windowSizeOne [open WindowSizeOne w]
set windowSizeAll [open WindowSizeAll w]

set qmon [$ns monitor-queue $r1 $r2 [open qm.out w] 0.1];
[$ns link $r1 $r2] queue-sample-timeout;

# Мониторинг очереди:
set redq [[$ns link $r1 $r2] queue]
$redq set thresh_ 75
$redq set maxthresh_ 150
$redq set q_weight_ 0.002
$redq set linterm_ 10

```

## Описание скрипта

```
set tchan_ [open all.q w]
$redq trace curq_
$redq trace ave_
$redq attach $tchan_

for {set i 0} {$i < $N} {incr i} {
	$ns at 0.0 "$ftp($i) start"
	$ns at 0.0 "plotWindow $tcp($i) $windowSizeAll"
}

$ns at 0.0 "plotWindow $tcp(1) $windowSizeOne"

# at-событие для планировщика событий, которое запускает
$ns at 25.0 "finish"
# запуск модели
$ns run

```

## Модель сети в аниматоре nam

![Модель описываемой сети](image/1.jpg){#fig:001 width=90%}

## Графики изменения размера окна

![Графики изменения размера окна](image/2.jpg){#fig:002 width=90%}

## Графики изменения размера длины очереди на линке (R1–R2) 

![Графики изменения размера длины очереди](image/3.jpg){#fig:003 width=90%}


## Скрипт для построения графиков в GNUplot

```
#!/usr/bin/gnuplot -persist
# задаём текстовую кодировку,
# тип терминала, тип и размер шрифта

set encoding utf8
set term pngcairo font "Helvetica,9"
```

## Скрипт для построения графиков в GNUplot

```
# задаём выходной файл графика
set out 'window_one.pdf'

# задаём название графика
set title "Изменение размера окна TCP на линке 1-го источника при N=25"

# подписи осей графика
set xlabel "t[s]" font "Helvetica, 10"
set ylabel "CWND [pkt]" font "Helvetica, 10"

# построение графика, используя значения
# 1-го и 2-го столбцов файла WindowSizeOne
plot "WindowSizeOne" using ($1):($2) with lines title "Размер окна TCP"
```

## Скрипт для построения графиков в GNUplot

```
# задаём выходной файл графика
set out 'window_all.pdf'

# задаём название графика
set title "Изменение размера окна TCP на всех источниках при N=25"

# построение графика, используя значения
# 1-го и 2-го столбцов файла WindowSizeAll
plot "WindowSizeAll" using ($1):($2) with lines title "Размер окна TCP"
```

## Скрипт для построения графиков в GNUplot

```
# задаём выходной файл графика
set out 'queue.pdf'

# задаём название графика
set title "Изменение размера длины очереди на линке (R1–R2)"

# подписи осей графика
set xlabel "t[s]" font "Helvetica, 10"
set ylabel "Queue Length [pkt]" font "Helvetica, 10"

# построение графика, используя значения
# 1-го и 2-го столбцов файла temp.q
plot "temp.q" using ($1):($2) with lines title "Текущая длина очереди"
```

## Скрипт для построения графиков в GNUplot

```
# задаём выходной файл графика
set out 'av_queue.pdf'

# задаём название графика
set title "Изменение размера средней длины очереди на линке (R1–R2)"

# подписи осей графика
set xlabel "t[s]" font "Helvetica, 10"
set ylabel "Queue Avg Length [pkt]" font "Helvetica, 10"

# построение графика, используя значения
# 1-го и 2-го столбцов файла temp.a
plot "temp.a" using ($1):($2) with lines title "Средняя длина очереди"

```

## Графики изменения размера окна в GNUplot

![Графики изменения размера окна в GNUplot](image/4.jpg){#fig:004 width=90%}

## Графики изменения размера длины очереди на линке (R1–R2) в GNUplot

![Графики изменения размера длины очереди в GNUplot](image/5.jpg){#fig:005 width=90%}



## Выводы

В ходе выполнения лабораторной работы я закрепила навыки моделирования сетей с помощью средства имитационного моделирования NS-2, а также построения графиков с помощью GNUplot. 


# Спасибо за внимание!
