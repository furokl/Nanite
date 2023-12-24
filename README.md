<p align="right">
	<img src="resources/pictures/UnrealEngine-128x128.png" alt="Unreal Engine">
</p>

# Nanite

##### ![Youtube](resources/pictures/Youtube-20x16.png) Nikolai Poliarnyi :
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

[//]: # (--- Конец вкладки: Постановка задачи ---)

<br />

</details>

<details>
<summary>Виртуальные текстуры: почему задача решаема</summary>

#### Id Tech
###### Компания, разработавшая популярные игры, засчет технологического прорыва:
> Doom, Quake, <ins>Rage</ins>

###### Они и придумали виртуальные текстуры (ранее назывались Mega Texture, прижилось Virtual Texture)

<br />

<details>
<summary>
###### Mip Map:
</summary>

<br />

<details>
<summary>

###### Наглядный пример Mip Map

</summary>

![MipMap](https://github.com/furokl/Nanite/blob/main/resources/pictures/MipMap-660x440.png)

[//]: # (--- Конец изображения: Наглядный пример Mip Map ---)

</details>

###### Есть тяжелая по тем меркам текстура ландшафта 16к х 16к
> * ! Не влезает в память видеокарты (VRAM)
> * ! Нужно перерисовывать большой обьем информации

<details>
<summary>

###### (Смотреть изображение)

</summary>

![HighResolutionN1](https://github.com/furokl/Nanite/blob/main/resources/pictures/HighResolutionN1-581x430.png)

[//]: # (--- Конец изображения: Highmap resolution №1 ---)

</details>

<br />

###### * Если объект находится далеко, он может быть не виден персонажу или являться одним пикселем
###### Напрашивается разделить ландшафт на окрестности
> * Рядом с персонажем оригинальное качество
> * По удалению от него уменьшать разрешение
###### *<ins>Mip Map</ins> - Версия текстуры у которой есть разные уровни детализации* 

<details>
<summary>

###### (Смотреть изображение)

</summary>

![HighResolutionN2](https://github.com/furokl/Nanite/blob/main/resources/pictures/HighResolutionN2-579x396.png)

[//]: # (--- Конец изображения: Highmap resolution №2 ---)

</details>

<br />

###### Это все еще не решает проблему с объемом видеопамяти (VRAM)
> ! Теперь необходимо иметь несколько сжатых версий одной и той же текстуры

<details>
<summary>

###### (Смотреть изображение)

</summary>

![HighResolutionN3](https://github.com/furokl/Nanite/blob/main/resources/pictures/HighResolutionN3-681x544.png)

[//]: # (--- Конец изображения: Highmap resolution №3 ---)

</details>

<br />

###### Тогда мы будем хранить в видеопамяти только разбитые окрестности.
> * В конечном итоге должно выйти, что объем видеопамяти равен кол-ву пикселей монитора
> * Перестаем зависить от разрешения текстуры
###### НО
> * ! Мы предполагаем, что можем автоматически определить какие части текстуры нужны
> * ! Мы предполагаем, что кто-то сам положит в видеопамять эти окрестности

<details>
<summary>

###### (Смотреть изображение)

</summary>

![HighResolutionN4](https://github.com/furokl/Nanite/blob/main/resources/pictures/HighResolutionN4-666x516.png)

[//]: # (--- Конец изображения: Highmap resolution №4 ---)

</details>

[//]: # (--- Конец вкладки: Mip Map ---)

</details>

<details>
<summary>
###### Рендер
</summary>

###### 1. Первый проход (GPU)
###### Мы смотрим на объект и проецируем его на экран
###### Чтобы скомпенсировать: чем дальше объект от игрока, тем меньше у этого объекта уровень детальности, чтобы пиксель стал сопоставим с пикселем на экране
> * Знаем размер проекции
> * Знаем уровень Mip Map

<details>
<summary>

###### (Смотреть изображение)

</summary>

![RenderN1](https://github.com/furokl/Nanite/blob/main/resources/pictures/RenderN1-590x337.png)

[//]: # (--- Конец изображения: Render №1 ---)

</details>

###### 2. Второй проход (CPU)
###### Процессор смотрит на картину: там перечислено, что нужно для построения кадра
> В Кэше хранится информация о окрестностях, что уже лежат в видеопамяти
>> Если её нет, инициализируем эту информацию.

<details>
<summary>

###### (Смотреть изображение)

</summary>

![RenderN2](https://github.com/furokl/Nanite/blob/main/resources/pictures/RenderN2-741x375.png)

[//]: # (--- Конец изображения: Render №2 ---)

</details>

###### 3. Третий проход
###### Мы гарантировали, что вся информация на картинке прогружена
###### Рисуем виртуальную текстуру на тех уровнях разрешения, на которых нужно с учетом расстояния до персонажа
> Помним, VRAM пропорционально числу пикселей на экране
###### НО
> Гарантирует ли это Readltime? ${1\over 60}$
>> * Все быстро работает за исключением ожидания подгрузки данных в VRAM (пункт 2)
>> * Повезло, если текстура влезла в оперативную память, PCI-E шина может и справится; но <ins>придется ограничивать свободу художника</ins>

<details>
<summary>

###### (Смотреть изображение)

</summary>

![RenderN3](https://github.com/furokl/Nanite/blob/main/resources/pictures/RenderN3-464x345.png)

[//]: # (--- Конец изображения: Render №3 ---)

</details>

###### Что делать?
###### Пусть инициализация подгрузки будет происходить асихронно
###### В свою очередь, прорисовка начнется сразу с тем, что есть
> * Будем всегда держать в VRAM низкодетализированную версию
>> * Если информация о окрестностях есть, заменяем низкодетализированную версию
###### *<ins>Streaming</ins> - Процесс запроса + асихронной подргузки*

[//]: # (--- Конец вкладки: Рендер ---)

</details>

<br />

[//]: # (--- Конец вкладки: Виртуальные текстуры: почему задача решаема ---)

</details>

<details>
<summary>Почему с геометрией задача сложнее</summary>

###### Есть чуйка, что мы можем применить Streaming в том числе к геометрии
###### Однако, стоит отметить, что задача связанная с геометрией не тривиально адаптируется:
> * 2D картинка фильтруема
> * Работаем с регулярной структурой нашей картинки
>> При упрощении 4 пикселя в 1, мы можем просто усреднить их цвет
###### С геометрией мы работаем с большим множеством треугольников
> * Даже если треугольники находятся рядом, мы не можем их упрощать или усреднять

[//]: # (--- Конец вкладки: Почему с геометрией задача сложнее ---)

<br />

</details>

<details>

<summary>Этапы стандартного пайплайна отрисовки</summary>

###### Рассмотрим случай стандартного OpenGL пайплайна
###### (Камера игрока смотрит на вход в пещеру)
###### Видеокарта, с наивной точки зрения, пытается проицировать ВСЮ пещеру на экран, но мы видим лишь ближайшую поверхность

<br />

###### **1. Vertex shader**

<details>
<summary>

###### (Смотреть изображение)

</summary>

![PipelineN1](https://github.com/furokl/Nanite/blob/main/resources/pictures/PipelineN1-464x250.png)

[//]: # (--- Конец изображения: Pipeline №1 ---)

</details>

###### Ближайшую поверхность мы видим из-за Frame buffer / Depth buffer, это еще один виртуальный экран:
> * Вместо цвета храним глубину (float), он же *<ins>Z / depth<</ins>*
> * Побеждает цвет с самой меньшей глубиной
###### Также изображение, находящееся за пределами угла обзора, не будет расчитываться
###### *<ins>Frustum culling</ins> - отсечение геометрии вне видимости игрока*

<br />

<details>
<summary>

###### (Смотреть изображение)

</summary>

![PipelineN2](https://github.com/furokl/Nanite/blob/main/resources/pictures/PipelineN2-218x90.png)

[//]: # (--- Конец изображения: Pipeline №2 ---)

</details>

###### **2. Rasterization**

###### *<ins>Растаризация</ins> - преобразует каждый треугольник в фрагменты (набор пикселей)*
###### У нас есть информация о трех вершинах и нам интересны пиксели находящиеся в треугольнике

<br />

###### **3. Fragment Shader**

###### *<ins>Фрагмент шейдер</ins> - обрабатывает отдельные фрагменты, строя корректное изображение*
###### Шейдер расчитывает такие параметры как:
> * Z
> * UV
> * Color
> * Lighting
>> И на выходе получаем RGB, если победили по <ins>Z-тесту</ins>

[//]: # (--- Конец вкладки: Этапы стандартного пайплайна отрисовки ---)

<br />

</details>

<details>

<summary>Оптимизация</summary>

###### ! Асимптотика Vertex shader вышла O(N), где N - число треугольников, что не может нас устраивать

<details>

<summary>

###### Кластеризация

</summary>

###### Объединим треугольники по 128, каждую такую область возьмем в Bounding Box
> * Если Box не подходит - делаем frusting culling для всех треугольников
> * Если Box частично / полностью заходит - рассматриваем треугольники более подробно

<details>
<summary>

###### (Смотреть изображение)

</summary>

![Clustering](https://github.com/furokl/Nanite/blob/main/resources/pictures/Clustering-336x393.png)

[//]: # (--- Конец изображения: Кластеризация ---)

</details>

[//]: # (--- Конец вкладки: Кластеризация ---)

</details>

<details>

<summary>

###### Иерархический Z Buffer

</summary>

###### Представим, что у нас появилось видение с различными глубинами
###### Возьмем тот же кластер Bounding Box в Depth Buffer
> * Не нужно делать Вершинный шейдер
> * Не нужно делать Растаризацию
> * ! Нужно обработать большое количество значений Z
###### Тогда продолжаем сжимать изображение до тех пор, пока не дойдем до константного значения, к примеру до 1 или 4 пикселя
> * Теперь можем осуществлять Z-Тест, если провалили проверку, эти 128 треугольников нас не интересуют
> * ! Кто дал буфер глубины?

<details>
<summary>

###### (Смотреть изображение)

</summary>

![HierarhicalBuffer](https://github.com/furokl/Nanite/blob/main/resources/pictures/HierarhicalBuffer-112x30.png)

[//]: # (--- Конец изображения: Иерархический Z Buffer ---)

</details>

###### Проекция кадров в VR:

###### Боль: при быстром движении головой, появлялись микрофризы
###### Каждый кадр - Depth Buffer + RGB
###### Будем не перерисовывать каждый кадр, а менять уже существующий, проецируя основную часть экрана
> + Плавное изображение в движении
> - Дольше отрисовка новых обьектов

<details>
<summary>

###### VR

</summary>

![VR](https://github.com/furokl/Nanite/blob/main/resources/pictures/VR-282x198.png)

[//]: # (--- Конец изображения: VR ---)

</details>

###### Точно также, как с VR: применяем иерархический Z Buffer на основе предыдущего кадра
> Удобнее запомнить, какие треугольники победили по Z-Тесту на основной части экрана

[//]: # (--- Конец вкладки: Иерархический Z Buffer ---)

</details>

[//]: # (--- Конец вкладки: Оптимизация ---)

<br />

</details>

- - -

## License

MIT

[//]: # (Created on 23/12/2023)
[//]: # (By furokl)