# YGHT

## Code should work

Код должен работать. Что, звучит сильно очевидно? Ну да, есть такое, но не спешите с выводами. Во-первых я уверен, что каждый из вас когда нибудь запушит на ремоут неработающий код. А во-вторых работающий код это чуть больше чем "просто работающий код". На самом деле я бы хотел поговорить о двух вещах.

Первое, да, код должен работать. Я знаю что я повторяюсь, но это важный момент. Допустим есть задача которую вы не то что бы представляете как делать. И вместо того что бы заняться исследованием проблемы, попробовать сделать свой PoC вы начинаете делать декомпозицию, создавать какие то слои абстракции или писать инфраструктурный код. Так вот, это плохая идея которая закончится тем, что у вас закончится время, а вы никуда не продвинетесь. Запоминте, плохо работающий код, во много раз лучше хорошего, но не работающего кода. Сделайте сначала работающий прототип. Пусть он будет хромой, кривой и несчастный, зато когда с вас спросят - вы сможете сказать что решение уже есть, осталось его интегрировать. И перепишите его как надо. Если вы знаете как делать задачу - делайте ее хорошо. Если не знаете - решите ее хоть как-то.

Второй момент заключается в том, что есть работающий код и есть "работающий" код. Давайте посмотрим на этот пример:

``` c#
public string Do(int x)
{
    using (WebClient xx = new WebClient())
    {
        return xx.DownloadString("https://some.super.url");
    }
}
```

Это замечательный пример "работающего" когда. Почему? Потому, что он не не учитывает, что рано или поздно, этот ендпоинт отвалиться. Этот пример не учитывает так называемые edge case - пограничные, "плохие случаи". Когда вы начинаете писать код, задумайтесь о том, что может пойти не так. На самом деле я сейчас говорю не только об удаленных вызовах а обо всех ресурсах которые находятся вне зоны вашего контроля - пользовательский ввод, посторонние файлы или файлы настроек, сетевые соединения. Все что может сломаться, сломается в самый неподходящий момент и единственное что вы можете с этим сделать - быть к этому готовым настолько, насколько это возможно. //И именно здесь вам поможет принцип разделения ответственности. Если у вас большой и сложный метод - в нем гарантированно будет целая куча мест которые могут сломаться из-за данных или чего то еще.//

Теперь, когда мы знаем проблему, ее легко исправить просто обернув рискованный вызов в try/catch

``` c#
public string Do(int x)
{
    using (WebClient xx = new WebClient())
    {
        try
        {
            return xx.DownloadString("https://some.super.url");
        }
        catch (Exception e)
        {
            L(e);
            throw;
        }
    }
}
```

К сожалению, не все проблемы настолько очевидны. Вот например такой простейший метод:

``` c#
public bool IsYes(string userPrompt)
{
    return userPrompt === "c";
}
```

Ну что, что тут может пойти не так? Ничего, кроме неопредленного поведения из-за локали машины.

ПРИМЕР!

Итак, подсумируем. Сначала заставьте код рабоать, и забывайте про edge cases и обработку ошибок.

## Code should be maintainable

Второе требование к хорошему коду это про поддержку. Поддержка понятие сложное но я бы включил Наверное вы слышали, что больше всего времени мы тратим на чтение кода, а не написание нового и это правда в достаточной степени. Возьмите код, который вы написали месяц назад и попробуйте его бегло прочесть. Вы понимаете что там происходит? Если да - я вас поздравляю. если нет, значит у вас проблема с поддерживаемостью кода. И снова, я разобью этот принцип на две составляющие.

Ваш код должен легко читаться и легко меняться. Начнем с первого.

Первое это именования. Возьмем наш предыдущей пример

``` c#
public string Do(int x)
{
    using (WebClient xx = new WebClient())
    {
        return xx.DownloadString("https://some.super.url");
    }
}
```

