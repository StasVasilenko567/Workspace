Механизм обнаружения изменений. Проходит от корневого компонента до последнего.

Под капотом у Angular стоит библиотека Zone.js. Ее предназначение в том, чтобы создавать "зоны". Это избавляет нас от проблемы потери контекста у асинхронных функций. Также, эта библиотека используется для обнаружения изменений. Для каждой операции, создается зона, после ее окончания вызывается прогон проверок на изменения.

У ChangeDetection есть 2 стратегии:
**Default** - (по умолчанию). Проверка компонентов, включая их дочерние на изменения, если происходит асинхронные вызовы или события.
**OnPush** - Компоненты проверяются только при изменении входных данных компонента (объявленные через @Input)

Также проверку на изменения можно проводить и вручную с помощью сервиса **ChangeDetectionRef**. Например:

```ChangeDetectionRef.detectChanges()``` - метод вручную проверяет на наличие изменений.
```ChangeDetectionRef.markForCheck()``` - метод маркирует компонент, если выставлена стратегия OnPush

#### Когда запускается `ChangeDetection`:

- После события пользователя.
- После выполнения асинхронных операций (например, HTTP-запросов).
- После таймеров, таких как `setTimeout`, `setInterval`.
- При обновлениях через `Observable` или Promises.

## NgZone

NgZone является оберткой Zone.js, или вернее *интерфейсом* между Angular и Zone.js
Это сервис, позволяет запускать процессы внутри/вне зоны Ангуляра. Если процесс запущен вне зоны, то и вызова ChangeDetection тоже не будет.

#### Методы ngZone

``` run(fn: Function) ``` - Выполняет функцию внутри зоны Ангуляр.
``` runOutsideAngular(fn: Function) ``` - Выполняет вне зоны Ангуляр.