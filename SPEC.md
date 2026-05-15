# Krokodil Studio — Полная спецификация приложения

> Документ описывает архитектуру, структуру данных, дизайн-систему и все ключевые паттерны
> одностраничного приложения для управления детской арт-студией.
> Используется как эталон для поддержки существующего приложения и генерации аналогичных.

---

## 1. Проект

| Параметр | Значение |
|---|---|
| Название | Арт-студия Крокодил |
| Заголовок страницы | `Арт-студия Крокодил — детская студия творчества в Архангельске` |
| Отображаемое имя | `🐊 Крокодил — Арт-студия` |
| Тип | Single-File SPA (HTML + CSS + JS в одном файле) |
| Язык интерфейса | Русский |
| Репозиторий | `tapemode-hash/krokodil_studio` |
| Деплой | GitHub Pages, ветка `main`, файл `index.html` |
| `DATA_VERSION` | `2` |

---

## 2. Архитектура

### Принцип single-file SPA

Всё приложение — один файл `index.html` (~7500+ строк):

```
index.html
├── <head>          — мета, шрифты, <style> (весь CSS)
├── <body>          — вся разметка: страницы, модалки, оверлеи
└── <script>        — весь JS: константы, данные, функции, обработчики
```

Никаких бандлеров, фреймворков, npm-зависимостей. Чистый ванильный JS.

### Навигация

- Одна глобальная переменная `currentPage` хранит ID текущей страницы.
- `showPage(id)` скрывает все `.page`, показывает нужную, вызывает render-функцию страницы.
- В URL ничего не меняется (нет `history.pushState`).

### Хранилище

Единственное хранилище — `localStorage`. Все ключи с префиксом `krok_`.
Утилиты: `lsLoad(key, def)` / `lsSave(key, val)`. Вызов `persist()` сохраняет все 29 переменных разом.

### Рендеринг

Все данные хранятся в глобальных переменных. При любом изменении:
1. Данные обновляются в переменной.
2. Вызывается `persist()`.
3. Вызывается нужная render-функция для перерисовки DOM.

Никакого Virtual DOM, реактивности или наблюдателей. Полная перерисовка DOM при каждом изменении.

---

## 3. Дизайн-система

### Шрифты

```html
<link href="https://fonts.googleapis.com/css2?family=Caveat:wght@400;600;700&family=Nunito:wght@300;400;600;700&display=swap" rel="stylesheet">
```

| Шрифт | Назначение | Веса |
|---|---|---|
| **Nunito** | Основной текст UI | 300, 400, 600, 700 |
| **Caveat** | Декоративные элементы, заголовки | 400, 600, 700 |

### CSS Custom Properties

```css
:root {
  /* Фон */
  --cream:   #FFF8F0;   /* основной фон приложения */
  --warm:    #FFFDF9;   /* тёплый белый, карточки */

  /* Акцентные цвета */
  --peach:   #FFD4B8;   /* персиковый */
  --coral:   #FF8C69;   /* коралловый (основной акцент) */
  --coral-l: #FFB8A0;   /* светлый коралл */
  --mint:    #B8F0D4;   /* мятный */
  --mint-d:  #5ECFA0;   /* тёмная мята */
  --lav:     #D4C4F0;   /* лавандовый */
  --lav-d:   #9B84D8;   /* тёмная лаванда */
  --sky:     #C4E4F8;   /* голубой */
  --sky-d:   #5BAED6;   /* тёмный голубой */
  --yel:     #FFE97A;   /* жёлтый */
  --yel-d:   #E8C840;   /* тёмный жёлтый */
  --grn:     #4CAF50;   /* зелёный (успех) */
  --grn-l:   #A5D6A7;   /* светлый зелёный */

  /* Текст */
  --ink:     #3A2E2E;   /* основной текст */
  --ink-l:   #6B5E5E;   /* вторичный текст */

  /* Тень */
  --sh:      rgba(58,46,46,.08);
}
```

### Цветовые палитры

