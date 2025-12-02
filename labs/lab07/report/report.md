---
## Front matter
title: "Отчёт по лабораторной работе 7"
subtitle: "Адресация IPv4 и IPv6. Настройка DHCP"
author: "Сейдалиев Тагиетдин Ровшенович"

## Generic otions
lang: ru-RU
toc-title: "Содержание"

## Bibliography
bibliography: bib/cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl

## Pdf output format
toc: true # Table of contents
toc-depth: 2
lof: true # List of figures
lot: true # List of tables
fontsize: 12pt
linestretch: 1.5
papersize: a
documentclass: scrreprt
## I18n polyglossia
polyglossia-lang:
  name: russian
  options:
	- spelling=modern
	- babelshorthands=true
polyglossia-otherlangs:
  name: english
## I18n babel
babel-lang: russian
babel-otherlangs: english
## Fonts
mainfont: IBM Plex Serif
romanfont: IBM Plex Serif
sansfont: IBM Plex Sans
monofont: IBM Plex Mono
mathfont: STIX Two Math
mainfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
romanfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
sansfontoptions: Ligatures=Common,Ligatures=TeX,Scale=MatchLowercase,Scale=0.94
monofontoptions: Scale=MatchLowercase,Scale=0.94,FakeStretch=0.9
mathfontoptions:
## Biblatex
biblatex: true
biblio-style: "gost-numeric"
biblatexoptions:
  - parentracker=true
  - backend=biber
  - hyperref=auto
  - language=auto
  - autolang=other*
  - citestyle=gost-numeric
## Pandoc-crossref LaTeX customization
figureTitle: "Рис."
tableTitle: "Таблица"
listingTitle: "Листинг"
lofTitle: "Список иллюстраций"
lotTitle: "Список таблиц"
lolTitle: "Листинги"
## Misc options
indent: true
header-includes:
  - \usepackage{indentfirst}
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Цель работы

Получение навыков настройки службы DHCP на сетевом оборудовании для распределения адресов IPv4 и IPv6.

# Выполнение

## Настройка DHCP-сервера на маршрутизаторе VyOS и получение адреса клиентом VPCS

В рабочем пространстве GNS3 был развёрнут новый проект и размещены устройства согласно заданной топологии. Каждому элементу были присвоены имена: PC1-trseidaliev, trseidaliev-sw-01, trseidaliev-gw-01. Итоговая схема представлена ниже:

