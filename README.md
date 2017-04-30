Что это?
========

Этот репозиторий служит для разработки и распространения драйвера для интеграции
устройств серии [систем безопасности "Астра"](http://www.teko.biz/) и
[контроллеров автоматизации WirenBoard](http://contactless.ru/).

Проверена работоспособность драйвера совместно со следующими устройствами:
1. [Wiren Board 5 rev 5.8](http://contactless.ru/wiki/index.php/Wiren_Board_5:_%D0%90%D0%BF%D0%BF%D0%B0%D1%80%D0%B0%D1%82%D0%BD%D1%8B%D0%B5_%D1%80%D0%B5%D0%B2%D0%B8%D0%B7%D0%B8%D0%B8)
2. [Астра РИ-М РР](http://www.teko.biz/catalog/823/7006/)
3. [Датчик изменения положения Астра-3531](http://www.teko.biz/catalog/675/5326/)
4. [Датчик протечки Астра-361 РК](http://www.teko.biz/catalog/295/4921/)
5. [Датчик движения Астра-5131.А](http://www.teko.biz/catalog/223/680/)
6. [Датчик открытия магнитоконтактный Астра-3321](http://www.teko.biz/catalog/333/849/)

Преверена работа с единственным Астра РИ-М на канале. Теоретически драйвер
должен работать с любым количеством РИ-М на канале (в пределах 250 штук, в связи
со спецификой протокола), а так же с Астра-Z и любыми извещателями от Теко,
совместимыми с Астра РИ-М и Астра-Z.

FYI: Рекомендую покупать Астра-Z, а не Астра РИ-М, из-за того, что у последнего
есть небольшая путаница в радиорежимах (например если вы захотите использовать
датчик протечки и датчик температуры — ничего не выйдет, т.к. первому нужен
радиорежим 1, а второму — 2), в то же время для Астра-Z нет (и не будет) датчика
изменения положения (впрочем и сам этот датчик для РИ-М снят с производства).

Перед началом работы
====================

Устройства Астра-РИ-М выходят с завода в режиме "автономный", он не
предполагает работу в качестве ведомого устройства на линии RS-485, в связи с
этим необходимо переключить РИ-М в режим "системный". Делается это заменой
прошивки, для чего потребуется загрузить и установить
[ПО ПКМ Астра Pro](http://www.teko.biz/support/programms/pc/) (во время
установки убедитесь, что среди модулей ПКМ активирована галочка рядом с модулем
смены ПО), и далее следуйте документации из [Инструкции пользователя](http://www.teko.biz/upload/rukovod/RR-RI-M_%D0%98%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%86%D0%B8%D1%8F%20%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D1%8F.pdf),
раздел "Смена ПО на РР для работы в режиме «Системный»".

Использование
=============

## Установка
Найдите последнюю версию драйвера в разделе [Releases](https://github.com/andrey-yantsen/wb-mqtt-astra/releases/latest)
и загрузите её на WirenBoard командой (не забудьте заменить ссылку на корректную)
```
wget https://github.com/andrey-yantsen/wb-mqtt-astra/releases/download/v0.1/wb-mqtt-astra_0.1_armel.deb
```

Затем установите (имя файла определяется в результате выполнения предыдущей команды):
```
dpkg -i wb-mqtt-astra_0.1_armel.deb
```

## Настройка
Отредактируйте файл `/etc/default/wb-mqtt-astra` в соответствии с вашей
установкой: для корректной работы драйвера достаточно указать опции `serial` (по
умолчанию `/dev/ttyAPP4`) и `address` (при наличии нескольких устройств Астра
РИ-М на одной шине нужно указывать адреса в таков формате: `-address 1 -address 
2 -address 3`). Если вы ещё не регистрировали РИ-М (т.е. не изменяли стандартный
адрес), то значение адреса можно выбрать любое, в диапазоне от 1 до 250.
После задания настроек запустите фоновый процесс: `invoke-rc.d wb-mqtt-astra start`.

## Привязка Астра РИ-М и извещателей
Откройте раздел `Devices` в вэб-интерфейсе WirenBoard и найдите блок с
заголовком *astra_1* (где `1` — адрес устройства, указанный при настройке).
Переключите триггер `register` в положение `on`, чтобы зарегистрировать РИ-М.
При успешной регистрации триггер `delete` перейдёт в `off`, а `register` станет
чекбоксом с установленной галочкой.

По умолчанию РИ-М будет работать в частоте "1" (соответстует извещателям с
пометкой *лит. 1*), чтобы изменить канал — необходимо переключить контрол
`l2_channel` в желаемое положение. При изменении частоты все
зарегистрированные извещатели будут забыты.

После регистрации РИ-М можно привязывать извещатели, для этого в том же
интерфейсе нужно активировать триггер `register_l2`, дождаться, пока на РИ-М
замигает светодиод *радиосеть* и установить аккумулятор в регистрируемый
извещатель. После регистрации извещатель появится в списке устройств, а триггер
`register_l2` переключится в состояние `off`.

При регистрации нового извещателя в MQTT публикутеся минимальное количество
контроллов (в интерфейсе WB будет видно только `delete` и `sensor id`), для
получения полного списка необходимо заставить извещатель сработать — намочить
датчик протечки, передвинуть датчик изменения положения и т.д.

Видео-инструкция доступна по ссылке [http://take.ms/xpOEm](http://take.ms/xpOEm).

Часть контролов отображаются в 2 экземплярах — "подтверждённая" и "не
подтверджённая" тревога, например `Channel1` и `Channel1_confirmed` для датчика
протечки. "Подтверждённая" тревога меняет своё состояние через несколько секунд
после изменения "не подтверждённой". К сожалению мне не удалось добиться
стабильной отправки с извещателей событий об изменении состояния подтверждённой
тревоги, в связи с этим я рекомендую её игнорирвать.
