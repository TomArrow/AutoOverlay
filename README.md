# AutoOverlay AviSynth plugin

### Требования
- AviSynth 2.6 и выше, AviSynth+, рекомендуется последняя версия x64: https://github.com/pinterf/AviSynthPlus/releases
- AvsFilterNet plugin https://github.com/mysteryx93/AvsFilterNet/ (включен в сборку, предыдущие версии работают некорректно)
- .NET framework 4.0 и выше

### Установка
- Скопировать x86 и/или x64 версии AvsFilterNet в папку с плагинами.
- Скопировать AutoOverlay_netautoload.dll в папку с плагинами, один и тот же файл используется как для x86, так и x64.

### Описание
Плагин предназначен для оптимального наложения одного видеоклипа на другой в автоматическом режиме.  
Сопоставление осуществляется путем тестирования различных координат наложения, разрешений, соотношений сторон и углов вращения клипа с целью найти наилучшую комбинацию этих параметров. Функция сравнения областей двух кадров - среднеквадратическое отклонение (СКО, RMSE). Результат выполнения функции обозначается как diff (т.к. тестировались и другие функции).  
Сопоставление разбивается на несколько шагов с помощью масштабирования и определения границ параметров наложения, чтобы сократить время обработки. Если на определенном шаге достигнута требуемая точность, остальные шаги не выполняются.  
После сопоставления кадров плагин позволяет адаптировать и наложить их друг на друга различными способами, получив результирующее видео, собранное из нескольких источников.  
Плагин содержит несколько фильтров, которые взаимодействуют друг с другом, чтобы разбить работу на несколько этапов.

### Подключение плагинов
    LoadPlugin("%папка с плагинами%\AvsFilterNet.dll")
    LoadNetPlugin("%папка с плагинами%\AutoOverlay.dll")
В AviSynth+ плагины загружаются автоматически.

## Фильтры
### OverlayConfig
    OverlayConfig(float minOverlayArea, float minSourceArea, float aspectRatio1, float aspectRatio2, 
                  float angle1, float angle2, int minSampleArea, int requiredSampleArea, float maxSampleDifference, 
                  int subpixel, float scaleBase, int branches, float acceptableDiff, int correction,
                  int minX, int maxX, int minY, int maxY, int minArea, int maxArea, bool fixedAspectRatio)
Фильтр описывает конфигурацию поиска оптимальных параметров наложения: устанавливает границы значений и настройки алгоритма поиска. Фильтр на выходе дает служебный клип, состоящий всего из одного фрейма, в котором закодированы переданные параметры, которые могут быть считаны другим фильтром. 

#### Параметры
- **minOverlayArea** - минимальная площадь накладываемого кадра в пересечении в процентах. По умолчанию рассчитывается таким образом, чтобы кадр основного клипа мог быть полностью вписан в накладываемый (пансканирование), но не более того.
- **minSourceArea** - минимальная площадь основного кадра в пересечении в процентах. По умолчанию рассчитывается таким образом, чтобы кадр наклдываемого клипа мог быть полностью вписан в основной (пансканирование), но не более того.
- **aspectRatio1** и **aspectRatio2** - диапазон допустимых соотношений сторон наклдываемого изображения. По умолчанию каждый из параметров равен соотношения сторон накладываемого клипа в оригинале. Могут быть заданы в любом порядке: `aspectRatio1=2.35, aspectRatio2=2.45` - то же самое, что и `aspectRatio1=2.45, aspectRatio2=2.35`.
- **angle1** и **angle2** (default 0) - диапазон угла вращения наклдываемого изображения. Могут быть заданы в любом порядке. Отрицательные значения - поворот по часовой стрелке, положительные - против.
- **minSampleArea** (default 1000) - минимальная площадь в пикселях уменьшенного кадра основного клипа во время тестирования. Чем меньше, тем быстрее, но и выше риск получить некорректный результат. Рекомендуемый диапазон: 500-3000.
- **requiredSampleArea** (default 1000) - максимально допустимая площадь в пикселях уменьшенного кадра основного клипа во время первой итерации тестирования. Чем меньше, тем быстрее, но и выше риск получить некорректный результат. Рекомендуемый диапазон: 1000-5000.
- **maxSampleDifference** (default 5) - максимально допустимое СКО между сэмплами на соседних итерациях масштабирования. Исходя из этого параметра расчитывается начальная площадь сэмпла в диапазоне *minSampleArea-requiredSampleArea*.
- **subpixel** (default 0) - субпиксельная точность наложения. 0 - наложение с точностью до пикселя, 1 - до полупикселя, 2 - до четверти и так далее. Рекомендуется значение 0, если один клип сильно уступает в качестве другому, 1-3, если клипы примерно одного качества и разрешение результирующего клипа не будет уменьшено.
- **scaleBase** (default 1.5) - основание для расчета коэффициента уменьшения изображения по формуле `coef=scaleBase^(1 - (maxStep - currentStep))`.
- **branches** - количество промежуточных "веток" тестирования при переходе от одного шага к другому. Рекомендуется 1-10. Чем выше - тем дольше.
- **acceptableDiff** (default 5) - приемлемое СКО, после получения которого последующие OverlayConfig не выполняются.
- **correction** (default 1) - величина коррекции параметров наложении между шагами масштабирования. Чем выше значение - тем больше тестируется различных вариантов наложения. 
- **minX**, **maxX**, **minY**, **maxY** - допустимый диапазон координат наложения, по умолчанию не определены.
- **minArea**, **maxArea** - минимальная и максимальная площадь накладываемого изображения, по умолчанию не определены.
- **fixedAspectRatio** - режим точного соблюдения соотношения сторон. Включается автоматически, если *aspectRatio1* и *aspectRatio2* равны.

