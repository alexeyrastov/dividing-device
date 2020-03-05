# Электронная зубофрезерная делительная голова


Реализация электронной зубофрезерной делительной головы на Ардуино.
Для успешного запуска проекта необходима настройка и соединение вместе трех компонентов: механических, электронных и программных.

### Основной функционал:
 - Нарезание шестеренок червячной фрезой
 - Деление окружности на части (для нарезания шестенок дисковой фрезой и/или получения квадрата, шестигранника, снятия лысок и т.п.)

### Вспомогательный функционал:
 - Смена направления вращения заготовки
 - Индикация текущих оборотов шпинделя
 - Вращение заготовки в делительной головке (необходимое для настройки, центрирования, проворота и т.п.) 

### Список компонентов, требуемых для полноценного запуска делительной зубофрезерной головы:
 - энкодер
 - УДГ (или четвертая ось для станков с ЧПУ)
 - шаговый двигатель
 - STEP/DIR драйвер управления ШД
 - Arduino Mega или Arduino Uno
 - LCD Keypad Shield
 - питание для драйвера ШД
 - питание для Ардуино, энкодера

### Порядок настройки и запуска проекта:
1) Собрать все механические и электронные компоненты вместе
2) Скачать последнюю версию кода ПО с Github
3) Прописать параметры вашей конфигурации в скетче для Ардуино.
4) Загрузить код, скомпилировать программу, залить прошивку в Ардуино
5) Проверить функционал в работе (нет ли ошибок, неточностей, некорретной работы отдельных элементов)
6) При необходимости повторять п. 1. - 5. до достижения нужного результата
7) Собирать все в прочный постоянный защищенный корпус
8) Использовать в работе

Перемещение по меню, ввод параметров реализованы на **LCD Keypad Shield**. Доступно пять кнопок (шестая - "Reset", перезагрузка контроллера), нажимая на них задаем нужные параметры. Кнопки **"Вверх"**, **Вниз"** отвечают за выбор пунктов меню, **"Вправо"**, **"Влево"** - уменьшение/увеличение/смена параметра, **"Select"** - запуск/остановка выбранной программы.

При нарезании шестеренок червячной фрезой скорость вращения заготовки будет подстраиваться под скорость вращения шпинделя в режиме реального времени.

### Синхронность вращения (без потери шагов) зависит от:
1) Разрешения энкодера (количество линий на полный оборот)
2) Передаточного соотношения в УДГ
3) Установленном числе микрошагов ШД
4) Скорости вращения шпинделя
5) Требуемого числа зубьев на шестеренке

Комбинируя и соотнося эти параметры между собой, можно добиться успешного нарезания шестеренки.

#### Ограничения, которые необходимо учитывать для того, чтобы добиться устойчивого сихронного вращения заготовки и шпинделя:
 - максимальная скорость вращения шагового двигателя (не более 400-600 об./мин.)
 - максимальная частота обработки сигналов энкодера. 

Возможно частичное (без использования энкодера) использование делительной головы. Нарезание шестеренок в этом случае возможно путем простого деления окружности на части. В случае, когда число частей (на которые делится окружность) без остатка делится на число требуемых шагов ШД (необходимых для одного полного оборота заготовки) то деление будет точным. 

### Технические детали (по коду программы):
 - Функция слежения за положением заготовки (в зависимости от получаемых импульсов с энкодера) реализована на прерываниях.
 - При делении окружности на части происходит программное подстраивание, округление числа шагов для достижения полного оборота заготовки (с минимально возможной погрешностью).
 - Управление шагами ШД реализовано путем подачи сигнала STEP на контроллер; скорость вращения (обороты в минуту) зависит от задержек между посылаемыми сигналами (этот параметр настраивается в конфигурации прошивки).


-------------

### Примеры расчетов (для выбора энкодера и связки УДГ с ШД).

-------------

##### Пример 1: Расчет минимально возможного количества зубьев в шестерне
#### 
**Коэффициент (К)** = "Количество линий энкодера" __*__ "Требуемое количество зубьев в шестерне" __/__ "Количество шагов ШД на один оборот заготовки"

