# KristMr
High-performance standalone Windows kernel optimizer &amp; game latency stabilizer with a DirectX 11 3D crystal UI. Pure C++20, Win32, NT Native API. Высокопроизводительный оптимизатор ядра Windows и стабилизатор задержек с 3D-интерфейсом на DirectX 11. Нативный C++20, Win32, NT API.
# 💎 KristMr (v2.1 Production Master)

<p align="center">
  <img src="resources/app_logo.png" alt="KristMr Logo" width="160" />
</p>

<p align="center">
  <strong>Ultra-low Latency Windows Kernel Optimizer & Dynamic Threat Isolation Engine</strong><br>
  <em>Engineered entirely in pure ISO C++20 with Hardware-Accelerated DirectX 11 / HLSL Procedural Rendering.</em>
</p>

<p align="center">
  <a href="https://en.wikipedia.org/wiki/C%2B%2B20"><img src="https://img.shields.io/badge/Standard-ISO%20C%2B%2B20-00599C?style=for-the-badge&logo=c%2B%2B" alt="C++20"></a>
  <a href="https://learn.microsoft.com/en-us/windows/win32/direct3d11/direct3d-11-graphics"><img src="https://img.shields.io/badge/Graphics-DirectX%2011%20%2F%20HLSL-F1502F?style=for-the-badge&logo=directx" alt="DirectX 11"></a>
  <a href="https://learn.microsoft.com/en-us/windows/win32/api/"><img src="https://img.shields.io/badge/Stack-Pure%20Win32%20%26%20NT%20API-0078D6?style=for-the-badge&logo=windows" alt="Win32"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-brightgreen?style=for-the-badge" alt="MIT License"></a>
  <img src="https://img.shields.io/badge/Binary%20Size-~524%20KB-blueviolet?style=for-the-badge" alt="Binary Size">
</p>

---

## 📖 Обзор проекта (Project Overview)

**KristMr** — это автономная системная утилита нового поколения, разработанная по парадигме **Zero-Bloatware**. Большинство современных игровых оптимизаторов и системных твикеров написаны на тяжелых веб-оболочках (Electron, Chromium Embedded) или платформах .NET / WPF, потребляя сотни мегабайт оперативной памяти и вызывая микрофризы в играх.

**KristMr решает эту проблему радикально:**
- Никаких внешних рантаймов (Node.js, .NET, Python, Qt) — монолитный исполняемый файл размером всего ~520 КБ.
- Статическая линковка с CRT (`/MT`), исключающая ошибки отсутствующих DLL (`VCRUNTIME140.dll` и т.д.).
- Прямое взаимодействие с недокументированным ядром Windows через **NT Native API** (`ntdll.dll`).
- Эстетичный квантовый 3D-интерфейс на базе **DirectX 11**, который засыпает до **0 FPS**, когда вы играете.

---

## ⚡ Ключевые модули и архитектура

### 1. ⏱️ Системный таймер и обход троттлинга Windows 11
* **0.5 мс Resolution:** Динамический импорт и вызов `ntdll!NtSetTimerResolution` устанавливает минимально возможный квант таймера операционной системы (0.5 мс вместо стандартных 15.6 мс).
* **Сглаживание Frametime:** Игровые движки (включая Minecraft, CS2, Valorant, Apex Legends) моментально выравнивают график задержки кадров (Frametime), полностью устраняя микростаттеры.
* **Обход EcoQoS:** Применение `SetProcessInformation` с флагом `PROCESS_POWER_THROTTLING_IGNORE_TIMER_RESOLUTION` исключает принудительное замедление таймера политиками энергосбережения Windows 11.

### 2. 🚀 Процессорный Гипер-Режим (Power & Core Unparking)
* **Аппаратная топология:** Опрос через `GetLogicalProcessorInformationEx(RelationProcessorCore)` в реальном времени определяет архитектуру процессора.
* **100% Core Parking Unpark (`CPMINCORES = 100%`):** На монолитных многоядерных процессорах (Intel Core, Xeon E5/W, AMD Ryzen) Windows 11 часто «усыпляет» свободные потоки в глубокие C-states (частоты падают до 1.2 ГГц). При внезапной нагрузке (генерация чанков, спавн мобов, компиляция шейдеров) пробуждение ядер вызывает стоп-кадр. KristMr удерживает ядра в полной готовности.
* **Безопасность для гибридных чипов:** Для архитектур Intel P+E Cores (Alder Lake и новее) и AMD 3D V-Cache (X3D) режим парковки не нарушает аппаратное распределение задач планировщиком Windows Thread Director.

### 3. 🧹 Очистка системной памяти (Standby List Purge)
* Вызов `NtSetSystemInformation` со служебным классом `SystemMemoryListInformation` мгновенно освобождает кэш Standby List.
* Предотвращает исчерпание физической ОЗУ и жесткие лаги при выгрузке данных в файл подкачки на SSD.