Метод который называется Do, переменные которые называются x и xx. Это конечно читабельно, но непонятно. Я думаю вы уже неоднократно это слышали но повторюсь. Названия классов, методов и переменных должны быть понятными и нести смысловую нагрзуку. Грубо говоря, вы не должны смотреть в код, что бы понять что же на самом деле делает этот класс или метод. Это не так просто, как может показаться. Есть такая вот довольно известная цитата. Подумайте о ней.

«There are only two hard things in Computer Science: cache invalidation and naming things.»  (C) Phil Karlton

Давайте перепишем наш пример на что-то более внятное:

``` c#
public string GetUserName(int userId)
{
    using (var http = new WebClient())
    {
        try
        {
            return http.DownloadString("https://some.super.url");
        }
        catch (Exception e)
        {
            LogException(e);
        }
    }
}
```

Теперь нам не нужно смотреть в код что бы понять что же конкретно возвращает нам метод. Вторая составляющая читабельного кода это его структура. Да, сейчас я говорю про тех, кто любит писать шесть вложеных ифов, или вписывать колбку в колбек колбека внутри колбека. Пожалейте ваших коллег и самих себя. Спустя неделю вам придется буквадно продираться сквозь этот код что бы что-то в нем исправить или добавить. По возможности придерживайтесь плоской структуры кода с небольшим количеством ветвлений.

Теперь поговорим про изменения. Кому знаком термин Big ball of mud? Если кому то не знаком - посмотрите на картинку. Каждый модуль зависит от каждого модуля и изменения контракта в одном месте скорее всего приведет к пришествию полярной лисички или, как минимум, к очень длительному дебагу. Что с этим делать? Избегайте циклических зависимостей. Объединяйте и выделяйте слои вашего кода. Следите за потоком данных. Обычно, если у класса слишком много зависимостей, он выполняет слишком много работы. Разгрузите его. В этом плане мне очень нрав
F# который подталкивает нас к линейному коду.

## Code should be performant enough

Наверное все слышали, что преждевременные оптимизации это зло. Это правда, но правда так же и то, что вы должны знать свой инструмент и не писать на нем так, что бы веб клиенты почты загружал core I7 на 60%. Давайте снова обратимся к нашему примеру. Казалось бы с ним все хорошо и там нечего оптимизировать. Но это не так. Если присмотреться, то мы увидим  синхронное выполнение загрузки по сети. Это I/O операция которая, в этом случае, заблокирует наш поток до своего выполнения. В десктопных приложениях это приведет к подвисшему юаю, в серверных, в бесполезной резервации памяти и исчерпания ресурсов ThreadPull-а. Т.е. мало того что вы бесполезно расходуете память (напомню на каждый процесс нам выделюят память под стек), но и заставляете ThreadPull создавать новые потоки (что не так уж и быстро) вместо того что бы их освобождать. Конечно, эту проблему решить легко - использовать асинхронные операции. Но я говорю не конкретной проблеме. Я говорю о принципе - знайте свой инструмент и не создавайте проблем на пустом месте.

## Be tested with reasonable coverage

Это еще одна холиварная тема, но давайте ее тоже коснемся. Зачем нам вообще нужен CodeCoverga? В идеальном мире - не нужен. В идеальном мире код пишут без багов, а требования никогда не меняются. Но мы живем в далеко не идеальном мире, поэтому тесты нам нужны для того, что бы быть уверенными что код работает действительно правильно (там нет багов) и что код работает по-прежнему правильно, после того как что-то поменялось. Значит ли это, что код должен покрывать все что только возможно? Нет, не значит. Во-первых это занимает время, а времени всегда не хватает. Во-вторых тесты так же становятся вашей кодовой базой за которой надо приглядывать и время от времени поправлять. И в третьих это не приносит значимой пользы, потому что заветный 100% код ковер ничего не гарантирует. Вы все равно где-то что-то не учтете. Так, что, ура!? Тесты можно не писать? И снова нет, тесты писать нужно. Но нужено учиться понимать, какие конкретно вещи должны быть покрыты тестами. В основном речь идет о логике. Нет смысла тестировать, что если вы присвоете свойству значение оно таки присвоится. А вот проверить вычисляемый флаг зависящий от этого свойста уже будет иметь некоторый смысл.