```js
// Цвета аватаров учителей
const TC = ['var(--peach)','var(--mint)','var(--lav)','var(--sky)','var(--yel)','#FFD4E8','#E8F5E9']

// Emoji аватаров учеников
const SE = ['🦊','🐸','🦋','🐬','🌻','🌸','🌟','🎨']

// Цвета карточек учеников
const SC = ['var(--peach)','var(--mint)','var(--lav)','var(--sky)','var(--yel)','#FFD4E8','#E8F4FD','var(--cream)']
```

### Отзывчивые брейкпоинты

| Брейкпоинт | Назначение |
|---|---|
| `480px` | Мобильные устройства |
| `768px` | Планшеты |
| `900px` | Широкий планшет / узкий десктоп |
| `print` | Стили печати |

---

## 4. Роли и авторизация

| Роль | Доступ | Пароль |
|---|---|---|
| `guest` | `dashboard`, `schedule`, `teachers`, `gallery` | Нет |
| `parent` | `dashboard`, `schedule`, `teachers`, `gallery`, `attendance` | Номер телефона родителя (ищется в `students[].phone`) |
| `admin` | Все страницы | SHA-256: `9511aa000732a111985d7c746e454b746d7e4cd69c5c941ed0d5dac9c99c228b` |

```js
const ADMIN_PW_H = '9511aa000732a111985d7c746e454b746d7e4cd69c5c941ed0d5dac9c99c228b'
const guestPages  = ['dashboard','schedule','teachers','gallery']
const parentPages = ['dashboard','schedule','teachers','gallery','attendance']
```

Авторизация: `currentRole` — глобальная переменная (`'guest'` | `'parent'` | `'admin'`).
Родительский режим: `_parentStudentIds` — массив ID детей текущего родителя.

---

## 5. Страницы

| ID | Название | Роль | Render-функция |
|---|---|---|---|
| `page-dashboard` | Главная / дашборд | guest+ | `renderDashboard()` |
| `page-schedule` | Расписание | guest+ | `renderSchedule()` |
| `page-students` | Ученики | admin | `renderStudents()` |
| `page-teachers` | Педагоги | guest+ | `renderTeachersPublic()` / `renderTeachers()` |
| `page-attendance` | Посещаемость | parent+ | `renderAttendance()` |
| `page-gallery` | Галерея | guest+ | `renderGallery()` |
| `page-exhibitions` | Выставки/Достижения | admin | `renderAchievements()` |
| `page-materials` | Материалы | admin | — |
| `page-payments` | Платежи | admin | `renderPayments()` |
| `page-reports` | Отчёты | admin | `renderReports()` |

### Dashboard (главная)

- Сегодняшнее расписание: `renderTodaySchedule()`
- Мини-календарь: `renderMiniCal()`
- Предстоящие занятия для родителя: `renderParentUpcoming()`
- Абонементы родителя: `renderParentSubs()`
- Объявления: `renderAnnouncements()`

### Schedule (расписание)

- Список всех групп расписания с днями/временем/педагогом
- `renderDateCal()` — выбор даты для посещаемости

### Students (ученики) — только admin

- Пагинация: `STUDENTS_PAGE = 50` учеников на страницу
- Поиск: `_studentQuery`
- Архивные: `showArchived` toggle
- Запросы на запись: `renderRequests()`
- Список зачислений: `renderEnrollList()`
- Список абонементов: `renderSubList()`

### Teachers (педагоги)

- Guest/Parent: публичный вид `renderTeachersPublic()`
- Admin: управление `renderTeachers()`, рейтинги `renderReviewsInModal(tid)`

### Attendance (посещаемость)

- Выбор даты через `attDate`
- Список учеников по группам на выбранный день
- `addPairs()` — формирует массив пар {ученик, группа, ключ посещаемости}
- История посещений: `renderAttHistory()`
- Родительский вид: `renderParentAttendance()`

### Payments (платежи)

- Только admin
- Сортировка `_paySort`: `name` | `att_desc` | `att_asc`
- `renderPayments()`

### Reports (отчёты)

- Только admin
- Отчётный период: `getRpDates()`
- Таблица учеников: `renderReportTable()` — только **активные** ученики
- Метрики аналитики: `renderReports()`
- Чёрн-лог: `renderChurnLog()`