### OverlayEngine
    OverlayEngine(clip, clip, string statFile, int backwardFrames, int forwardFrames, 
                    clip sourceMask, clip overlayMask, float maxDiff, float maxDiffIncrease, float maxDeviation, 
                    bool stabilize, clip configs, string downsize, string upsize, string rotate, bool editor)
Фильтр принимает на вход два синхронизированных по времени клипа и находит оптимальные параметры наложения, то есть минимизирует СКО между клипами в области пересечения. Эта информация записывается в выходной кадр и может быть использована другими фильтрами. Статистика может быть записана в файл для повторного использования, экономии времени и правок вручную с помощью встроенного редактора.

#### Параметры
- **source** - первый, основной клип.
- **overlay** - второй клип, накладываемый на первый. Оба клипа должны пыть либо в планарном, либо в RGB24 цветовом пространстве.
- **statFile** (default empty) - путь к файлу со статистикой. Если путь не указан, статистика хранится в оперативной памяти в пределах одного сеанса, рекомендуется для тестирования различных конфигураций наложения. После подбора оптимальной конфигурации рекомендуется указать файл со статистикой и прогнать analysis pass.
- **backwardFrames** и **forwardFrames** (default 3) - количество предыдущих и последующих кадров для анализа. 
- **sourceMask**, **overlayMask** (default empty) - маски основного и накладываемого клипа. Пиксели, имеющие значение 0 в маске исключаются из анализа в соответствующих изображениях. В RGB каналы анализируются независимо. В YUV анализируется только яркость. Y канал маски должен быть в полном диапазоне (`ColorYUV(levels="TV->PC")`). 
- **maxDiff** (default 5) - максимально допустимое СКО.
- **maxDiffIncrease** (default 1) - максимально допустимое отклонение СКО в пределах выборки.
- **maxDeviation** (default 1) - максимально допустимое различие между областями наложения, в процентах.
- **stabilize** (default true) - попытка стабилизации параметров наложения.
- **configs** (default конфигурация по умолчанию) - список конфигураций наложения в виде клипа. Задается следующим образом: `configs=OverlayConfig(subpixel=1) + OverlayConfig(angle1=-1, angle2=1)`. В этом примере, если во время тестирования первой конфигурации получен приемлемое СКО, вторая, более тяжелая с вращением изображения не выполняется. 
- **downsize** и **upsize** (default *BilinearResize*) - функции для уменьшения и увеличения размера накладываемого изображения.
- **rotate** (default *BilinearRotate*) - функция вращения накладываемого изображения.
- **editor** (default false). Если true, отображается окно визуального редактора параметров наложения по всей накопленной статистике.

#### Прицнип работы
Движок находит оптимальное пересечение двух кадров. Параметры совмещения описываются следующими свойствами: координаты верхнего угла накладываемого изображения (x,y), угол поворота (angle), ширина и высота изображения (width и height), величина обрезки изображения (crop) для субпиксельной точности (с каждой из четырех сторон изображение может быть обрезано максимум на пиксель в диапазоне вещественных чисел), СКО (diff).  