![Топология лабораторного стенда](Screenshot_1.png){ #fig:001 width=80% }

После запуска маршрутизатора VyOS выполнен вход под учётной записью по умолчанию. Далее произведена установка системы, настройка имени хоста, доменного имени и создание нового пользователя. 

![Настройка имени хоста, домена и пользователя](Screenshot_2.png){ #fig:002 width=75% }

Была удалена учётная запись vyos, отключён DHCP-клиент на интерфейсе eth0 и выполнена подготовка интерфейса для статической адресации.

![Удаление пользователя vyos и подготовка интерфейса](Screenshot_3.png){ #fig:003 width=75% }

На интерфейсе eth0 установлен адрес 10.0.0.1/24. Затем на маршрутизаторе настроен DHCP-сервер: создана сеть trseidaliev, указана подсеть 10.0.0.0/24, настроен DNS-сервер и диапазон выдаваемых адресов 10.0.0.2–10.0.0.253.

На клиенте PC1-trseidaliev был выполнен запрос DHCP с опцией -d. Клиент успешно получил параметры конфигурации:  
IP-адрес — 10.0.0.2/24  
Шлюз — 10.0.0.1  
DNS — 10.0.0.1  
Домен — trseidaliev.net  
DHCP-сервер — 10.0.0.1

![Результат DHCP на PC1](Screenshot_4.png){ #fig:004 width=75% }

Параметры подключения подтверждаются выводом команды show ip. Проверка связности с маршрутизатором выполнена с помощью ping.

![Информация об адресе и проверка доступности маршрутизатора](Screenshot_5.png){ #fig:005 width=75% }

На маршрутизаторе была просмотрена статистика DHCP-сервера и список активных аренд. Зафиксирована одна активная аренда — адрес 10.0.0.2, выданный клиенту VPCS.

![Статистика DHCP и список аренды](Screenshot_6.png){ #fig:006 width=70% }

Журнал DHCP отображает последовательность сообщений DHCPDISCOVER, DHCPOFFER, DHCPREQUEST и DHCPACK.

![Журнал DHCP на маршрутизаторе](Screenshot_7.png){ #fig:007 width=80% }

### Анализ DHCP-трафика

На анализаторе трафика захвачена полная последовательность DHCP-обмена. Пакет DHCP Request содержит MAC-адрес клиента, идентификатор транзакции, запрашиваемый IP-адрес (10.0.0.2), идентификатор сервера (10.0.0.1) и перечень параметров (маска, шлюз, DNS, доменное имя).

![Анализ пакета DHCP в Wireshark](Screenshot_8.png){ #fig:008 width=100% }

## Настройка IPv6 и DHCPv6 (Stateless) в расширенной топологии

В рабочем пространстве был дополнен ранее созданный проект. Добавлены два новых коммутатора и хост PC3 с образом Kali Linux CLI. Им присвоены имена: trseidaliev-sw-02, trseidaliev-sw-03, PC3-trseidaliev. Схема приведена ниже:

![Топология лабораторного стенда](Screenshot_9.png){ #fig:009 width=80% }

На маршрутизаторе trseidaliev-gw-01 выполнена настройка IPv6-адресов на интерфейсах eth1 и eth2. Интерфейсы получили адреса 2000::1/64 и 2001::1/64. Команда show interfaces подтверждает корректное назначение адресов.

![Назначение IPv6-адресов на интерфейсы](Screenshot_10.png){ #fig:010 width=75% }

После настройки адресации была включена служба Router Advertisement для интерфейса eth1. Задан префикс 2000::/64 и активирован флаг other-config-flag, указывающий, что дополнительные параметры должны быть получены по DHCPv6.

Затем был настроен DHCPv6-сервер без отслеживания состояния. Создана разделяемая сеть trseidaliev-stateless, добавлены общие параметры: DNS-сервер 2000::1 и доменное имя trseidaliev.net.

![Конфигурация RA и DHCPv6 Stateless](Screenshot_11.png){ #fig:011 width=80% }

Команда show configuration отображает созданный DHCPv6-сервер и конфигурацию RA.

![Просмотр конфигурации маршрутизатора](Screenshot_12.png){ #fig:012 width=70% }

На клиенте PC2-trseidaliev (Kali Linux CLI) проверены сетевые параметры. Интерфейс eth0 получил адрес из префикса 2000::/64 по SLAAC. Маршруты IPv6 сформированы автоматически. Пинг маршрутизатора 2000::1 прошёл успешно.

![Настройки IPv6 на PC2 и проверка связности](Screenshot_13.png){ #fig:013 width=80% }

Содержимое resolv.conf показывает, что DNS пока не назначен, так как DHCPv6 Stateless ещё не выполнен.

Далее выполнена команда dhclient -6 -S -v eth0 для запроса параметров DHCPv6. Клиент получил DNS-сервер 2000::1 и доменное имя trseidaliev.net. После этого повторная проверка связности с маршрутизатором также прошла успешно.

![Запрос DHCPv6 и обновлённый DNS](Screenshot_14.png){ #fig:014 width=80% }

На маршрутизаторе просмотрены активные записи DHCPv6. В данном режиме (stateless) сервер не назначает адреса, поэтому список выданных адресов пуст, что является нормальным результатом.

![Просмотр аренд DHCPv6](Screenshot_15.png){ #fig:015 width=70% }

### Анализ DHCPv6-трафика

При анализе трафика в захвате видны сообщения Information-Request и Reply. Клиент отправляет запрос без запроса адреса, только параметры конфигурации. В ответе сервер передаёт поле DNS Recursive Name Server (2000::1) и Domain Search List (trseidaliev.net).

![Анализ пакета DHCPv6 в Wireshark](Screenshot_16.png){ #fig:016 width=100% }

## Настройка DHCPv6 Stateful на интерфейсе eth2 и получение IPv6-адреса узлом PC3

На маршрутизаторе trseidaliev-gw-01 была выполнена настройка DHCPv6 с отслеживанием состояния. На интерфейсе eth2 активирован флаг managed-flag, указывающий, что конфигурация адресации должна выполняться через DHCPv6 Stateful. Далее создана разделяемая сеть trseidaliev-stateful и настроена подсеть 2001::/64 с диапазоном выдаваемых адресов от 2001::100 до 2001::199. Указаны параметры DNS и доменного имени.

![Настройка DHCPv6 Stateful и RA](Screenshot_17.png){ #fig:017 width=80% }

После сохранения конфигурации на маршрутизаторе был выполнен просмотр активных аренд DHCPv6. На данном этапе сервер не имеет выданных адресов, что соответствует отсутствию запросов со стороны клиентов.

На узле PC3-trseidaliev (Kali Linux CLI) были проверены текущие параметры IPv6. Интерфейс eth0 получил только SLAAC-адрес, а маршрут по умолчанию и DNS отсутствуют, что подтверждается выводами ifconfig, route и resolv.conf.

![Параметры IPv6 на PC3 до получения адреса](Screenshot_18.png){ #fig:018 width=80% }

Далее выполнена команда dhclient -6 -v eth0. Клиент успешно получил параметры DHCPv6: назначенный адрес из диапазона (2001::198 или 2001::199), DNS-сервер 2001::1, время аренды и дополнительные служебные значения IAID и T1/T2. Клиент корректно обработал DHCPv6 Advertise и Reply.

![Запрос DHCPv6 и получение адреса](Screenshot_19.png){ #fig:019 width=80% }

После получения адреса через DHCPv6 параметры сети на PC3 обновились. Интерфейс eth0 теперь содержит адрес из диапазона DHCPv6, маршруты IPv6 включают маршрут по умолчанию fe80::/10, а DNS-сервер и доменное имя trseidaliev.net прописаны в resolv.conf. Пинг маршрутизатора 2001::1 проходит успешно.

![Проверка IPv6 после получения DHCPv6 Stateful адреса](Screenshot_20.png){ #fig:020 width=80% }

На маршрутизаторе просмотрены активные аренды DHCPv6. Сервер показывает две активные записи: одна из диапазона 2001::198 и одна из 2001::199, каждая со своим DUID клиента и параметрами срока аренды.

![Просмотр аренд DHCPv6 Stateful](Screenshot_21.png){ #fig:021 width=80% }

### Анализ DHCPv6-трафика

В захваченном трафике видны сообщения Solicit, Advertise, Request и Reply, характерные для DHCPv6 Stateful. Клиент в запросе Request передаёт свой DUID, идентификатор IA_NA и параметры, которые он хочет получить. Сервер в ответе указывает назначенный адрес, время аренды и параметры DNS.

![Анализ пакета DHCPv6 Stateful в Wireshark](Screenshot_22.png){ #fig:022 width=100% }

# Заключение

В ходе выполнения работы:

- была построена расширенная топология сети с использованием нескольких коммутаторов, маршрутизатора VyOS и узлов на базе VPCS и Kali Linux;
- выполнена настройка IPv6-адресации на маршрутизаторе, включая распределение префиксов по интерфейсам eth1 и eth2;
- реализована конфигурация DHCPv6 в двух режимах: Stateless и Stateful, с настройкой Router Advertisements и параметров common-options;
- проверено автоматическое получение IPv6-параметров клиентами: SLAAC, DNS по DHCPv6 Stateless, а также выделение адресов из заданного диапазона по DHCPv6 Stateful;
- подтверждена корректная работа сетевой конфигурации с помощью команд ifconfig, route и ping на узлах PC2 и PC3;
- изучены и проанализированы DHCPv6-пакеты в Wireshark, включая сообщения Solicit, Advertise, Request, Reply, а также параметры IA_NA, DUID и T1/T2;
- проверена корректность выдачи арендуемых адресов и состояния сервера DHCPv6 через команды show dhcpv6 server leases на маршрутизаторе.

Работа позволила отработать навыки настройки IPv6, механизмов SLAAC, DHCPv6 Stateless и Stateful, а также анализ сетевого трафика и взаимодействие узлов в сложной виртуальной топологии.
