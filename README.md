# Домашнее задание к занятию 1. «Типы и структура СУБД» - `Горбачёв Олег`
## Введение
Перед выполнением задания можно ознакомиться с [дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/virt-11/additional)

## Задача 1

Архитектор ПО решил проконсультироваться у вас, какой тип БД 
лучше выбрать для хранения определённых данных.

Он вам предоставил следующие типы сущностей, которые нужно будет хранить в БД:
Выберите подходящие типы СУБД для каждой сущности и объясните свой выбор.

- `электронные чеки в json-виде` - *Определенно, документоориентированная СУБД. Например: MongoDB. Так как в данном случае схема данных не определена заранее. В 
чеках может быть разное кол-во позиций и полей.*
- `склады и автомобильные дороги для логистической компании` - *Графовая СУБД. Neo4j. Позволит реализовать механизм рекомендаций оптимальных маршрутов до требуемых точек назначения.*
- `генеалогические деревья` - *Всё та же графовая СУБД. Всё та же Neo4j. Думаю такой тип СУБД отлично подойдет, ведь генеалогическое древо, по сути, граф и есть.*
- `кэш идентификаторов клиентов с ограниченным временем жизни для движка аутенфикации` - *Понятно, что нужно организовать систему кэширования. Значит, тип СУБД - "ключ-значение". Например, Redis.*
- `отношения клиент-покупка для интернет-магазина` - *Я бы выбрал реляционную СУБД. К примеру, MySQL. Хорошо БД интернет-магазина, которая включает в себя множество сущностей и отношений.*

## Задача 2

Вы создали распределённое высоконагруженное приложение и хотите классифицировать его согласно 
CAP-теореме. Какой классификации по CAP-теореме соответствует ваша система, если 
(каждый пункт — это отдельная реализация вашей системы и для каждого пункта нужно привести классификацию):

- данные записываются на все узлы с задержкой до часа (асинхронная запись);
- при сетевых сбоях система может разделиться на 2 раздельных кластера;
- система может не прислать корректный ответ или сбросить соединение.

Согласно PACELC-теореме как бы вы классифицировали эти реализации?

### Ответ:
| Вводные | CAP | PACELC |
| ----------- |:-------------:|:-------------:|
| данные записываются на все узлы с задержкой до часа (асинхронная запись) | CP | PC/EL |
| при сетевых сбоях система может разделиться на 2 раздельных кластера | AP или CA | PA/EL или PC/EL |
| система может не прислать корректный ответ или сбросить соединение | CP | PA/EC |

## Задача 3

Могут ли в одной системе сочетаться принципы `BASE` и `ACID`? Почему?

### Ответ:
Данные принципы не могут сочетаться в одной системе. Они противоречат друг другу. И главное противоречие состоит в подходе к согласованности данных в системе. Так согласно `ACID`, если в системе не согласованы данные, то никакая транзакция не завершится успешно. В модели `BASE` основной упор сделан на высокую доступность, а к требования к согласованности данных гораздо мягче: когда-то данные должны прийти в согласованное состояние, но в какой момент это произойдет, не регламентируется.

## Задача 4

Вам дали задачу написать системное решение, основой которого бы послужили:

- фиксация некоторых значений с временем жизни,
- реакция на истечение таймаута.

Вы слышали о `key-value`-хранилище, которое имеет механизм `Pub/Sub`. 
Что это за система? Какие минусы выбора этой системы?

### Ответ:
Таким решением может стать `Redis`. К минусам я бы отнес:
1. однопоточность `Redis` (он не может распараллелить выполнение задач по имеющимся ресурсам. Задействует 1 ядро)
2. так как данные загружены в ОЗУ, существует риск их потери (есть механизмы сохранения данных на диск. `RDB`, `AOF`. При этом `RDB` позволяет сохранить производительность на высоком уровне, но всё же подразумевает потерю части данных (между точками сохранения) в случае сбоя. А при использовании `AOF` производительность теряется). 
3. В случае жесткого завершения работы Redis, существует риск частичного выполнения транзакции. 
4. `Redis Cluster` не дает гарантий согласованности данных. Существует риск потери записей в процессе асинхронной репликации. 
5. `Pub/Sub Redis` отправитель  отправляет сообщение один раз и не ждет никакого подтверждения получения сообщения от подписчика. Если по какой-то причине сообщение не дошло - оно будет утеряно.