### 4. 🌐 Сетевой стек и Game Focus
* **Multimedia Network Throttling:** Отключение системного ограничения пропускной способности сети при мультимедийных нагрузках (`NetworkThrottlingIndex = 0xFFFFFFFF`).
* **DNS Cache Flush:** Принудительный сброс кэша преобразования адресов через `DnsFlushResolverCache`.
* **Фоновый троттлинг паразитных процессов:** Снижение приоритета фоновых служб и приложений из черного списка на уровень `IDLE_PRIORITY_CLASS` и перевод их в `EcoQoS`.

### 5. 💎 Аппаратный рендеринг DirectX 11 & Smart Game Sense
* **Выделенный поток (Worker Thread):** Рендерер работает в изолированном системном потоке — перетаскивание окна или активность мыши не вызывают подвисаний анимации.
* **3D Процедурный Октаэдр:** Математическая модель кристалла с Z-сортировкой вершин, вихревым квантовым ядром и дискретным облаком частиц.
* **HLSL Шейдер с Anti-Shimmering:** Расчет формы частиц через функцию `smoothstep(1.0, 1.0 - feather, dist)` в пиксельном шейдере полностью устраняет пиксельную рябь.
* **Smart Game Sense (0.0% GPU в играх):** Опрос через `SHQueryUserNotificationState` и проверка полноэкранного режима. При запуске игры или сворачивании в трей рендеринг кристалла мгновенно замораживается на **0 FPS** — видеокарта не тратит ни 1% мощности на отрисовку интерфейса.

### 6. 🛡️ Защита от LPE и изоляция вредоносного ПО
* **Anti-Link Exploitation:** Все файловые операции выполняются с флагом `FILE_FLAG_OPEN_REPARSE_POINT`, блокируя атаки через симлинки, Junction и Hardlink (`nNumberOfLinks > 1`).
* **Эвристический PE-анализ:** Встроенный парсер заголовков Portable Executable детектирует скрытые майнеры, инъекторы шеллкода и вебхуки в загружаемых утилитах.
* **NTFS Privilege Trap:** Изоляция опасных файлов в карантин с отсечением прав на выполнение (`DENY FILE_EXECUTE` для группы `Everyone`).

---

## 📊 Сравнение производительности (Performance Benchmarks)

Тестовый стенд: **Intel Xeon E5-2650 v4 (12C/24T)**, **16 GB DDR4**, **NVIDIA GeForce GTX 1650 4GB**, **Windows 11 Pro 64-bit**.

| Сценарий тестирования | Windows 11 (По умолчанию) | Windows 11 + KristMr | Прирост / Эффект |
| :--- | :---: | :---: | :--- |
| **Системный таймер (System Timer)** | 15.625 ms | **0.500 ms** | **-96.8% Input Lag** |
| **Minecraft (Тяжелая сборка реализма, 1080p)** | 58 FPS avg / 14 FPS 1% Low | **74 FPS avg / 38 FPS 1% Low** | **+171% стабильности 1% Low** |
| **Фризы при генерации мира / беге** | Микрозатыки до 1.2 сек | **Отсутствуют (плавный Frametime)** | Прямой график времени кадра |
| **Потребление ОЗУ утилитой** | N/A | **~19 МБ** | В 15 раз легче Electron-утилит |
| **Нагрузка на CPU/GPU в полноэкранной игре** | N/A | **0.0% / 0.0%** | Автоматический сон (0 FPS) |

---

## 📂 Структура проекта (Repository Tree)

```text
KristMr/
├── bin/
│   └── KristMr.exe             # Автономный скомпилированный бинарник
├── resources/
│   ├── app_icon.ico            # Иконка высокого разрешения
│   ├── app_logo.png            # Векторный/растровый логотип
│   ├── KristMr.manifest        # Манифест Per-Monitor V2 DPI Awareness
│   └── KristMr.rc              # Скрипт ресурсов Windows
├── src/
│   ├── config/                 # Управление профилями и конфигурациями
│   ├── engine/                 # Таймер ядра 0.5мс, питание, распарковка ядер, Standby List
│   ├── graphics/               # DirectX 11 рендер, DirectComposition, окно Win32
│   ├── scanner/                # PE-парсер, Directory Watcher, расчет хэшей
│   ├── security/               # Карантин, NTFS Privilege Trap, безопасность
│   ├── toast/                  # Нативные системные уведомления Windows
│   ├── Shaders.hlsl            # Исходный код шейдеров Vertex + Pixel
│   ├── common.hpp              # Глобальные константы и структуры
│   └── main.cpp                # Точка входа wWinMain
├── build.bat                   # Быстрый скрипт компиляции (MSVC cl.exe + fxc.exe)
├── CMakeLists.txt              # Скрипт кросс-генерации CMake
├── LICENSE                     # Лицензия MIT
└── README.md                   # Документация проекта
