# Параллельные вычисления: алгоритмы и анализ производительности

Две учебные работы на C++: умножение матриц алгоритмом Штрассена и построение выпуклых оболочек компонент бинарного изображения. В каждой работе есть последовательная реализация, параллельные варианты, функциональные тесты и отчёты с интерпретацией измерений.

Цель — исследовать, когда распараллеливание помогает, какие накладные расходы ограничивают ускорение и насколько выводы поддерживаются экспериментом.

> Это подборка двух собственных задач из общих репозиториев курса Parallel Programming Course. Инфраструктура курса и работы других студентов сюда не включены. Задачи зависят от исходной инфраструктуры: этот репозиторий не собирается самостоятельно.

## Что внутри

### Умножение плотных матриц: Штрассен и MPI

Каталог: [`tasks/kamalagin_a_mat_mult_strassen`](tasks/kamalagin_a_mat_mult_strassen). [Отчёт](tasks/kamalagin_a_mat_mult_strassen/report.md).

Даны квадратные матрицы `double`; требуется вычислить их произведение. Алгоритм использует семь произведений подматриц вместо восьми, дополнение размера до степени двойки и наивное умножение для малых блоков. Вычисления организованы через явный стек вместо рекурсивных вызовов.

MPI-версия распределяет произведения верхнего уровня между процессами, собирает результаты и формирует итоговую матрицу. Последовательная версия служит базовой реализацией.

Навигация: [`seq`](tasks/kamalagin_a_mat_mult_strassen/seq), [`mpi`](tasks/kamalagin_a_mat_mult_strassen/mpi), [функциональные тесты](tasks/kamalagin_a_mat_mult_strassen/tests/functional/main.cpp), [измерение времени](tasks/kamalagin_a_mat_mult_strassen/tests/performance/main.cpp).

**Ограничение текущих измерений.** В исходном отчёте есть таблицы ускорения до 2,82× в `task_run` и 3,55× в `pipeline`. Однако сохранённый performance-тест использует `N=32`, а порог перехода к Штрассену — 64. При таком размере MPI-версия обходит распределение произведений. Следовательно, опубликованные таблицы нельзя считать воспроизводимым подтверждением ускорения текущей конфигурации теста.

Исходники и исторический отчёт сохранены без исправления результатов. Следующий шаг — отдельная серия замеров для размеров выше порога, фиксированные входы, несколько повторений и фиксация разброса. Функциональные MPI-тесты включают размер 65, последовательные — до 33; это также следует учитывать при оценке покрытия.

### Выпуклые оболочки компонент изображения

Каталог: [`tasks/kamalagin_a_binary_image_convex_hull`](tasks/kamalagin_a_binary_image_convex_hull). [Общий отчёт](tasks/kamalagin_a_binary_image_convex_hull/report.md).

Для бинарного изображения сначала находятся 4-связные компоненты, затем для каждой строится выпуклая оболочка алгоритмом Грэхема. Версии OpenMP, TBB и `std::thread` параллельно обрабатывают компоненты. Гибридная версия ALL распределяет компоненты через MPI и обрабатывает их с OpenMP внутри каждого процесса.

Навигация: [`seq`](tasks/kamalagin_a_binary_image_convex_hull/seq), [`omp`](tasks/kamalagin_a_binary_image_convex_hull/omp), [`tbb`](tasks/kamalagin_a_binary_image_convex_hull/tbb), [`stl`](tasks/kamalagin_a_binary_image_convex_hull/stl), [`all`](tasks/kamalagin_a_binary_image_convex_hull/all), [функциональные тесты](tasks/kamalagin_a_binary_image_convex_hull/tests/functional/main.cpp), [измерение времени](tasks/kamalagin_a_binary_image_convex_hull/tests/performance/main.cpp).

**Основной результат из отчёта:** на разреженном изображении 5000×5000 значимого ускорения не получено. Последовательное время — 0,0371 с, лучшие приведённые времена потоковых версий — около 0,0369 с. Поиск компонент остаётся последовательным, а обработка малых компонент не компенсирует расходы параллельного выполнения.

Замеры относятся к одной машине; отдельные прогревы и статистическая неопределённость не зафиксированы. В таблицах эффективности ALL знаменатель учитывает только число OpenMP-потоков, хотя используется два MPI-процесса. Поэтому этот показатель нельзя сравнивать с эффективностью относительно общего числа работников; для последней понадобился бы знаменатель `2 × число потоков`. Исторические таблицы оставлены без изменений, их ограничение зафиксировано здесь.

## Структура задач

В каждой задаче находятся:

- `common/include` — типы входов и результатов;
- каталоги реализаций с `include` и `src`;
- `tests/functional` и `tests/performance` — тесты для общих runner-ов курса;
- `settings.json` — включённые реализации;
- `info.json` — исходные сведения об учебной работе;
- `report.md` — постановка, описание реализации, измерения и выводы.

Сохранённые отчёты — материалы учебных работ. При подготовке публикации программы не запускались; наличие тестов и записанного результата не означает новую проверку сборки или CI.

## Как собрать и запустить

