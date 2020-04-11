---
layout: post
title: Структуры в C#
description: Структуры, свойства структур, ограничения структур, структуры и классы, мифы о структурах.
summary: Структуры, свойства структур, ограничения структур, структуры и классы, мифы о структурах.
comments: true
tags: [csharp]
---

# Что такое структуры в С#?
Структуры - значимый тип, обычно используется для объединения небольшой группы связанных переменных. Структуры подходят для описания легковесных объектов, как например: `Point`, `Rectangle` и `Color`. Когда создается объект структуры и назначается на переменную, то эта переменная хранит полностью значение этой структуры. Когда переменная, содержащая в себе структуру копируется, то копируются все данные и любые изменения в копии не влияют на данные, с которых была сделана копия. Поскольку структуры не используют ссылки, они не имеют идентификаторов. Вы не можете отличить друг от друга 2 сущности значимого типа с одинаковыми данными.
 
Структуры могут содержать конструкторы, константы, поля, методы, свойства, индексаторы, операторы, события и вложенные типы. Однако, если у вас есть потребность в нескольких из них, лучше использовать классы.
 
## Структуры обладают следующими свойствами:
- Структуры являются значимыми типами.
- В отличие от классов, структуры могут быть инстанциированы без использования оператора new.
- В структурах могут быть конструкторы, но только с параметрами.
- Структуры не могут наследоваться от других структур или классов и от структур тоже запрещено наследование. Все структуры уже являются прямыми наследниками System.ValueType.
- Структуры могут реализовывать интерфейсы.
- Структуры можно использовать как nullable типы `Nullable<MyStruct>`, тогда такой переменной можно присвоить `null`.
 
## Какими ограничениями обладают структуры по сравнению с классами?
- Нельзя инициализировать поля структур при объявлении структуры, т.к. они по сути являются статическими или константами.
- В структурах нельзя объявлять конструктор по-умолчанию (без параметров), потому что в нем нет необходимости, т.к. копии структур создаются и уничтожаются компилятором автоматически. Компилятор сам как-бы имплементирует конструктор по-умолчанию, который присваивает дефолтные значения.
- Члены структуры не могут быть объявлены как `protected`, т.к. в этом нет смысла, потому что структуры не могут наследоваться.
- Структуры не могут быть объявлены как `static`
 
## В каких случаях отдать предпочтение структуре, а не классу?
- Она логически представляет собой значение, похожее на примитивный тип (integer, DateTime, Color и т.д.)
- Размер экземпляра структуры не превышает 16 байт.
- Она immutable.
- Она не должны будет упаковываться (часто или совсем).
- Предполагается кратковременный цикл жизни экземпляров структуры.

Итоговое решение зависит от ситуации, но следует понимать, что при нарушении одного или более условий возможно негативное влияние на производительность.


# Мифы и заблуждения, связанные со структурами

## Миф 1. Типы значений обеспечивают большую производительность.
В некоторых случаях это верно: для них не нужна сборка мусора, когда они не упакованы, они не сопряжены с накладными расходами по идентификации типов и не требуют разыменования.
В других случаях более производительными являются ссылочные типы:
- передача параметров,
- присваивание значений переменным,
- возвращение значений.

Эти операции не требуют копирования всех данных.

## Миф 2. Ссылочные типы находятся в куче, а типы значений - в стеке.
Ссылочные типы действительно всегда находятся в куче.
В стеке же размечаются только локальные переменные (объявленные внутри метода) и параметры метода.