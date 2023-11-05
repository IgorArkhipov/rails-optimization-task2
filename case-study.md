# Case-study оптимизации

## Актуальная проблема
В нашем проекте возникла серьёзная проблема.

Необходимо было обработать файл с данными, чуть больше ста мегабайт.

У нас уже была программа на `ruby`, которая умела делать нужную обработку.

Она успешно работала на файлах размером пару мегабайт, но для большого файла она работала слишком долго, и не было понятно, закончит ли она вообще работу за какое-то разумное время.

Я решил исправить эту проблему, оптимизировав эту программу.

## Формирование метрики
Для того, чтобы понимать, дают ли мои изменения положительный эффект на быстродействие программы я придумал использовать такую метрику: объем используемой памяти на протяжении исполнения программы.

## Гарантия корректности работы оптимизированной программы
Программа поставлялась с тестом. Выполнение этого теста в фидбек-лупе позволяет не допустить изменения логики программы при оптимизации.

## Feedback-Loop
Для того, чтобы иметь возможность быстро проверять гипотезы, я выстроил эффективный `feedback-loop`, который позволил мне получать обратную связь по эффективности сделанных изменений за меньшее 30 секунд время.

Вот как я построил `feedback_loop`: предоставленные данные разбиваются на меньшие объемы (100, 1000 ... 10000 строк), и уже на основе них строятся тесты для фиксирования текущей производительности и используемой памяти в процессе работы как отправной точки. Для этого код самой программы незначительно модифицируется, чтобы поддерживать параметры запуска: имя файла с данными и режим работы Garbage Collector. Самая первая итерация представляла из себя написание тестов производительности (время выполнения программы, объем используемой памяти в процессе работы). В данном случае это можно отнести к шагу Profile & Test & Benchmark. Полученные при первом запуске значения бенчмарка мы можем записать в цели теста для исключения регрессий на последующих итерациях.
С этого момента наш feedback loop создан, и мы можем переходить к профилированию и изменению кода, после чего запускать уже написанные тесты для сравнения результатов с отправным шагом (или шагом предыдущей итерации).

## Вникаем в детали системы, чтобы найти главные точки роста
Для того, чтобы найти "точки роста" для оптимизации я воспользовался инструментами memory-profiler, ruby-prof и stackprof после завершения первого этапа, в котором исходный код был изменен, чтобы файл с данными читался и обрабатывался построчно, а не загружался целиком в память перед обработкой.

Вот какие проблемы удалось найти и решить

### Ваша находка №1
- какой отчёт показал главную точку роста
  * memory_profiler с профилем на 10.000 строк указал на MEMORY USAGE: 103 MB, Total allocated: 18.92 MB (259923 objects), Total retained: 4.79 kB (9 objects). Строка, где создавалось больше всего объектов - это та, где происходит конвертация формата даты пользовательской сессии. Эта строка станет главной точкой роста
- как вы решили её оптимизировать
  * на этой ознакомительной итерации были удалены избыточные преобразования дат во время обработки полученной информации о сессиях для каждого пользователя
- как изменилась метрика
  * MEMORY USAGE: 76 MB, Total allocated: 11.63 MB (155267 objects), Total retained:  40.00 B (1 objects)
- как изменился отчёт профилировщика
  * на первое место в отчете профилировщика теперь попала строка `cols = line.split(',')`

### Ваша находка №2
- какой отчёт показал главную точку роста
- как вы решили её оптимизировать
- как изменилась метрика
- как изменился отчёт профилировщика

### Ваша находка №X
- какой отчёт показал главную точку роста
- как вы решили её оптимизировать
- как изменилась метрика
- как изменился отчёт профилировщика

## Результаты
В результате проделанной оптимизации наконец удалось обработать файл с данными.
Удалось улучшить метрику системы с *того, что у вас было в начале, до того, что получилось в конце* и уложиться в заданный бюджет.

*Какими ещё результами можете поделиться*

## Защита от регрессии производительности
Для защиты от потери достигнутого прогресса при дальнейших изменениях программы *о performance-тестах, которые вы написали*