Параметры поиска задаются с помощью цепочки конфигураций *OverlayConfig* в параметре *configs*. Движок по очереди прогоняет все конфигурации, пока они не закончатся или не будет получено приемлемое значение СКО, заданное в *OverlayConfig.AcceptableDiff*. Тестирование каждой конфигурации также состоит из нескольких шагов масштабирования. На первом шаге тестируются все возможные комбинации наложения в максимально низком разрешении. На последующих шагах тестируются уточненные параметры.  
Ресайз изображений осуществляется с помощью функций, переданных в параметрах *downsize* и *upsize*. Первая в основном используется на начальных стадиях, вторая на последних, поэтому ее можно явно переопределить на более сложную, например *BicubicResize*. Сигнатура функций должны иметь следующую сигнатуру: `Resize(clip clip, int target_width, int target_height, float src_left, float src_top, float src_width, float src_height)`. Допускаются дополнительные параметры, то есть сигнатура соответствует стандартным функциям ресайза. Рекомендуется использовать плагин ResampleMT, который предоставляет те же самые функции, работающие внутри в несколько потоков.  

Помимо поиска оптимального наложения единичного кадра, движок анализирует последовательность соседних кадров. За это отвечают параметры *backwardFrames, forwardFrames, maxDiff, maxDiffIncrease, maxDeviation, stabilize*.  
Когда у движка запрашивают параметры наложения определенного кадра, происходит следующий порядок действий:
- Возврат сохраненных данных, если ранее этот кадр уже анализировался.  
- Если предыдущие кадры в количестве *backwardFrames* проанализированы, параметры наложения совпадают и их СКО не превышает *maxDiff*, такие же параметры тестируются для текущего кадра. Если максимальное отклонение СКО от среднего по выборке не превышает *maxDiffIncrease*, происходит анализ последующих фреймов, иначе текущий кадр помечается как независимый от предыдущих.  
- Далее последующие кадры тестируются с теми же параметрами наложения, если *forwardFrames* больше нуля и предыдущий кадр включается в серию предыдущих. Варианты следующие: последующие кадры также могут быть потенциально включены в эпизод на основе анализа СКО, тогда к текущему кадру однозначно применяются текущие параметры наложения. Если последующие кадры не могут быть включены в текущий эпизод, анализируется насколько близка область их оптимального наложения к текущей. Если области пересекаются на 100-*maxDeviation* процентов или более, текущий кадр рассматривается как независимый на участке пансканирования. Если области пересечения значительно не совпадают, значит последующие кадры относятся к другому эпизоду и текущий кадр зависит только от предыдущих.  
- Если текущий кадр после предыдущих действий рассматривается как независимый, то делается попытка стабилизировать параметры наложения соседних кадров, если включен параметр *stablize*.  
- Если параметр *backwardFrames* равен нулю, каждый кадр рассматривается как независимый, что резко снижает производительность и может приводить к "тряске" изображения, но обеспечивает максимальную точность отдельно взятого кадра. Рекомендуется использовать только в случае, если оба входных клипа не стабилизированы относительно друг друга.  

##### Визуальный редактор
Запускается, если *OverlayEngine.editor*=true.  
Слева превью кадра. Внизу трекбар по количеству кадров и поле ввода текущего кадра. Справа таблица, отображающиая кадры с одинаковыми параметрами наложения, объединенные в эпизоды. Между эпизодами можно переключаться. Под гридом панель управления.  
Overlay settings - параметры наложения текущего эпизода. Обрезка (crop) в тысячных пикселя в диапазоне от 0 до 1000.  
Кнопка AutoOverlay frame - повторный прогон AutoOverlay для текущего кадра, характеристики наложения распространяются на весь эпизод.  
Кнопка AutoOverlay scene - повторный прогон AutoOverlay для всех кадров, составляющих эпизод, в результате чего он может быть разбит на несколько.  
Измененные и несохраненные эпизоды подсвечиваются желтым цветом в гриде. Кнопка save - сохранение изменений. Reset - сброс изменений и повторная загрузка данных. Reload - перезагрузка характеристик для текущего кадра, распространяющиеся на весь эпизод.  
Separate - обособление кадра. Join prev - присоединить кадры предыдущего эпизода. Join next - присоединить кадры слудующего эпизода. Join to - присоединить кадры до введенного включительно.  

