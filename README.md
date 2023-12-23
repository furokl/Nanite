<p align="right">
	<img src="resources/pictures/UnrealEngine-128x128.png" alt="Unreal Engine">
</p>

# Nanite

##### ![image](resources/pictures/Youtube-20x16.png) Nikolai Poliarnyi :
###### *Как работает Nanite в Unreal Engine 5*
- - -

<details>

<summary>Постановка задачи / мечта</summary>

|  | Кино | Игры|
|:------|:------:|:------:|
| Отрисовка | Offline | Realtime ${1\over 60}$ |
| Скорость обработки | Высокое качество | Бюджет качества |
| Подготовка ассетов | **Оригинал** | **Упрощаем assets** |

> Боль игр: Упрощение assets
>> * Время людей
>> * Специфика задачи
>> * Деньги

> Боль кино: Отрисовка
>> * Не хочется долго ждать результата

##### Хотим отдать задачу «упрощение assets» движку Unreal Engine, чтобы удовлетворить все запросы

<br />

</details>

[//]: # (--- Следующая вкладка ---)

<details>

<summary>Виртуальные текстуры: почему задача решаема</summary>

#### Id Tech
###### Компания, разработавшая популярные игры, засчет технологического прорыва:
> Doom, Quake, <ins>Rage</ins>

###### Они и придумали виртуальные текстуры (ранее назывались Mega Texture, прижилось Virtual Texture)

<br />

<details>

<summary>Mip Map: </summary>

###### Есть тяжелая по тем меркам текстура ландшафта 16к х 16к
> * ! Не влезает в память видеокарты (VRAM)
> * ! Нужно перерисовывать большой обьем информации

<details>

<summary>Highmap resolution №1</summary>

![image](resources/pictures/HighResolutionN1-581x430.png)

</details>

<br />

###### * Если объект находится далеко, он может быть не виден персонажу или являться одним пикселем
###### Напрашивается разделить ландшафт на окрестности
> * Рядом с персонажем оригинальное качество
> * По удалению от него уменьшать разрешение
###### *<ins>Mip Map</ins> - Версия текстуры у которой есть разные уровни детализации* 

<details>

<summary>Highmap resolution №2</summary>

![image](resources/pictures/HighResolutionN2-579x396.png)

</details>

<br />

###### Это все еще не решает проблему с объемом видеопамяти (VRAM)
> ! Теперь необходимо иметь несколько сжатых версий одной и той же текстуры

<details>

<summary>Highmap resolution №3</summary>

![image](resources/pictures/HighResolutionN3-681x544.png)

</details>

<br />

###### Тогда мы будем хранить в видеопамяти только разбитые окрестности.
> * В конечном итоге должно выйти, что объем видеопамяти равен кол-ву пикселей монитора
> * Перестаем зависить от разрешения текстуры
###### НО
> * ! Мы предполагаем, что можем автоматически определить какие части текстуры нужны
> * ! Мы предполагаем, что кто-то сам положит в видеопамять эти окрестности

<details>

<summary>Highmap resolution №3</summary>

![image](resources/pictures/HighResolutionN4-666x516.png)

</details>

<br />

</details>

<br />

<details>

<summary>Mip Map, наглядный пример</summary>

![image](resources/pictures/MipMap-660x440.png)

</details>

<br />

</details>

<br />

[//]: # (--- Следующая вкладка ---)

- - -

## License

MIT

[//]: # (Created on 23/12/2023)
[//]: # (By furokl)