---

## 6. Модальные окна

| ID | Назначение |
|---|---|
| `m-student` | Добавление / редактирование ученика |
| `m-teacher` | Добавление / редактирование педагога |
| `m-schedule` | Добавление / редактирование группы расписания |
| `m-sub-select` | Выбор и привязка абонемента |
| `m-request` | Форма заявки на запись / успех заявки |
| `m-achievement` | Достижение / выставка |
| `m-announce` | Объявление |
| `m-materials` | Материалы занятия |
| `m-photo` | Фото в галерею |
| `m-churn` | Журнал оттока |
| `m-rate-teacher` | Оценка педагога |
| `m-tsalary` | Детализация зарплаты педагога |

---

## 7. Модель данных

### 7.1 localStorage (все ключи с префиксом `krok_`)

```
krok_teachers          — массив учителей
krok_students          — массив учеников
krok_schedule          — массив групп расписания
krok_attendance        — посещаемость { 'YYYY-MM-DD': {attKey: status} }
krok_churnLog          — журнал оттока
krok_marketingCosts    — маркетинговые расходы
krok_nextChurnId       — счётчик ID чёрн-лога
krok_fixedCostsData    — фиксированные расходы [{month, rent, utilities, other}]
krok_enrollments       — зачисления {sid: [scId, ...]}
krok_dateEnrollments   — зачисления по дате {sid: ['YYYY-MM-DD', ...]}
krok_dayAssignments    — временные назначения в группу {'YYYY-MM-DD': {sid: scId}}
krok_announcements     — объявления
krok_nextAnnId         — счётчик ID объявлений
krok_achievements      — достижения/выставки
krok_nextAchId         — счётчик ID достижений
krok_requests          — заявки на запись
krok_nextReqId         — счётчик ID заявок
krok_subscriptions     — абонементы {sid: [subscription, ...]}
krok_lessonCosts       — стоимость занятий {'YYYY-MM-DD': {attKey: price}}
krok_globalLessonPrice — глобальная цена занятия (default: 500)
krok_activePromo       — активная промо {type, value, name}
krok_teacherReviews    — отзывы о педагогах {tid: [{rating, text, date}, ...]}
krok_photos            — фотографии галереи
krok_nextPhotoId       — счётчик ID фото
krok_nextTId           — следующий ID учителя
krok_nextSId           — следующий ID ученика
krok_nextScId          — следующий ID группы
krok_dataVersion       — версия данных (текущая: 2)
krok__fbTs             — технический timestamp
```

### 7.2 Структура объектов

#### Teacher (педагог)
```js
{
  id:      number,     // уникальный ID
  name:    string,     // имя
  avatar:  string,     // emoji
  color:   string,     // CSS var или hex цвет
  role:    string,     // специализации (напр. "Рисунок, живопись")
  groups:  number,     // количество групп
  students: number,    // количество учеников
  rating:  number      // 0–5
}
```

#### Student (ученик)
```js
{
  id:             number,   // уникальный ID
  emoji:          string,   // аватар (emoji)
  color:          string,   // цвет карточки
  name:           string,   // имя
  dob:            string,   // дата рождения 'YYYY-MM-DD'
  age:            number,   // возраст
  parentName:     string,   // имя родителя
  phone:          string,   // телефон (используется как пароль родителя)
  dir:            string,   // направление (напр. "Рисунок")
  group:          string,   // группа
  pay:            string,   // статус оплаты ('paid', etc.)
  pricePerLesson: number    // индивидуальная цена занятия
}
```

#### Schedule Entry (группа расписания)
```js
{
  id:      number,   // уникальный ID
  name:    string,   // название группы
  teacher: string,   // имя педагога
  room:    string,   // зал/комната
  days:    string,   // дни (напр. "Пн, Чт")
  time:    string,   // время (напр. "15:00–17:00")
  status:  string,   // статус (напр. "Активна")
  sc:      string    // цвет (напр. "green")
}
```

