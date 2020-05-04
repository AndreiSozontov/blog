---
layout: post
title: Exceptions - гайдлайн по созданию
description: Как правильно определить класс-исключение
summary: Как правильно определить класс-исключение
comments: true
tags: [csharp, best_practices]
---

В данном посте я буду ссылаться на:
1. Гайдлайны Microsoft <https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/exceptions/creating-and-throwing-exceptions#defining-exception-classes>
2. Исходники на referencesource <https://referencesource.microsoft.com/#mscorlib/system/exception.cs,f092fb2b893a0162>

Давайте сразу на примере, и сразу на хорошем (плохой пример приводить не будем). Допустим, у вас есть какой-то сервис, назовем его PaymentService. Вам нужен базовый класс (а может и единственный, т.к. производные от него классы и не понадобятся) для исключений, которые случаются по вине пользователей, а не по вине ошибок в коде сервиса. И еще, допустим, вы хотите добавить одно или несколько кастомных свойств в этот класс.

Тогда вам нужно примерно следующее:

```csharp
[Serializable]
public class PaymentServiceException : Exception
{
    // Кастомное св-во для демонстрации.
    public int PaymentId { get; set; }

    public override string Message => base.Message + $" PaymentId={PaymentId}.";

    public PaymentServiceException() : base() {}

    public PaymentServiceException(string message) : base(message) {}

    public PaymentServiceException(string message, int paymentId) : base(message)
    {
        PaymentId = paymentId;
    }

    public PaymentServiceException(string message, Exception innerException) : base(message, innerException) {}

    // Конструктор, который нужен для корректной сериализации, если ваш сервис передает исключение за свои пределы.
    public PaymentServiceException(SerializationInfo info, StreamingContext context) : base(info, context) {}
}
```

Все просто и понятно, но встает 2 вопроса:
1. Зачем нужен `SerializationAttribute`?
2. Что там за тема с переопределением `ToString()`, о котором говорят гайдлайны Microsoft, если есть кастомные свойства?

**Поддержка сериализации**

Я впервые столкнулся с этим и узнал про это, когда смотрел отчет SonarQube о сервисе, над которым трудилась моя команда. SonarQube ставил ошибки в наших класса-исключениях, не поддерживающих сериализацию.

Исключения (абсолютно все!) по-умолчанию должны иметь возможность выйти за пределы текущей сборки нормально, т.е. они должны сериализоваться, и точка. Более того для этого от вас требуется всего-ничего добавить только 2 строчки кода: аттрибут и конструктор, т.е. код абсолютно не усложняется, т.к. это не те 2 строчки кода, в которые ты вчитываешься и пытаешься понять, что они делают и какую логику в себе содержат.

Если этого недостаточно, то заглянем в исходники в базовый класс `System.Exception`:
```csharp
[ClassInterface(ClassInterfaceType.None)]
[ComDefaultInterface(typeof(_Exception))]
[Serializable]
[ComVisible(true)]
public class Exception : ISerializable, _Exception
{
    // ...
}
```

Что нам скажет **Liskov Substitution Principle** о ситуации, когда базовый класс поддерживает сериализацию, а его прямой наследник не поддерживает?

**Кастомные свойства и переопределения.**

Если посмотрим, как же переопределен метод `ToString()` в самом классе `System.Exception`, то увидим, что итоговая строка, возвращаемая методом `ToString()` вернет:
Имя класса + свойство Message(если есть) + InnerException(если есть) + StackTrace(если есть) 

Код из исходников:

```csharp
public override String ToString()
{
    return ToString(true, true);
}


private String ToString(bool needFileLineInfo, bool needMessage) {
    String message = (needMessage ? Message : null);
    String s;

    if (message == null || message.Length <= 0) {
        s = GetClassName();
    }
    else {
        s = GetClassName() + ": " + message;
    }

    if (_innerException!=null) {
        s = s + " ---> " + _innerException.ToString(needFileLineInfo, needMessage) + Environment.NewLine + 
        "   " + Environment.GetResourceString("Exception_EndOfInnerExceptionStack");
    }

    string stackTrace = GetStackTrace(needFileLineInfo);
    if (stackTrace != null)
    {
        s += Environment.NewLine + stackTrace;
    }

    return s;
}
```

Смотрите сами, так ли необходимо вам переопределять метод `ToString()`. В большинстве случаев это излишне. Скорее всего достаточно будет переопределить св-во `Message` и добавить информацию из кастомных свойств туда (_пометка: это исключительно авторитетное мнение здравого смысла и экспертов со stackoverflow_)).
