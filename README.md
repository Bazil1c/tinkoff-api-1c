﻿# REST API клиент Тинькофф Банк для 1С

REST клиент 1С для получения данных из личного кабинета Тинькофф Банка.

## Возможности
- Двухфакторная авторизация с использованием кода подтверждения по СМС.
- Получение данных по операциям за период.

## Требования
- Платформа **8.3.10** и выше.
- Библиотека стандартных подсистем **2.4** и выше.

## Установка
Скопируйте общие модули к себе в конфигурацию.

## Пример использования

### Получение операций при простой авторизации

```bsl
&НаКлиенте
Процедура ПолучитьОперации(Команда)
	
	ПолучитьОперацииНаСервере();
	
КонецПроцедуры

&НаСервере
Процедура ПолучитьОперацииНаСервере() 
	
	Перем Сессия;
	
	НачалоПериода = НачалоМесяца(ТекущаяДатаСеанса());
	КонецПериода = ТекущаяДатаСеанса();

	ТинькоффApiСервер.ВыполнитьАвторизацию(Сессия, "Bazil1c", "MyPassword");

	Операции = ТинькоффApiСервер.ПолучитьОперацииЗаПериод(Сессия, 
		НачалоПериода, КонецПериода); 
	
КонецПроцедуры
```

### Получение операций при двухфакторной авторизации
Асинхронные вызовы используются, так как необходимо вводить код подтверждения.

```bsl
&НаКлиенте
Процедура ПолучитьОперации(Команда)
	
	Сессия = Неопределено;
	
	ДополнительныеПараметры = Новый Структура;
	ДополнительныеПараметры.Вставить("НачалоПериода", НачалоМесяца(ОбщегоНазначенияКлиент.ДатаСеанса()));
	ДополнительныеПараметры.Вставить("КонецПериода", ОбщегоНазначенияКлиент.ДатаСеанса());
	ОписаниеОповещения = Новый ОписаниеОповещения("ПолучитьОперацииПродолжение", ЭтотОбъект, ДополнительныеПараметры);
	
	ТинькоффApiКлиент.Авторизация(ОписаниеОповещения, "Bazil1c", "MyPassword");
	
КонецПроцедуры

&НаКлиенте
Процедура ПолучитьОперацииПродолжение(Сессия, ДополнительныеПараметры = Неопределено) Экспорт
	
	Операции = ТинькоффApiВызовСервера.ПолучитьОперацииЗаПериод(Сессия, 
		ДополнительныеПараметры.НачалоПериода, ДополнительныеПараметры.КонецПериода); 
	
КонецПроцедуры
```
