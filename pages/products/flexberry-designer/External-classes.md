---
title: Внешние классы (классы со стереотипом external) 
sidebar: product--sidebar
keywords: Flexberry Designer, Flexberry ORM, Public
toc: true
permalink: ru/external-classes.html
folder: product--folder
lang: ru
---

[Стереотип](key-concepts-flexberry-designer.html) `external` позволяет объявить в `CASE` класс, соответствующий любому внешнему (не объявленному в `CASE`, языковому) классу. Это удобно и необходимо в тех случаях, когда необходимо оперировать типом, который в модели не объявлен, но объявлен где-то на уровне кода (для случая `.Net`-языка, - в исходных или подключенных к исходным сборках).


Пример внешнего класса:
![](/images/pages/img/Flexberry plugins/external.jpg)

# Что генерируется с описания внешнего класса
{| border="1"
! Что генерируется
! Генерация в SQL DDL
! Генерация в .Net-язык
|-
| Любая часть любого UML-класса (атрибут, параметр метода и т.д.), объявленная этим типом
| Как есть или указывается преобразование в карте типов для генератора.
| Как есть. Атрибут, параметр метода и т.п. объявляются этим типом
|}

# Ссылки на другие стадии
Так же через external-класс можно поставить ссылку на другую стадию. В этом случае будет генерироваться всё множество классов из этой и из той стадии. SQL будет аналогично сгенерирован для классов обоих стадий.

# Особенности генерации
Если в исходной стадии и external стадии есть классы с одинаковым именем, то приоритет отдается external классу. При генерации БД все атрибуты из external класса сохраняются и к ним добавляются атрибуты из исходного класса. Если атрибут уже есть, то он не будет добавлен. <br />
Приоритет отдается external классу, т.к. чаще всего это общеприкладные компоненты, например, "Аудит", "Полномочия" и изменять их нельзя.
<br />
<br />
__Например__: Во внешней стадии есть класс с названием `Класс1` и атрибутами `А1` и `А2:string`, а в исходной стадии есть класс с таким же названием и атрибутами `А2:int` и `А3`. Тогда, при приведении базы данных в соответствие, будет сгенерирован класс `Класс1` с атрибутами `А1`, `А2:string` и `А3`. 