**Hotkeys**:
* Ctrl + S - save
* Ctrl + R - reload
* D - enable/disable difference
* P - enable/disable preview
* Ctrl + arrow keys - move overlay image
* Ctrl + add/subtract - scale overlay image
* A, Z - next/previous frame

### OverlayRender
    OverlayRender(clip, clip, clip, clip sourceMask, clip overlayMask, bool lumaOnly, int width, int height, 
                  int gradient, int noise, bool dynamicNoise, bool mode, float opacity, int colorAdjust, 
                  string matrix, string upsize, string downsize, string rotate, bool debug)
Фильтр осуществляет рендеринг результата совмещения двух клипов с определенными настройками.

#### Параметры
- **engine** - клип типа *OverlayEngine*, который предоставляет параметры наложения.
- **source** - первый, основной клип.
- **overlay** - второй клип, накладываемый на первый. Оба клипа должны пыть либо в планарном цветовом пространстве без chroma subsampling, либо в RGB24.
- **sourceMask**, **overlayMask** (default empty) - маски основного и накладываемого клипа. В отличие от OverlayEngine смысл этих масок такой же, как в обычном фильтре *Overlay*. Маски регулируют интенсивность наложения клипов относительно друга друга.
- **lumaOnly** (default false) - накладывается только канал яркости YUV при *mode*=1.
- **width** и **height** - ширина и высота выходного изображения. По умолчанию соответствует основному клипу.
- **gradient** (default 0) - длина прозрачного градинета в пикселях по краям накладываемой области. Делает переход между изображениями более плавным.
- **noise** (default 0) - длина градинета шума в пикселях по краям накладываемой области. Делает переход между изображениями более плавным.
- **dynamicNoise** (default true) - динамический шум по краям изображения от кадра к кадру, если *noise* > 0.
- **mode** (default 1) - способ совмещения изображений:  
1 - обрезка по краям основного изображения.  
2 - совмещение обоих изображений с обрезкой по краям выходного клипа.  
3 - совмещение обоих изображений без обрезки.  
4 - как 3, только с заполнением пустых углов по типу ambilight.  
5 - как 3, только с заполнением всего пустого пространства по типу ambilight.  
6 - маска режима 3. Используется для совмещения результата с еще одним клипом.  
7 - как в режиме 1, только выводится разница между изображениями (удобно для тестирования).  
- **opacity** (default 1) - степень непрозрачности накладываемого изображения от 0 до 1.
- **colorAdjust** (default 0) *экспериментально* - способ цветокоррекции:  
0 - отсутствует  
1 - накладываемый клип приводится к основному  
2 - основной клип приводится к накладываемому  
Цветокоррекция осуществляется с помощью histogram matching области пересечения. Для YUV изображений корректируется только яркость.
- **matrix** (default empty). Если параметр задан, YUV изображение конвертируется в RGB по указанной матрице для цветокоррекции.
- **downsize** и **upsize** (default *BilinearResize*) - функции для уменьшения и увеличения размера изображений.
- **rotate** (default *BilinearRotate*) - функция вращения накладываемого изображения.
- **debug** - вывод параметров наложения.

### OverlayCompare
    OverlayCompare(clip, clip, clip, string sourceText, string overlayText, int sourceColor, int overlayColor,
                   int borderSize, float opacity, int width, int height, bool info)

### ColorRangeMask
    ColorRangeMask(clip, int low, int high)
Вспомогательный фильтр, создает маску, включая в нее пиксели в диапазоне от low до high по входному клипу.

tbd

## Пример использования
    OM=AviSource("c:\test\OpenMatte.avs") # YV12 clip
    WS=AviSource("c:\test\Widescreen.avs") # YV12 clip
    config=OverlayConfig(subpixel=2) + OverlayConfig(angle1=-0.5, angle2=0.5) # rotate only if needed
    OverlayEngine(OM.ConvertToY8(), WS.ConvertToY8(), configs=config) # ConvertToY8() for better perfomance
    # return last # uncomment for analysis pass
    OverlayRender(OM.ConvertToYV24(), WS.ConvertToYV24(), debug=true, noise=50, upsize="Spline64Resize")
    ConvertToYV12() # from YV24
    # Prefetch(4) uncomment after analysis pass