Нужны Git, CMake 3.25+, компилятор с поддержкой C++23, MPI (например, MS-MPI на Windows), OpenMP и инструменты сборки. Зависимости GoogleTest, oneTBB и другие подключаются инфраструктурой курса; при первой сборке может понадобиться сеть. Подробности окружения: [документация курса](https://learning-process.github.io/parallel_programming_course/ru/).

Ниже приведён сценарий PowerShell. Выполните его **из корня этого репозитория**. Он создаёт две отдельные рабочие копии в `work/`, закрепляет известные ревизии авторских forks и копирует в них сохранённые задачи. В выборочной рабочей копии остаётся инфраструктура и только нужная задача; файлы существующих учебных проектов не затрагиваются.

### Подготовка окружений

```powershell
$portfolioRoot = (Get-Location).Path
New-Item -ItemType Directory -Path work -Force | Out-Null

git clone --no-checkout https://github.com/Lynxll/ppc-2025-processes-engineers.git work/ppc-processes
git -C work/ppc-processes sparse-checkout init --no-cone
git -C work/ppc-processes sparse-checkout set '/*' '!/tasks/*/' '/tasks/common/' '/tasks/kamalagin_a_mat_mult_strassen/'
git -C work/ppc-processes checkout f04844ec50f826a9d616f25bbfe2be33914c4e48
git -C work/ppc-processes submodule update --init --recursive
Copy-Item -LiteralPath tasks/kamalagin_a_mat_mult_strassen -Destination work/ppc-processes/tasks -Recurse -Force

git clone --no-checkout https://github.com/Lynxll/ppc-2026-threads.git work/ppc-threads
git -C work/ppc-threads sparse-checkout init --no-cone
git -C work/ppc-threads sparse-checkout set '/*' '!/tasks/*/' '/tasks/common/' '/tasks/kamalagin_a_binary_image_convex_hull/'
git -C work/ppc-threads checkout 3c7741551b764b3469ad5795b591f7660625b83f
git -C work/ppc-threads submodule update --init --recursive
Copy-Item -LiteralPath tasks/kamalagin_a_binary_image_convex_hull -Destination work/ppc-threads/tasks -Recurse -Force
```

Команды рассчитаны на первый запуск, когда указанных рабочих копий ещё нет. Ревизии и upstream-репозитории перечислены в [SOURCES.md](SOURCES.md). Наличие обеих закреплённых ревизий в публичных авторских forks подтверждено 8 ноября 2026 года: каждый commit успешно получен через `git fetch --depth 1`. Сборка и тесты при этой проверке не запускались.

### Сборка

```powershell
cmake -S work/ppc-processes -B work/ppc-processes/build -D USE_FUNC_TESTS=ON -D USE_PERF_TESTS=ON -D CMAKE_BUILD_TYPE=Release
cmake --build work/ppc-processes/build --config Release --parallel

cmake -S work/ppc-threads -B work/ppc-threads/build -D USE_FUNC_TESTS=ON -D USE_PERF_TESTS=ON -D CMAKE_BUILD_TYPE=Release
cmake --build work/ppc-threads/build --config Release --parallel
```

Имена `ppc_func_tests` и `ppc_perf_tests`, каталог `build/bin` и параметры сборки взяты из сохранённых конфигураций курса. При публикации этот сценарий не выполнялся.

### Функциональные проверки

```powershell
$env:PPC_NUM_THREADS = '4'
$env:PPC_NUM_PROC = '1'
mpiexec -n 1 ./work/ppc-processes/build/bin/ppc_func_tests.exe --gtest_filter='*kamalagin_a_mat_mult_strassen_seq*'

$env:PPC_NUM_PROC = '2'
mpiexec -n 2 ./work/ppc-processes/build/bin/ppc_func_tests.exe --gtest_filter='*kamalagin_a_mat_mult_strassen_mpi*'

$env:PPC_NUM_PROC = '1'
./work/ppc-threads/build/bin/ppc_func_tests.exe --gtest_filter='*kamalagin_a_binary_image_convex_hull*:-*kamalagin_a_binary_image_convex_hull_all*'

$env:PPC_NUM_PROC = '2'
mpiexec -n 2 -env PPC_NUM_THREADS 4 -env OMP_NUM_THREADS 4 ./work/ppc-threads/build/bin/ppc_func_tests.exe --gtest_filter='*kamalagin_a_binary_image_convex_hull_all*'
```

Параметр `-env` в последней команде соответствует MS-MPI. Для другой реализации MPI используйте её способ передачи переменных окружения. На Linux исполняемые файлы обычно не имеют суффикса `.exe`.

Для просмотра доступных имён тестов можно запустить нужную программу с `--gtest_list_tests`. Для измерений используется `ppc_perf_tests` с теми же фильтрами задачи и согласованным количеством процессов; SEQ нужно измерять с одним MPI-процессом. Сначала учитывайте описанные выше ограничения входов и методики.

## Авторство и лицензия

Сюда выделены только задачи `kamalagin_a_mat_mult_strassen` и `kamalagin_a_binary_image_convex_hull`. Исходные сведения об авторе и ссылки в отчётах сохранены. Общий тестовый framework, runner-ы и сборочная система разработаны участниками курса Learning Process и используются как внешняя инфраструктура.

Сохранена исходная [BSD 3-Clause License](LICENSE) Learning Process developers. Происхождение, ревизии и особенности локального снимка описаны в [SOURCES.md](SOURCES.md).
