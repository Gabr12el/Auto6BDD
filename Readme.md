# Project Sample: [![Build status](https://ci.appveyor.com/api/projects/status/n6es6q3kofj5n7mk?svg=true)](https://ci.appveyor.com/project/Gabr12el/auto6bdd)

# Домашнее задание к занятию «2.4. BDD»

В качестве результата пришлите ссылки на ваши GitHub-проекты в личном кабинете студента на сайте [netology.ru](https://netology.ru).

Все задачи этого занятия нужно делать **в разных репозиториях**.

**Важно**: если у вас что-то не получилось, то оформляйте Issue [по установленным правилам](../report-requirements.md).

**Важно**: не делайте ДЗ всех занятий в одном репозитории! Иначе вам потом придётся достаточно сложно подключать системы Continuous Integration.

## Как сдавать задачи

1. Инициализируйте на своём компьютере пустой Git-репозиторий
1. Добавьте в него готовый файл [.gitignore](../.gitignore)
1. Добавьте в этот же каталог код ваших авто-тестов
1. Сделайте необходимые коммиты
1. Добавьте в каталог `artifacts` целевой сервис (`app-ibank-build-for-testers.jar`)
1. Создайте публичный репозиторий на GitHub и свяжите свой локальный репозиторий с удалённым
1. Сделайте пуш (удостоверьтесь, что ваш код появился на GitHub)
1. Удостоверьтесь, что в Appveyor сборка выполняется: запускается тестируемый сервис и тесты. При отсутствии багов в сервисе сборка должна быть зелёной
1. Поставьте бейджик сборки вашего проекта в файл README.md
1. Ссылку на ваш проект отправьте в личном кабинете на сайте [netology.ru](https://netology.ru)
1. Задачи, отмеченные, как необязательные, можно не сдавать, это не повлияет на получение зачета
1. Если вы обнаружили подозрительное поведение SUT (похожее на баг), создайте описание в Issue на GitHub. [Придерживайтесь схемы при описании](../report-requirements.md).
1. Если в проекте реализован тест(тесты), направленные на поиск описанных в Issues багов тестируемого сервиса, то такие тесты будут падать до исправления багов сервиса, сборка в Appveyor будет красной

## Настройка CI

Настройка CI осуществляется аналогично предыдущему заданию. Поскольку у вас итак "специальная тестовая сборка", то ничего в самом сервисе делать не нужно.

## Задача №1 - Page Object's

Вам необходимо "добить" тестирование функции перевода с карты на карту. Разработчики пока реализовали возможность перевода только между своими картами, но уже хотят, чтобы вы всё протестировали.

Для этого они не поленились и захардкодили вам целого одного пользователя:
```
* login: 'vasya'
* password: 'qwerty123'
* verification code (hardcoded): '12345'
* cards:
    * first:
        * number: '5559 0000 0000 0001'
        * balance: 10 000 RUB
    * second:
        * number: '5559 0000 0000 0002'
        * balance: 10 000 RUB
```

После логина (который уже мы сделали на лекции), вы получите список карт:

![](pic/cards.png)

Нажав на кнопку "Пополнить" вы перейдёте на страницу перевода средств:

![](pic/transfer.png)

При успешном переводе, вы вернётесь назад на страницу со списком карт.

Это ключевой кейс, который нужно протестировать.

Нужно, чтобы вы через Page Object'ы добавили доменные методы:
* Перевода с определённой карты на другую карту n'ой суммы
* Проверки баланса по карте (со страницы списка карт)

**Вы можете познакомиться с некоторыми подсказками [по реализации этой задачи](balance.md)**.

PS: чтобы вам было не скучно, мы там добавили порядком багов, поэтому как минимум одно Issue в GitHub у вас должно быть 😈.

<details>
    <summary>Подсказка</summary>

    Обратите внимание на то, что ваши тесты должны проходить целиком (т.е. весь набор тестов). Мы как всегда заложили там небольшую ловушку, чтобы вам не было скучно 😈.
    
    Не закладывайтесь на то, что на картах для каждого теста всегда одна и та же фиксированная сумма, подумайте, как работать с SUT так, чтобы не приходилось её (SUT) перезапускать для каждого теста.
</details>

Придерживайтесь схемы при описании:

Для того, чтобы получить информацию о текущем балансе карт, вам необходимо написать вспомогательный метод (в одном из ваших PageObjects), который из содержимого элемента получает значение баланса.

Как работать с объектами, методами (чистая Java), селекторами (Selenide) и Page Object'ами вы уже знаете, поэтому нас будет интересовать только техническая сторона вопроса.

Итак, переходим на нужную нам страницу и открываем DevTools, выбирая элемент с балансом:

![](pic/DOM.png)

Видим следующую вещь: содержимое элемента представляет из себя следующую строку:

```
**** **** **** 0001, баланс: 10000 р.
```

Но помимо строки с данными, там же есть ещё кнопка?

Давайте накидаем Page Object и посмотрим, что мы получим:

![](pic/text.png)

Т.е. метод `text` возвращает и текст на кнопке тоже.

В итоге мы получаем:
```
**** **** **** 0001, баланс: 10000 р.\nПополнить
```

Теперь давайте внимательно посмотрим на эту строку: сама сумма располагается между `баланс: ` и `р.`. Т.е. если мы сможем "вытащить" оттуда эту часть строки и преобразовать её к `int`, то получим нужные нам данные*.

Примечание*: мы не стали усложнять для вас пример, но вы должны помнить о копейках и о том, что некоторые банки не показывают копейки, если они по нулям (00), а показывают только если там действительно они есть:

![](pic/with-kopecks.png)

![](pic/without-kopecks.png)

Примечание: скриншоты Банка Тинькофф

А ещё запятые вместо точек, пробелы и т.д. - в общем, полная жуть.

Поэтому придётся мучаться либо договариваться с разработчиками, чтобы они куда-нибудь положили сумму в атрибут (в нормальном виде).

Итак - у нас есть строка, из неё нужно вырезать кусок. Давайте обратимся к документации класса [`String`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html).

Нас будут интересовать два метода:
1. `indexOf` - возвращает позицию, с которой начинается подстрока в строке
1. `substring` - вырезает подстроку из строки

Почитайте на них обязательно документацию, мы же покажем вам упрощённую реализацию:
```java
public class DashboardPage {
  // к сожалению, разработчики не дали нам удобного селектора, поэтому так
  private ElementsCollection cards = $$(".list__item div");
  private final String balanceStart = "баланс: ";
  private final String balanceFinish = " р.";

  public Dashboard() {
  }

  public int getFirstCardBalance() {
    val text = cards.first().text();
    return extractBalance(text);
  }

  private int extractBalance(String text) {
    val start = text.indexOf(balanceStart);
    val finish = text.indexOf(balanceFinish);
    val value = text.substring(start + balanceStart.length(), finish);
    return Integer.parseInt(value);
  }
}
```

Таким образом, метод `getFirstCardBalance` умеет получать баланс карты. Не сложно его модифицировать под то, чтобы он назывался `getCardBalance` и умел получать баланс по индексу (см. документацию на `ElementsCollection`).

Важно: это не единственный способ. Можно вычленить баланса через `split(":")` а потом `substring(0, .indexOf("р.")).trim()`;

Ну а дальше по аналогии вы можете реализовать всё остальное, если знаете баланс (а ещё лучше отдавать вызывающему коду не индекс карты, а её id, по которому потом можно проверять успешность или неуспешность проведения платежей). Например:

```java
public class DashboardPage {
  // к сожалению, разработчики не дали нам удобного селектора, поэтому так
  private ElementsCollection cards = $$(".list__item div");
  private final String balanceStart = "баланс: ";
  private final String balanceFinish = " р.";

  public Dashboard() {
  }

  public int getCardBalance(String id) {
    // TODO: перебрать все карты и найти по атрибуту data-test-id
    return extractBalance(text);
  }

  private int extractBalance(String text) {
    val start = text.indexOf(balanceStart);
    val finish = text.indexOf(balanceFinish);
    val value = text.substring(start + balanceStart.length(), finish);
    return Integer.parseInt(value);
  }
}
```