Для устойчивой работы "К" должен быть больше (или равен) **1** (единице). 
От соотношения **"Количество линий энкодера"** / **"Количество шагов ШД на один оборот заготовки"** зависит минимально возможное количество нарезаемых зубьев. 

**Исходные данные**:
 - Энкодер на 500 линий
 - Число требуемых шагов ШД, для одного полного оборота заготовки = 200 * 4 * 6 = 4800 шагов (шаговый двигатель - 200 импульсов на оборот; выставлен микрошаг 1:4; соотношение шкивов ШД : УДГ = 1 : 6)
 - Число нарезаемых зубьев: 30

```
500 * 30 / 4800 = 3.125
```
**Выводы**:
 - 3.125 > 1 - это значит, что на 3.125 шага энкодера необходимо сделать 1 (один) шаг ШГ.
 - 3.125 - рациональное число (деление произошло нацело, без остатка в периоде, как, например: 10 / 3 = 3,333333...). Нарезание зубьев будет происходить успешно, без накопления погрешности.

В случае необходимости **нарезать 6 зубьев** расчеты будут следующими:
```
500 * 6 / 4800 = 0.75
```
**Выводы**: 0.75 < 1 - нарезание зубьев не может быть произведено стабильно.

-------------

##### Пример 2: Расчет возможности точного (без погрешности) нарезания зубьев заготовки
#### 
Чтобы не набегала погрешность при сихронном вращении заготовки и шпинделя - энкодер надо брать кратный количеству шагов ШД на оборот заготовки.

**Исходные данные**:
 - Энкодер на 500 линий
 - Число требуемых шагов ШД, для одного полного оборота заготовки = 200 * 4 * 6 = 4800 шагов
 - Число нарезаемых зубьев: 13
```
500 * 13 / 4800 = 1,3541666...
```
**Выводы**: 1,3541666... - рациональное число с остатком в периоде. При нарезании шестеренок будет накапливаться погрешность.

-------------

##### Пример 3: Изменим передаточное соотношение ШД : УДГ с "1 : 6" на "1 : 2"
#### 
**Исходные данные**:
 - Энкодер на 500 линий
 - Число требуемых шагов ШД, для одного полного оборота заготовки = 200 * 4 * 2 = 1600 шагов
 - Число нарезаемых зубьев: 13
```
500 * 13 / 1600 = 4.0625
```
**Выводы**: 4.0625 - рациональное число (деление произошло нацело, без остатка в периоде). Нарезание зубьев будет происходить успешно, без накопления погрешности.

-------------

##### Пример 4: Деление окружности на части
#### 
**Исходные данные**: ШД - 200 импульсов на оборот, выставлен микрошаг 1:4, соотношение шкивов ШД : УДГ = 1 : 6.
Число требуемых шагов ШД, для одного полного оборота заготовки = 200 * 4 * 6 = 4800 шагов
```
4800 / 24 = 200
```
**Выводы**: 200 - целое число, без остатка. Значит, деление (нарезание зубьев шестерни) будет точным.

Точность зубьев в случае не целых соотношений (например: 17, 19, 27, 35, 49 зубьев) получается с небольшой погрешностью. Эта погрешность тем меньше, чем больше передаточное соотношение между ШД и УДГ; чем более мелкий микрошаг выставлен на драйвере.

----------------
#### 
### Ответственность
 -  Использование ПО **"Электронная зубофрезерная делительная голова"** не предполагает каких-либо обязательств со стороны автора.
 -  Автор также не несет ответственности за ущерб, причиненный в результате использования данного ПО.
 - ПО **"Электронная зубофрезерная делительная голова"** поставляется "как есть" в соответствии с принципом "AS IS", общепринятым в международной компьютерной практике. Это означает, что за возникающие в процессе эксплуатации ПО проблемы разработчик ответственности не несет. Однако со стороны разработчика делается все возможное, чтобы подобных проблем не возникало.