#### Subscription (абонемент)
```js
{
  groupName: string,   // название группы
  total:     number,   // всего занятий
  used:      number,   // использовано занятий
  price:     number,   // цена абонемента (₽)
  createdAt: string    // дата создания 'DD.MM.YYYY'
}
// Хранится: subscriptions[sid] = [sub, sub, ...]
```

### 7.3 Посещаемость (attendance)

```js
// Структура
attendance = {
  'YYYY-MM-DD': {
    [attKey]: 'p' | 'a' | 'trial'
  }
}
```

#### Форматы ключей посещаемости (attKey)

| Формат | Когда используется |
|---|---|
| `String(sid)` | Ученик без группы или в одной группе |
| `"sid:scId"` | Ученик, временно назначенный в группу через `dayAssignments` |

**Мета-ключи** в `lessonCosts[dateStr]`:

| Суффикс | Значение |
|---|---|
| `attKey` | Цена занятия |
| `attKey_via_sub` | Оплачено через абонемент |
| `attKey_auto` | Назначено автоматически |
| `attKey_promo` | Применена промо-скидка |
| `attKey_base_price` | Базовая цена |
| `attKey_sub_price` | Цена по абонементу |
| `attKey_sub_total` | Всего по абонементу |
| `attKey_subIdx` | Индекс абонемента |
| `attKey_trial` | Пробное занятие |

### 7.4 Зачисления

```js
// Постоянные зачисления
enrollments[sid] = [scId1, scId2, ...]

// Зачисления по конкретной дате
dateEnrollments[sid] = ['YYYY-MM-DD', ...]

// Временное назначение в группу (на один день)
dayAssignments['YYYY-MM-DD'] = { String(sid): scId }
```

---

## 8. Ключевые JS-функции

### Утилиты данных

```js
lsLoad(key, def)              // загрузить из localStorage с дефолтом
lsSave(key, val)              // сохранить в localStorage
persist()                     // сохранить все переменные разом
fmtDate(date)                 // Date → 'YYYY-MM-DD'
sha256(str)                   // SHA-256 хэш строки
sidFromAttKey(k)              // извлечь sid из attKey (plain или composite)
```

### Навигация и UI

```js
showPage(id)                  // переключить страницу
openModal(id)                 // открыть модалку
closeModal(id)                // закрыть модалку
```

### Ученики и активность

```js
isStudentActive(sid)          // активен ли ученик (был в последние 6 мес.)
getStudentFirstActivity(sid)  // дата первого появления
getStudentLastActivity(sid)   // дата последнего появления
getAtRiskStudents()           // список "под риском" ухода
getAttPct(sid)                // процент посещаемости
```

### Расписание и запись

```js
getScheduleDates(sc, ahead)   // даты занятий группы на ahead месяцев вперёд
getActiveSub(sid)             // активный абонемент ученика
```

### Посещаемость

```js
addPairs()                    // формирует [{s, sc, attKey, isExtra, dayAssigned}]
assignToGroup(sid, scId)      // временно назначить в группу (+ миграция attKey)
removeGroupAssignment(sid)    // убрать назначение (+ обратная миграция)
```

### Финансы и метрики

```js
getRpDates()                               // {from, to} отчётного периода
getRevenueInPeriod(from, to)               // выручка за период
getTrialsInPeriod(from, to)                // пробные за период
getActiveStudentsInPeriod(from, to)        // активные ученики в периоде
getSessionDaysInPeriod(from, to)           // учебные дни в периоде
getStudioLoad(from, to)                    // загруженность студии
getRevenuePerTeacher(from, to)             // выручка по педагогам
openTeacherSalaryDetail(teacherName, ...) // детализация зарплаты педагога
```

### Render-функции

