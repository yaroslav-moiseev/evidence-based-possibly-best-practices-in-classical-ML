# evidence-based-possibly-best-practices-in-classical-ML
В данном проекте решалась задача бинарной классификации клиентов, которые могут сделать вклад.

Данные для проекта были взяты из [https://archive.ics.uci.edu/ml/datasets/Bank+Marketing](https://archive.ics.uci.edu/ml/datasets/Bank+Marketing)

Здесь описываются шаги, которые были предприняты в ходе работы с данными. Выбор на каждом шаге обосновывается теоретически и/или ссылками на статьи. Также здесь приведены некоторые результаты, полученные в ходе работы с данными.

Анализ, очистка данных

На данном этапе:

Проверили наличие:

- некорректных значений,
- выбросов/экстремальных значений и редких категорий,
- дубликатов,
- пропусков.

Визуализировали распределения признаков.

![image](https://user-images.githubusercontent.com/56043980/115216226-f6f71700-a10c-11eb-9a8f-4b05ec747351.png)

Рисунок 1. Количество объектов на каждом уровне категориального признака education.

Предобработали данные, не допуская утечки данных:

- Удалили дубликаты.
- Удалили признаки, приводящие к утечке данных.
- Удалили/объединили редкие категории.
- Кодировали порядковые и бинарные признаки в числа.

Визуализировали наличие пропусков и их «корреляцию» пропусков (наличие пропусков в одних и тех же объектах.

![image](https://user-images.githubusercontent.com/56043980/115216261-00807f00-a10d-11eb-9083-a95e951c0b73.png)

Рисунок 2. Визуализация пропусков. Черные столбцы – признаки, белые линии – пропуски.

![image](https://user-images.githubusercontent.com/56043980/115216286-05ddc980-a10d-11eb-95e7-9aa023a1ede1.png)

Рисунок 3. «Корреляция» пропусков между признаками. Видно, что пропуски в ioan и housing у одних и тех же объектов.

**Кодирование категориальных признаков**

Алгоритмы на основе деревьев – случайный лес и градиентный бустинг показывают одни из лучших результатов среди всех алгоритмов машинного обучения. Какой категориальный кодировщик будет лучшим для них? Есть несколько наиболее известных кодировщиков - OneHotEncoder, LabelEncoder, TargetEncoder. Рассмотрим каждый из них.

**OneHotEncoder**

В алгоритмах, которые используют деревья решений выбирается порог, разделение по которому максимизирует прирост информации. Когда мы используем градиентный бустинг, мы ограничиваем сложность дерева, например, с помощью ограничения глубины или ограничения количества листьев. При кодировании категориальных признаков с помощью OneHotEncoder мы как бы «распределяем» прирост информации одного категориального признака между множеством фиктивных признаков. Из-за того, что разделение по фиктивным признакам приводит лишь к небольшому приросту информации, разбиение по таким признакам будет происходить редко или не будет происходить вообще. Получается, используя OneHotEncoder, особенно, при кодировании категориальных признаков с большим количеством уровней, мы будем терять информацию, не используя её. Для полноценного использования информации необходимо будет строить очень глубокие деревья, которые могут приводить к переобучению.

**LabelEncoder**

Использование LabelEncoder упорядочивает уровни категориального признака случайным образом. Если рассмотреть крайний случай, когда каждому уровню категории принадлежат только объекты 1 или 0 класса и они чередуются 1010101, а номера им причислены соответственно 1234567, понадобится сделать множество разделений по признаку для того чтобы хорошо разделить классы, разделение по такому признаку также будет давать небольшой прирост информации.

**TargetEncoder** упорядочивает уровни категориального признака по пропорции объектов 1 класса среди всех объектов уровня, приводя к утечке данных.

**CatboostEncoder** использует регуляризацию, основанную на понятии времени для расчета целевого значения для того, чтобы избежать утечки данных. CatboostEncoder оказался лучшим кодировщиком при использовании однократной вложенной кросс-валидации среди 11 кодировщиков при тестировании на 12 датасетах с LGBMClassifier**[1]**.

**GLMMEncoder** , как и CatboostEncoder является целевым кодировщиком, использует автоматически определяемый параметр регуляризации. GLMMEncoder с вложенной кросс-валидацией оказался наиболее эффективным кодировщиком по результатам тестирования на 27 датасетах с категориальными признаками с большим количеством уровней 5 алгоритмов машинного обучения среди 9 типов кодировщиков (исследование не включало CatboostEncoder) **[2]**.

Эксперимент показал, что на данном датасете при использовании CatBoostEncoder и GLMMEncoder с однакратной вложенной кросс-валидацией averegeprecision и rocauc выше, чем без неё, а CatBoostEncoder лучше GLMMEncoder.

**Обработка пропущенных значений**

Тип пропущенных данных MCAR (отсутствуют совершенно случайно) редко встречается. Пропущенные данные обычно относятся в определенной пропорции к MAR (отсутствуют случайно), и MNAR (отсутствуют неслучайно). При отсутствии данных менее 10% заполнение с помощью множественного вменения хорошая идея вне зависимости от причины отсутствия данных, даже если данные на 100% MNAR. MissForest показывает хорошие результаты, вменяя данные, пропущенные по разным причинам **[3,4]**.

Альтернативой **MissForest** может быть **KNNimputer** , однако он чувствителен к масштабированию, мультиколлинеарности, выбросам, шуму, неинформативным признакам, долго вычисляется при работе с большими датасетами, для хорошей точности предсказаний необходима настройка гиперпараметров, в отличие от MissForest. Существуют методы заполнения пропусков на основе нейросетей, например, **Datawig** , однако MissForest работает быстрее. Кроме того, существует несколько новых методов, которые на тестах показывают хорошую метрику при вменении смешанных данных, однако они не имеют реализации на python в открытом доступе.

Исходя из вышесказанного мы выбрали Missforest. К сожалению, заполнение с помощью Missforest происходит не очень быстро, поэтому мы сократили количество используемых деревьев. Также в документации к пакету Missforest на R, написано, что количество деревьев может быть выбрано довольно маленьким поскольку значения для всех пропусков предсказываются несколько раз. Для реальных задач при желании можно увеличить количество деревьев.

**Детекция выбросов**

Мы использовали **IsolationForest** – лучший/один из лучших алгоритмов для детекции выбросов в многомерном пространстве по результатам сравнения алгоритмов детекции выбросов на большом количестве датасетов **[5, 6, 7].**

Для лучшего понимания наших выбросов была визуализирована структура дерева решений, которое было обучено для классификации объектов на выбросы и не выбросы, на основе меток, присвоенных IsolationForest.

![image](https://user-images.githubusercontent.com/56043980/115216321-10985e80-a10d-11eb-80c8-031098c1c8f5.png)

Рисунок 4. Визуализация дерева решений, построенного для классификации выбросов.

Для расчёта более точных значений корреляций между признаками из датасета, использованного для разведочного анализа данных, были удалены выбросы. Порог отнесения к выбросам был выбран вручную, на основании графика оценок принадлежности объектов к выбросам, данные для которого были получены с помощью IsolationForest, и здравого смысла. Порог был выбран как -0.10.

![image](https://user-images.githubusercontent.com/56043980/115216347-15f5a900-a10d-11eb-9ce8-85053e30195f.png)

Рисунок 5. График счёта, который отражает количество разбиений необходимых для того, чтобы изолировать объект от других объектов. Чем меньше счет, тем меньше разбиений необходимо и объект вероятнее является выбросом.

**Масштабирование признаков**

Как известно, при использовании метрических алгоритмов и алгоритмов, использующих градиентный спуск необходимо масштабировать признаки. Однако, для масштабирования порядковых и бинарных признаков не следует использовать статистики, используемые для масштабирования признаков в количественной шкале. Используя порядковую шкалу, мы можем только сказать у какого объекта значение признака больше/меньше, но не на сколько или во сколько раз больше/меньше. Когда признаки находятся в номинальной шкале (включая бинарную) мы можем сравнить значения только на равенство и неравенство.

Наша задача при масштабировании признаков для метрических алгоритмов заключается в том, чтобы сделать так, чтобы признаки имели одинаковую важность, учитывая то, что существуют скошенные распределения и выбросы, её можно переформулировать следующим образом – большинство значений признаков должно находиться в одном интервале. В идеале хотелось бы максимизировать площадь перекрытия всех распределений между собой. Центрирование в общем способствует максимизации площади перекрытия.

Для масштабирования количественных признаков и кодированных номинальных признаков использовали **RobustScaler** , так как он использует статистики, устойчивые к выбросам – медиану и межквартальный размах.

Порядковые и бинарные признаки масштабировали на интервал, на котором находилось большинство значений количественных и номинальных признаков после масштабирования. Такое масштабирование аналогично подходу, предложенному в статье, где количественные признаки переносят на интервал близкий к 0-1, на котором находятся бинарные признаки **[8]**.

RobustScaler использует **IQR** для масштабирования. Масштабирование делением на IQR или MAD устойчиво к выбросам, однако имеет низкую эффективность при масштабировании распределений без выбросов (например нормального) - 37% **[9, 10]**. Были предложены варианты меры разброса устойчивые к выбросам, учитывающие асимметрию распределений, а также более эффективные для распределений без выбросов (например, нормального) - **Sn** (58%) **Qn** (82%). Они реализованы в библиотеке [Robustbase](https://pypi.org/project/robustbase/0.2.1/), однако расчёт их значений происходит очень долго на данном датасете. Расчёт Qn был сильно ускорен с помощью numba, также для масштабатора с использованием Qn и медианы был написан класс, однако у numba оказалось не все так просто с ускорением классов. Сравнение распределений признаков после масштабирования с Qn и с помощью RobastScaler, показало, что относительно цели - большинство значений признаков должно находиться на одном интервале (иметь одинаковую важность), RobustScaler почти для всех признаков лучше справился с выполнением данной задачи.

**Визуализация данных в двухмерном пространстве**

**Umap** работает быстрее, чем **t** - **sne** и, как пишут, лучше сохраняет расстояния, поэтому использовали umap. На рисунках ниже класс 2 это выбросы, которые отметил IsolationForest, при выбранном нами пороге.

![image](https://user-images.githubusercontent.com/56043980/115216395-20b03e00-a10d-11eb-833b-01218a9a5adc.png)

Рисунок 6. Визуализация датасета в двухмерном пространстве без выделения выбросов. 0 и 1, отрицательный и положительный классы соответственно. 2 – выбросы.

![image](https://user-images.githubusercontent.com/56043980/115216413-2574f200-a10d-11eb-82e7-a1936719aa54.png)

Рисунок 7. Визуализация датасета в двухмерном пространстве с выделением выбросов. 0 и 1, отрицательный и положительный классы соответственно. 2 – выбросы.

Для улучшения визуализации данных попробовали использовать **метрику махалонобиса** , которая учитывает корреляцию признаков при расчёте расстояний. При этом ковариация для бинарных и порядковых признаков, для которых статистики количественных признаков не имеют смысла, была заменена на ноль. Так как дисперсия чувствительна к выбросам и мы отдельно уже масштабировали признаки, дисперсия признаков в матрице ковариаций была заменена для всех признаков на единичную. Визуализация с использованием метрики махалонобиса и измененной матрицей ковариаций субъективна не стала лучше.

**Оценка корреляции**

Почему имеет смысл смотреть корреляцию признаков? Корреляция между признаками мешает правильно оценивать важность признаков с помощью моделей (в том числе RF и GB). Кроме того, мультиколлениарность может приводить к переобучению линейных моделей и влиять на качество предсказаний случайного леса (независимость базовых алгоритмов), однако настраивая регуляризацию и количество используемых признаков, этого можно избежать. Так как у нас дисбаланс классов, мы будем использовать SMOTE. Варианты SMOTE, основанные на KNN, как и KNN чувствительны к мультиколлинераности, поэтому желательно избавиться от сильно коррелирующих признаков или же использовать метрику махалонобиса. Использование метрики махалонобиса аналогично umap не дало желаемых результатов.

Для оценки попарной корреляции была выбрана **корреляция Спирмена** , а не Пирсона, потому что она более устойчива к выбросам, показывает монотонные, а не только линейные зависимости, подходит для оценки корреляции между количественными и порядковыми признаками (как и корреляция Кендалла). Низкая попарная корреляция одной переменной с другими не говорит об отсутствии коллинераности. Возможна ситуация, когда одна переменная сильно коррелирует с группой переменных, однако попарная корреляция с каждой из них слабая. Для оценки такого типа корреляции был использован **VIF**.

![](RackMultipart20210419-4-1m38ph2_html_5f078cd596927bb8.png)

Рисунок 8. Корреляция Спирмена.

![image](https://user-images.githubusercontent.com/56043980/115216462-31f94a80-a10d-11eb-859d-4b3702dd5816.png)

Рисунок 9. Значения фактора инфляции дисперсии для количественных и порядковых признаков. Признаки, имеющие значения VIF больше 5-10 являются сильно коррелирующими.

Оценка «корреляции» между количественными и категориальными признаками проводилась с помощью **ANOVA** на основе **коэффициента eta** , который является мерой силы связи между категориальными и количественными признаками. ANOVA имеет некоторые предположения, например, насчёт гомоскедастичности остатков. При относительно больших размерах выборки (как у нас) корректно использовать ANOVA, несмотря на гетероскедастичность. Однако, наиболее точные оценки можно получить, используя оценку стандартных ошибок в форме Уайта. Нет необходимости проводить тест на гетероскедастичность, если использовать по умолчанию устойчивый к гетероскедастичности вариант ANOVA**[11]**.

![image](https://user-images.githubusercontent.com/56043980/115216607-548b6380-a10d-11eb-8e08-eb6991f6e376.png)

Рисунок 10. «Корреляция» (коэффициент eta) между количественными (подписи слева) и категориальными признаками (подписи снизу).

«Корреляции» между категориальными признаками оценивалась с помощью **коэффициента неопределенности** (Thiel&#39;sUcorrelationcoefficient ( **Uncertainty** **Coefficient**)), а не коэффициента Крамера, так как при его использовании мы не теряем информацию из-за симметрии**[12,13]**.

![image](https://user-images.githubusercontent.com/56043980/115216625-59501780-a10d-11eb-92ed-4e0650dd7618.png)

Рисунок 11. «Корреляция» (коэффициент неопределенности) между категориальными признаками

Для более точного расчёта важности признаков были оставлены только по одному признаку из каждой из групп коррелирующих признаков. Отбор признаков был основан значении averageprecision для чувствительных к корреляции алгоритмов – метода ближайших соседей, машины опорных векторов с ядром RBF, линейной регрессии. Стоит отметить, что для всех трёх алгоритмов отобранные признаки оказались одинаковыми.

**Оценка важности признаков**

Удаление неважных признаков полезно для сокращения времени обучения моделей и может уменьшить переобучение. Из методов отбора признаков выбор был сделан в пользу встроенных методов, так как они учитываю взаимодействия между признаками, в отличие от методов фильтрации и менее затраты по времени, в отличие от методов оболочки. Для отбора признаков был выбран алгоритм Boruta, так как он позволяет более надежно оценить важность признаков благодаря использованию перестановок **[14]** и статистически обосновано выбрать порог, на основании которого важные признаки отделяются от неважных в отличие от регрессии, случайного леса, градиентного бустинга.

Кроме того, в рамках тестирования на 311 датасетах 12 алгоритмов для отбора признаков было установлено, что Boruta является одним из лучших алгоритмов, имеющих низкие временные затраты **[15]**.

В качестве метрики важности функций были выбраны значения Шепли ( **SHAP** ), которые позволяют количественно оценить вклад каждого признака в прогноз. Несмотря на то, что при расчёте значений Шепли должна учитываться корреляция признаков, аппроксимация значений, которая используется на практике, предполагает, что признаки не коррелируют между особой **[16]**.

![image](https://user-images.githubusercontent.com/56043980/115216639-5ead6200-a10d-11eb-8a54-7ef12c7937b9.png)

Рисунок 12. Визуализация важности признаков, полученная с помощью алгоритма Leshy.

**Работа с несбалансированными классами**

В алгоритмах машинного обучения обычно используются функции потерь, которые одинаково штрафуют за ошибку на объекте отрицательного или положительного класса. Если классы плохо разделимы, то алгоритму может быть выгодней, с точки зрения потерь, так построить разделяющую гиперплоскость, чтобы правильно классифицировать объекты того класса, которых больше и пренебречь объектами класса меньшинства.

В качестве метрики классификации была выбрана **average precision** ( **Precision**** Recall ****AUC** ), так как она не зависит от определенного порога и более чувствительна, чем **ROC**** AUC**, в нашем случае, когда классом меньшинства является положительный класс. При небольшом положительном классе количество ложноположительных ошибок (FP) относительно истинно отрицательных будет (TN) будет небольшим и при изменении количества FPFPR будет слабо изменяться, тогда как из-за небольшого количества истинно положительных TPPrecision будет сильно измениться при изменении количества FP**[17]**.

Для решения проблемы дисбаланса классов были использованы 2 стратегии – добавления веса классу меньшинства и SMOTE.

Выбор вариантов SMOTE для различных алгоритмов основывался на статье, где сравнили эффективность 85 вариантов SMOTE на 104 датасетах при использовании 4 моделей машинного обучения **[18]**.

**Удаление коррелирующих признаков**

Всего было обнаружено 6 сильно коррелирующих признаков, одна группа содержала 4 коррелирующих между собой признака, другая – 2. Предварительный отбор признаков показал, что для всех алгоритмов, кроме логистической регрессии удаление хотя бы одного коррелирующего признака приводило к увеличению average precision, в том числе для случайного леса и 3 вариантов бустинга. В результате сравнения получилась таблица ниже. Минимальным назывался такой набор, в котором в каждой группе коррелирующих признаков осталось по 1 признаку.

Таблица 1. Усредненные значения averageprecision, полученные в результате пятикратной кросс-валидации при обучении различных алгоритмов машинного обучения на различных наборах признаков (с разным количеством коррелирующих признаков)

|   |   | average precision для наборов признаков |
| --- | --- | --- |
| Алгоритмы | лучший набор получен удалением следующего количества корр. признаков | лучший | лучший из минимальных | полный |
| Log\_reg C=1 | 0 | 0,4294 | 0,4257 | - |
| Log\_reg C=100 | 1 | 0,4293 | 0,4255 | 0,42926 |
| SVC c rbf | 4 | 0,3957 | - | 0,3919 |
| SVC с linear | 3 | 0,3645 | 0,3561 | 0,2691 |
| KNN | 2 | 0,3146 | 0,3129 | 0,299 |
| LGBM | 2 | 0,4582 | 0,4557 | 0,4553 |
| XGB | 1 | 0,4577 | 0,4541 | 0,4571 |
| CatBoost | 2 | 0,4524 | 0,4486 | 0,4496 |
| RF | 1 | 0,4315 | 0,43 | 0,4305 |

**Удаление неважных признаков**

Из полученных на предыдущем этапе лучших наборов признаков удаляли неважные признаки и смотрели на изменение averageprecision для KNN, SVC, Log\_reg. Для SVC лучшим набором был набор без 1 неважного признака (AP= 0,3976), по сравнению с полным (AP = 0,3957), а для KNN, Log\_reg лучшими оказались полные наборы.

**Удаление выбросов**

Так как, при детекции выбросов к классу меньшинства относилось не менее 50% и была гипотеза, что удаление объектов класса меньшинства может ухудшить метрику, была написана функция, которая удаляла выбросы только из класса большинства. При детекции 8% выбросов, удаление из предсказанных выбросов только выбросов, относящихся к классу большинства улучшило метрику для KNN (AP = 0,3180), без удаления (AP = 0,2989). Удаление выбросов для SVC, Log\_reg только ухудшало метрику.

**Выбор**  **SMOTE**

Применение SMOTE улучшило метрику для SVC со SMOTE\_ENN (AP = 0,4125), без SMOTE (AP = 0,3918). KNN с Supervised\_SMOTE (AP = 0,3151) ибез SMOTE (AP = 0,2990) инезначительнодля CatBoost (polynom\_fit\_SMOTE). Применение SMOTE с небольшим тюнингом в другом эксперименте улучшило метрику для LightGBM (LVQ\_SMOTE), изменение метрики с небольшим тюнингом SMOTE неизвестно для XGBoost и RF. Уверенности, что использование SMOTE с XGBoost и RF было правильным решением, нет по предварительному тюнингу. Однако, при отрицательном влиянии на метрику применения SMOTE, параметр proportion мог принять минимальное значение, что означало бы отсутствие SMOTE.

**Вложенный тюнинг гиперпараметров**

Для выбора лучшего варианта SMOTE идеальным вариантом было не просто менять варианты SMOTE со стандартными значениями параметров, а менять варианты и тюнить их. Такой вложенный тюнинг гиперпараметров можно осуществить в sklearn, однако подбор гиперпараметров осуществляется только перебором. Я пробовал найти/реализовать вариант, когда вложенный тюнинг сочетается с байсовской оптимизацией и TPE.

TuneSearchCV позволило осуществить вложенный тюнинг только в сочетании с RandomSearch.

Несмотря на совместимость OptunaSearchCV с sklearn, она не позволяет передавать параметры в виде списка словарей для вложенного тюнинга гиперпараметров и выдает ошибку.

Обычный вариант Optuna не работает вместе с MissForest.

Optuna позволяет легко визуализировать гиперпараметры

**Выбор лучшего алгоритма**

Хорошим вариантом для сравнения алгоритмов является тест Макнемара, однако, используемый датасет недостаточно велик, а на тест изначально было выделено только 15%, поэтому не следует выбирать лучший алгоритм на основании теста Макнемара. Для небольших датасетов рекомендуется использовать вложенную кросс-валидиацию или 5x2cvf-тест **[19]**.

![image](https://user-images.githubusercontent.com/56043980/115216684-6967f700-a10d-11eb-9d11-c7cdcffe7aa3.png)

Рисунок 13. Precison- recall кривые, построенные на основании метрик, рассчитанных на основании предсказаний различных алгоритмов машинного обучения на тестовых данных. AP - average precision.

![image](https://user-images.githubusercontent.com/56043980/115216704-6ec54180-a10d-11eb-99a4-e572c2c37e9b.png)

Рисунок 14. Диаграммы размаха значений averageprecision, полученных в результате повторной (n\_repeats = 5) стратифицированной 5-блочной кросс-валидации с различными значениями randomstate при каждом повторном разбиении данных (RepeatedStratifiedKFold) для различных алгоритмов машинного обучения (всего 25 точек для каждого алгоритма).

![image](https://user-images.githubusercontent.com/56043980/115216963-ad5afc00-a10d-11eb-8641-6d7e98150653.png)

Рисунок 15. Среднее значение и стандартное отклонение averageprecision, рассчитанное для каждого алгоритма на основании RepeatedStratifiedKFold (25 точек).

Вложенная кросс-валидация затратна по времени и её следовало бы использовать вместе с тюнингом гиперпараметров. С применением 5x2cvf-тест возникли проблемы с использованием с моими пайплайнами.

Улучшению метрики averageprecision, возможно, поспособствовала бы генерация признаков и применение PCA, LDA.

**Список литературы**

1. [https://towardsdatascience.com/benchmarking-categorical-encoders-9c322bd77ee8](https://towardsdatascience.com/benchmarking-categorical-encoders-9c322bd77ee8)
2. A Benchmark Experiment on How to Encode Categorical Features in Predictive Modeling
3. [https://www.czasopisma.uni.lodz.pl/foe/article/view/2584](https://www.czasopisma.uni.lodz.pl/foe/article/view/2584)
4. [https://bmcmedresmethodol.biomedcentral.com/articles/10.1186/s12874-020-01080-1](https://bmcmedresmethodol.biomedcentral.com/articles/10.1186/s12874-020-01080-1)
5. [https://arxiv.org/abs/1503.01158](https://arxiv.org/abs/1503.01158)
6. [https://www.springer.com/gp/book/9783319547640](https://www.springer.com/gp/book/9783319547640)
7. [https://towardsdatascience.com/isolation-forest-is-the-best-anomaly-detection-algorithm-for-big-data-right-now-e1a18ec0f94f](https://towardsdatascience.com/isolation-forest-is-the-best-anomaly-detection-algorithm-for-big-data-right-now-e1a18ec0f94f)
8. [https://onlinelibrary.wiley.com/doi/10.1002/sim.3107](https://onlinelibrary.wiley.com/doi/10.1002/sim.3107)
9. https://www.tandfonline.com/doi/abs/10.1080/01621459.1993.10476408
10. [https://github.com/scikit-learn/scikit-learn/issues/10139](https://github.com/scikit-learn/scikit-learn/issues/10139)
11. [https://www.tandfonline.com/doi/abs/10.1080/00031305.2000.10474549](https://www.tandfonline.com/doi/abs/10.1080/00031305.2000.10474549)
12. [https://towardsdatascience.com/the-search-for-categorical-correlation-a1cf7f1888c9](https://towardsdatascience.com/the-search-for-categorical-correlation-a1cf7f1888c9)
13. [https://rstudio-pubs-static.s3.amazonaws.com/558925\_38b86f0530c9480fad4d029a4e4aea68.html](https://rstudio-pubs-static.s3.amazonaws.com/558925_38b86f0530c9480fad4d029a4e4aea68.html)
14. [https://explained.ai/rf-importance/index.html#6.2](https://explained.ai/rf-importance/index.html#6.2)
15. [https://www.sciencedirect.com/science/article/abs/pii/S0957417419303574](https://www.sciencedirect.com/science/article/abs/pii/S0957417419303574)
16. [https://arxiv.org/abs/1705.07874](https://arxiv.org/abs/1705.07874)
17. [https://www.nature.com/articles/nmeth.3945](https://www.nature.com/articles/nmeth.3945)
18. https://www.sciencedirect.com/science/article/abs/pii/S1568494619304429
19. [https://arxiv.org/pdf/1811.12808.pdf](https://arxiv.org/pdf/1811.12808.pdf)
