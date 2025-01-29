Библиотека, встроенная в Ангуляр, по сути, является "модом на ассинхронность". Предоставляет потоки, в качестве паттерна **Наблюдатель** (Observer)

## Потоки

Потоки (observables) - Замена промисов. Реализовано через паттерн Наблюдатель. Имеет такой функционал:
1. Подписка через subscribe. При создании подписки, мы передаем колбек, который будет исполняться, когда в поток передадут данные. Пример:
```
this.options?.changes.subscribe(
	(c) => { c.toArray().forEach((item: ElementRef) => {
		if (index === this.getIndexById(this.firstSelected)) {
			item.nativeElement.classList.add('checkmark__selected');
		} else {
			item.nativeElement.classList.add('checkmark');
		}
	index++;
	})
});
```