```js
renderAll()                   // перерисовать всё
renderDashboard()             // главная
renderTodayList(elId, ...)    // список занятий на день
renderTodaySchedule()         // расписание на сегодня
renderParentUpcoming(...)     // предстоящие занятия (родитель)
renderParentSubs()            // абонементы родителя
renderMiniCal()               // мини-календарь
renderSchedule()              // страница расписания
renderDateCal()               // выбор даты
renderRequests()              // заявки
renderAchievements()          // достижения
renderAnnouncements()         // объявления
renderStudents(query)         // страница учеников
renderEnrollList()            // список зачислений
renderSubList()               // список абонементов
renderTeachersPublic()        // педагоги (публичный вид)
renderTeachers()              // педагоги (admin)
renderReviewsInModal(tid)     // отзывы в модалке
renderParentAttendance()      // посещаемость (родитель)
renderAttHistory()            // история посещений
renderAttendance()            // посещаемость (admin)
renderPayments()              // платежи
renderReportTable()           // таблица отчёта (только активные ученики)
renderReports()               // аналитика и метрики
renderChurnLog()              // журнал оттока
renderGallery()               // галерея
renderWorkingHours()          // рабочие часы
```

---

## 9. Бизнес-логика (ключевые правила)

### Активность ученика

Ученик считается **активным** если:
- Есть посещение (статус `p`, `a` или `trial`) за последние 6 месяцев, **или**
- Есть запись в `dateEnrollments[sid]` за последние 6 месяцев.

Архивный ученик = не активен. Исключается из всех метрик "текущего состояния".

### Метрики (renderReports) — только активные ученики

- **Retention rate**: `withSubs` и `retained` — только `isStudentActive(s.id)`
- **Конверсия пробного → абонемент**: `trialConverted` — только `isStudentActive(s.id)`
- **LTV / avgLifetime**: `spans` массив — только `isStudentActive(s.id)`
- **Средняя посещаемость (avgAtt)**: `forEach` — только `isStudentActive(s.id)`
- **renderReportTable**: строки — только `isStudentActive(s.id)`

### Временное назначение в группу (dayAssignments)

Ученик без постоянной группы может быть назначен в группу на один день.
При назначении → `attKey` меняется с `String(sid)` на `"sid:scId"` (composite).
При снятии назначения → обратная миграция `"sid:scId"` → `String(sid)`.
При первом открытии страницы посещаемости старые данные (plain key) автоматически мигрируют.

### Атрибуция метрик по педагогу

Для корректной атрибуции в `getRevenuePerTeacher`, `getStudioLoad`, `openTeacherSalaryDetail`:
1. Проверяется `enrollments[sid]` — постоянные группы ученика.
2. Если `enrollments[sid]` пуст → fallback на `dayAssignments[dateStr][sid]`.
3. Если студент — `dayAssigned` → ищется по composite key `sid:scId`.

### Абонементы

- `subscriptions[sid]` — массив абонементов ученика.
- `getActiveSub(sid)` — последний абонемент с `used < total`.
- При посещении с абонементом: `used++`, `_via_sub` = true в `lessonCosts`.

### Оттокк (churn)

- `churnLog` — записи об уходящих учениках.
- `churnRate` в отчёте считается по `activePrev` vs `activeNow` (оба из attendance-записей, не из `students`).

---

## 10. Паттерны HTML/CSS

### Структура страницы

```html
<body>
  <!-- Навигация -->
  <nav class="navbar">...</nav>

  <!-- Экран входа -->
  <div id="login-screen">...</div>

  <!-- Основное содержимое -->
  <div id="app">
    <div id="page-dashboard" class="page">...</div>
    <div id="page-schedule"  class="page">...</div>
    <!-- ... все страницы -->
  </div>

  <!-- Модальные окна -->
  <div id="m-student" class="modal-overlay">
    <div class="modal">...</div>
  </div>
  <!-- ... все модалки -->

  <script>/* весь JS */</script>
</body>
```

### Классы UI-компонентов

| Класс | Назначение |
|---|---|
| `.page` | Страница (скрыта по умолчанию) |
| `.page.active` | Активная страница |
| `.modal-overlay` | Оверлей модалки |
| `.modal` | Контейнер модалки |
| `.card` | Карточка |
| `.btn` | Кнопка |
| `.btn-primary` | Основная кнопка (coral) |
| `.badge` | Бейдж/метка |
| `.chip` | Чип-тег |
| `.avatar` | Аватар (emoji + цвет фона) |

---

## 11. Деплой

```
feature-branch → PR → squash merge → main → GitHub Pages
```

1. Разработка в ветке (`fix/...`, `feat/...`, `claude/...`)
2. PR через GitHub MCP tools (`mcp__github__create_pull_request`)
3. Squash merge в `main`
4. GitHub Pages автоматически деплоит из `main` → `index.html`

---

## 12. Как сгенерировать похожее приложение

### Минимальный шаблон

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>НАЗВАНИЕ — SUBTITLE</title>
  <link href="https://fonts.googleapis.com/css2?family=Caveat:wght@400;600;700&family=Nunito:wght@300;400;600;700&display=swap" rel="stylesheet">
  <style>
    :root {
      --cream: #FFF8F0; --warm: #FFFDF9;
      --peach: #FFD4B8; --coral: #FF8C69; --coral-l: #FFB8A0;
      --mint: #B8F0D4;  --mint-d: #5ECFA0;
      --lav: #D4C4F0;   --lav-d: #9B84D8;
      --sky: #C4E4F8;   --sky-d: #5BAED6;
      --yel: #FFE97A;   --yel-d: #E8C840;
      --grn: #4CAF50;   --grn-l: #A5D6A7;
      --ink: #3A2E2E;   --ink-l: #6B5E5E;
      --sh: rgba(58,46,46,.08);
    }
    /* ... CSS */
  </style>
</head>
<body>
  <!-- разметка -->
  <script>
    const LS = 'APP_PREFIX_'  // <-- сменить для каждого приложения
    const DATA_VERSION = 1
    const ADMIN_PW_H = '...'  // sha256 пароля

    function lsLoad(k, def){ try{ const v=localStorage.getItem(LS+k); return v?JSON.parse(v):def }catch{ return def } }
    function lsSave(k, v){ localStorage.setItem(LS+k, JSON.stringify(v)) }
    function persist(){ /* сохранить все переменные */ }
    function sha256(str){ /* Web Crypto или SubtleCrypto */ }
    function fmtDate(d){ return d.toISOString().split('T')[0] }

    // ... данные, render-функции, обработчики
  </script>
</body>
</html>
```

### Чеклист для нового приложения

- [ ] Сменить `LS` префикс (напр. `'dance_'`, `'school_'`)
- [ ] Определить роли и страницы под задачу
- [ ] Описать модель данных (объекты + localStorage ключи)
- [ ] Реализовать `persist()` для всех переменных
- [ ] Для каждой страницы — одна `renderPageName()` функция
- [ ] Все модалки с ID `m-*`, открывать через `openModal(id)`
- [ ] Вся авторизация через `currentRole` + `sha256()` для admin
- [ ] Адаптив через CSS media queries (480px, 768px, 900px)
- [ ] `DATA_VERSION` + миграция при `lsLoad('dataVersion', 0) < DATA_VERSION`

### Адаптация дизайн-системы

Для изменения темы достаточно переопределить CSS custom properties в `:root`.
Шрифты можно заменить другой парой Google Fonts (декоративный + читаемый).
Цветовые константы `TC`, `SE`, `SC` — под аватары конкретной сущности.

---

## 13. Известные паттерны и решения

### Пагинация учеников
```js
const STUDENTS_PAGE = 50
let studentsPage = 0
// renderStudents() использует studentsPage * STUDENTS_PAGE как offset
```

### Кэш посещаемости
```js
let _attCache = null
// Сбрасывается при любом изменении attendance через persist()
```

### Автоматическая миграция данных
```js
if(lsLoad('dataVersion', 0) < DATA_VERSION) {
  // Миграция: переименовать ключи, добавить поля, etc.
  lsSave('dataVersion', DATA_VERSION)
}
```

### Composite attKey для разрешения конфликтов
Когда ученик посещает несколько групп в один день:
```js
const attKey = enrollments[sid].length > 1 ? `${sid}:${scId}` : String(sid)
```

### Фильтр активных учеников в метриках
```js
// Везде, где считается "текущее состояние", применять:
students.filter(s => isStudentActive(s.id))
// НЕ применять к историческим/финансовым данным
```
