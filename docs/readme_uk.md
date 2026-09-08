# Cursor History

<p align="center">
  <img src="readme-banner.png" alt="cursor-history: єдиний інтерфейс для всієї історії Cursor. Composer, Agent transcripts і Store / ACP об’єднано через CLI та API Node.js." width="960">
</p>

[![npm version](https://img.shields.io/npm/v/cursor-history.svg)](https://www.npmjs.com/package/cursor-history)
[![npm downloads](https://img.shields.io/npm/dm/cursor-history.svg)](https://www.npmjs.com/package/cursor-history)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js](https://img.shields.io/badge/Node.js-20%2C%2022--26-green.svg)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0%2B-blue.svg)](https://www.typescriptlang.org/)

🇺🇸 [English](../README.md) | 🇨🇳 [中文](./readme_zh.md) | 🇫🇷 [Français](./readme_fr.md) | 🇪🇸 [Español](./readme_es.md) | 🇯🇵 [日本語](./readme_ja.md) | 🇰🇷 [한국어](./readme_ko.md) | 🇺🇦 [Українська](./readme_uk.md)

**Єдиний інтерфейс для всієї вашої історії Cursor.**

Розмови Cursor можуть зберігатися окремо в різних робочих просторах, базах даних IDE, транскриптах Agent і новіших сховищах сеансів CLI / ACP. `cursor-history` автоматично знаходить підтримувані локальні джерела та надає доступ до них через один CLI й бібліотеку Node.js.

Шукайте в тексті розмов у різних робочих просторах, переглядайте повідомлення та записані дії інструментів, експортуйте в Markdown або JSON. Створюйте резервні копії історії Composer, відновлюйте її та переносьте підтримувані сеанси Composer, коли проєкт змінює розташування.

**Уже маєте кілька місяців історії Cursor? Попереднє збирання даних і налаштування індексу не потрібні.** Пошук працює локально, без ембедингів і ключа API.

<a id="quick-start"></a>
## Швидкий старт

```bash
npm install -g cursor-history

cursor-history list --all
cursor-history search "authentication"
cursor-history show 1
cursor-history export 1
cursor-history backup
```

Потрібні Node.js 20.x або 22.x–26.x та наявна локальна історія Cursor. Щоб спробувати без глобального встановлення, виконайте `npx cursor-history list --all`.

Числа в `show` та `export` відповідають списку з того самого джерела даних і в тих самих межах робочого простору. Для збережених команд використовуйте UUID сеансу. `backup` архівує бази Composer, але не бази Store чи транскрипти.

[Встановлення](#installation) · [Використання](#usage) · [Приклади виводу](../README.md#example-output) · [API бібліотеки](#library-api) · [План розвитку](#roadmap) · [Сумісність](#compatibility)

## Навіщо потрібен цей інструмент

Ви можете пам’ятати, що вже розв’язували проблему з Cursor, але не пам’ятати проєкт, сеанс чи інтерфейс. Пошук за ключовим словом у локальній історії допоможе знайти ту розмову.

[Agent CLI від Cursor](https://cursor.com/docs/cli/reference/parameters) має `agent ls`, `agent resume` і `agent --resume=<id>` для пошуку або продовження сеансів CLI. `cursor-history` додає засоби читання та керування підтримуваною локальною історією в різних робочих просторах і форматах зберігання.

| Що ви хочете зробити | З чого почати |
|---|---|
| Продовжити розмову в Cursor Agent CLI | `agent ls` або `agent --resume=<id>` від Cursor |
| Знайти фразу в тексті розмов у різних робочих просторах | `cursor-history search "connection pool"` |
| Прочитати або експортувати знайдений локальний сеанс | `cursor-history show 1` / `cursor-history export 1` |
| Створити резервну копію історії Composer або відновити її | [Резервне копіювання та відновлення](#backup-restore) |
| Перенести підтримувані сеанси Composer до іншого робочого простору | [Перенесення сеансів](#migration) |
| Використовувати історію у власній програмі | [API Node.js](#library-api) |

## Підтримка різних поколінь сховищ Cursor

Різні версії та інтерфейси Cursor можуть залишати на одному комп’ютері кілька локальних представлень історії. Інструмент знаходить такі підтримувані джерела:

| Джерело | Локальні файли | Призначення |
|---|---|---|
| Старий формат / Composer | `workspaceStorage/*/state.vscdb` і `globalStorage/state.vscdb` у каталозі даних користувача Cursor | Записи робочих просторів і глобальні дані розмов |
| Транскрипти Agent | `~/.cursor/projects/**/agent-transcripts/**/*.jsonl` | Записаний текст розмов і виклики інструментів |
| Store / CLI | `~/.cursor/chats/**/store.db` | Дані розмов Store для кожного сеансу |
| Сеанси ACP | `~/.cursor/acp-sessions/**/store.db` | Дані Store, знайдені в кореневому каталозі ACP |

Шляхи для різних платформ, власні кореневі каталоги та налаштування WSL описано в розділі [Розташування даних](#storage).

Ці формати не завжди містять однакові поля. У транскрипті можуть бути відсутні часові позначки чи результати інструментів. Якщо в дозволених межах одночасно є придатна база Store і транскрипт, основу розмови Store надає база, а транскрипт залишається відомостями про походження. Дані про джерела та походження часових позначок дають змогу розрізняти збережені значення, оцінки й неповні представлення.

Підтримка читання не означає підтримки резервного копіювання чи перенесення. Наявні резервні архіви містять лише Composer; сеанси Store-only та сеанси з об’єднаних джерел переносити не можна. Розширення підтримки включено до [плану розвитку](#roadmap). Повні обмеження наведено в [контракті сумісності та цілісності даних](./compatibility.md).

## Три способи використати історію

### Знайти

```bash
cursor-history list --all
cursor-history search "connection pool"
cursor-history show 1
```

Знайдіть розмову, в якій уже розв’язали проблему, навіть якщо вона належить іншому робочому простору. Пошук охоплює й підтримувану локальну історію, створену до встановлення інструмента.

### Зберегти

```bash
cursor-history export 1
cursor-history backup
cursor-history migrate-session 1 /path/to/new/workspace --dry-run
```

Експортуйте доступні для читання сеанси в Markdown або JSON, створюйте резервні копії Composer і переглядайте заплановане перенесення підтримуваних сеансів Composer до внесення змін.

### Використати знову

Працюйте безпосередньо з [бібліотекою Node.js](#library-api) або підключіть окремий сервер [cursor-history-mcp](https://github.com/S2thend/cursor-history-mcp), щоб сумісний із MCP асистент міг шукати у вашій наявній історії розробки.

Місяці рішень, виправлень, запитів і дій інструментів уже можуть бути на диску. Зробіть цей контекст доступнішим, коли він знадобиться знову.

## Можливості

- Робота через CLI або бібліотеку Node.js.
- Список сеансів і пошук за ключовим словом у тексті розмов у різних робочих просторах.
- Перегляд наявних у джерелах повідомлень, змін файлів, аргументів і результатів інструментів, міркувань ШІ та походження часових позначок.
- Експорт у Markdown / JSON.
- Переміщення та копіювання підтримуваних сеансів Composer.
- Резервне копіювання та відновлення баз Composer.
- Підтримка macOS, Windows і Linux.

<a id="installation"></a>
## Встановлення

Глобальне встановлення через npm:

```bash
npm install -g cursor-history
cursor-history list
```

Для pnpm:

```bash
pnpm add -g cursor-history
cursor-history list
```

Збирання з вихідного коду:

```bash
git clone https://github.com/S2thend/cursor-history.git
cd cursor-history
npm install
npm run build
node dist/cli/index.js list
npm link
```

Після `npm link` можна виконувати `cursor-history list` безпосередньо. Для pnpm використовуйте `pnpm install` і `pnpm build`.

## Вимоги

- Node.js 20.x або 22.x–26.x. Node 21 не підтримується. Читання через вбудований SQLite доступне з Node.js 22.5+.
- Наявна локальна історія Cursor IDE або Agent CLI у підтримуваному форматі.

## Налаштування драйвера SQLite

| Драйвер | Умова вибору | Підтримка Node.js |
|---|---|---|
| `node:sqlite` | Перший вибір, якщо є всі API, потрібні для операції | Читання з 22.5, онлайн-копіювання з 22.16.0 / 23.8.0 |
| `better-sqlite3` | Автоматична альтернатива, якщо встановлений і має потрібні можливості | Підтримувані основні версії: 20, 22–26 |

Драйвер обирається окремо для кожної операції за фактичними можливостями. Примусово вибраний драйвер не замінюється автоматично. Якщо потрібної функції немає, повертається типізована помилка з причиною та способом виправлення. Збої інфраструктури знімків Store не приховуються переходом до транскрипту.

Приклади примусового вибору драйвера та діагностичного виводу:

```bash
CURSOR_HISTORY_SQLITE_DRIVER=better-sqlite3 cursor-history list
CURSOR_HISTORY_SQLITE_DRIVER=node:sqlite cursor-history list
DEBUG=cursor-history:* cursor-history list
```

Налаштування через бібліотеку:

```typescript
import { setDriver, getActiveDriver, listSessions } from 'cursor-history';

setDriver('better-sqlite3');
console.log(getActiveDriver());

const result = await listSessions({ sqliteDriver: 'node:sqlite' });
```

<a id="usage"></a>
## Використання

### Список сеансів

Типово показуються 20 останніх сеансів. `--all` показує всі, `--ids` додає ID, `-n` задає кількість, а `--workspaces` показує робочі простори.

```bash
cursor-history list
cursor-history list --all
cursor-history list --ids
cursor-history list -n 10
cursor-history list --workspaces
```

`list --workspaces` виконує пошук без обмеження робочим простором, тому його не можна поєднувати з `--workspace`. Оберіть шлях із результату та використовуйте те саме значення `--workspace` у наступних командах.

### Перегляд сеансу

`--short` скорочує повідомлення, `--think` показує повні міркування, `--tool` — подробиці викликів інструментів, `--error` — повні помилки. У `--only` типи повідомлень розділяються комами.

```bash
cursor-history show 1
cursor-history show 1 --short
cursor-history show 1 --think
cursor-history show 1 --tool
cursor-history show 1 --error
cursor-history show 1 --only user,assistant
cursor-history show 1 --only tool,error
cursor-history show 1 --short --think --tool --error
cursor-history show 1 --json
```

Відповідь природною мовою зі структурованими викликами інструментів відповідає фільтрам `assistant` і `tool`. Записи лише з інструментами відповідають тільки `tool`. Відсутні в оригінальному джерелі дані відновити неможливо.

### Пошук

`-n` обмежує кількість результатів, а `--context` задає кількість символів контексту навколо збігу.

```bash
cursor-history search "react hooks"
cursor-history search "api" -n 5
cursor-history search "error" --context 100
```

### Експорт

Типовий формат — Markdown. `--format json` обирає JSON, `--all` експортує всі сеанси. `--force` перезаписує наявний результат.

```bash
cursor-history export 1
cursor-history export 1 -o ./my-chat.md
cursor-history export 1 --format json
cursor-history export --all -o ./exports/
cursor-history export 1 --force
```

<a id="migration"></a>
### Перенесення сеансів

Підтримуються лише сеанси Composer, які відповідають вимогам. Store-only, сеанси з об’єднаних джерел і неоднозначні сеанси відхиляються. Спершу перевірте операцію через `--dry-run`. `--copy` зберігає оригінальну розмову.

```bash
cursor-history migrate-session --dry-run 1 /path/to/new/project
cursor-history migrate-session 1 /path/to/new/project
cursor-history migrate-session 1,3,5 /path/to/project
cursor-history migrate-session --copy 1 /path/to/project
cursor-history migrate /old/project /new/project
cursor-history migrate --copy /project /backup/project
cursor-history migrate --force /old/project /existing/project
```

<a id="backup-restore"></a>
### Резервне копіювання та відновлення

Архіви містять дані Composer `state.vscdb`. Бази Store, транскрипти Agent і сховища сеансів ACP до них не входять. Доступні для читання розмови з цих джерел можна експортувати в Markdown / JSON, але такі експорти не є резервними архівами для відновлення.

```bash
cursor-history backup
cursor-history backup -o ~/my-backup.zip
cursor-history backup --force
cursor-history list-backups
cursor-history list-backups -d /path/to/backups
cursor-history restore ~/cursor-history-backups/backup.zip
cursor-history restore backup.zip --target /custom/cursor/data
cursor-history restore backup.zip --force

cursor-history list --backup ~/backup.zip
cursor-history show 1 --backup ~/backup.zip
cursor-history search "query" --backup ~/backup.zip
cursor-history export 1 --backup ~/backup.zip
```

`restore --force` перезаписує перевірені цільові файли, але не вимикає перевірки цілісності та шляхів. Параметр `--backup` дає змогу читати історію з архіву без відновлення.

### Глобальні параметри

```bash
cursor-history --json list
cursor-history --data-path /path/to/Cursor/User list
cursor-history --workspace /path/to/project list
cursor-history --workspace /path/to/project --include-cross-workspace-sources show 1
```

Успішний JSON записується у стандартний вивід, а фатальна JSON-помилка — у стандартний потік помилок із ненульовим кодом завершення. `--workspace` обмежує як належність сеансів, так і читання їхнього вмісту. `--include-cross-workspace-sources` розширює читання лише для додаткових джерел уже вибраних UUID.

## Що можна переглядати

Доступні в джерелах розмови, diff, вміст прочитаних файлів, умови пошуку, команди термінала, результати й помилки інструментів, міркування ШІ, блоки коду та діаграми Mermaid. Кожне отримане повідомлення відображається один раз у правильному порядку. Послідовні повтори не згортаються, щоб не приховати окремі дії інструментів або відомості про походження.

Часові позначки мають `timestampSource`; оцінені значення відрізняються від безпосередньо збережених. Якщо однаковий UUID є і в Composer, і в Store, зберігаються сумісні ID Composer, а дозволені джерела об’єднуються. Відомі джерела поза межами робочого простору не відкриваються, і результат явно позначається як неповний.

<a id="storage"></a>
## Розташування даних Cursor

| Платформа | Composer | Store |
|---|---|---|
| macOS | `~/Library/Application Support/Cursor/User/` | `~/.cursor/` |
| Windows | `%APPDATA%/Cursor/User/` | `%USERPROFILE%\.cursor\` |
| Linux / WSL | `~/.config/Cursor/User/` | `~/.cursor/` |

Обидва сховища виявляються автоматично. Основне джерело повідомлень Store — файл `store.db` кожного сеансу. За справного драйвера та інфраструктури знімків транскрипт може бути альтернативою, якщо база відсутня, не містить придатних повідомлень або має пошкодження чи помилку читання вихідних даних. Недостатні можливості драйвера або збій інфраструктури знімків спричиняють фатальну помилку. За наявності придатної бази транскрипт зберігається лише як відомості про замінене джерело.

Використовуйте `--data-path <path>` або `CURSOR_DATA_PATH` для власного каталогу даних Cursor. `CURSOR_STORE_ROOT` незалежно задає корінь Store. Можна вказати сам корінь або його підкаталоги `chats`, `projects`, `acp-sessions`; вони нормалізуються до одного кореня.

У WSL дані Store з боку Windows зазвичай доступні за шляхом `/mnt/c/Users/<windows-user>/.cursor`. Приклад: `CURSOR_STORE_ROOT=/mnt/c/Users/<windows-user>/.cursor cursor-history list --all`. Для історії, створеної агентом Cursor усередині WSL, використовуйте `~/.cursor` відповідного середовища.

Не використовуйте у WSL нативні `node_modules`, встановлені у Windows. Перед тестуванням чи збиранням встановіть залежності за допомогою Linux Node.js в окремому каталозі залежностей WSL. Інструмент не встановлює й не видаляє залежності автоматично.

<a id="library-api"></a>
## API бібліотеки

Крім CLI, бібліотеку можна використовувати з Node.js. Для посилань, які зберігаються та використовуються повторно, беріть точний UUID із результату списку.

```typescript
import {
  listSessions,
  getSession,
  searchSessions,
  exportSessionToMarkdown
} from 'cursor-history';

const page = await listSessions({ limit: 10 });
console.log(page.pagination.total);
for (const item of page.data) {
  console.log(item.id, item.messageCount);
}

const first = page.data[0];
if (first) {
  const session = await getSession(first.id);
  console.log(session.messages);
  const markdown = await exportSessionToMarkdown(first.id);
  console.log(markdown);
}

const matches = await searchSessions('authentication', { context: 2 });
for (const match of matches) {
  console.log(match.messageIndex, match.offset, match.match);
}
```

Індекси читання бібліотеки починаються з 0; індекси CLI та селектори перенесення бібліотеки — з 1. Це тимчасові значення, залежні від джерела даних, меж робочого простору та стану списку.

У v0.18.0 `messageIndex` у результатах пошуку означає позицію в повному масиві `session.messages`, а `offset` — зміщення в кодових одиницях UTF-16 у повному оригінальному повідомленні. `match` і контекст містять повні початкові рядки. Збережені координати v0.16/v0.17 слід обчислити повторно. JSON-експорти отримують додатковий `index`, що починається з 0.

### API перенесення

```typescript
import { migrateSession, migrateWorkspace } from 'cursor-history';

const preview = await migrateSession({
  sessions: 3,
  destination: '/path/to/new/project',
  dryRun: true
});
console.log(preview);

const copies = await migrateSession({
  sessions: [1, 3, 5],
  destination: '/path/to/project',
  mode: 'copy'
});
console.log(copies);

const workspaceResult = await migrateWorkspace({
  source: '/old/project',
  destination: '/new/project'
});
console.log(workspaceResult.successCount);
```

### API резервного копіювання

```typescript
import {
  createBackup,
  restoreBackup,
  validateBackup,
  listBackups,
  getDefaultBackupDir,
  listSessions
} from 'cursor-history';

const result = await createBackup({
  outputPath: '/path/to/cursor-backup.zip',
  onProgress: (progress) => {
    console.log(progress.phase, progress.filesCompleted, progress.totalFiles);
  }
});
console.log(result.backupPath, result.manifest.stats.sessionCount);

const validation = await validateBackup('/path/to/cursor-backup.zip');
console.log(validation.status, validation.corruptedFiles);

const restored = await restoreBackup({ backupPath: '/path/to/cursor-backup.zip' });
console.log(restored.filesRestored, restored.warnings);

console.log(getDefaultBackupDir());
const backups = await listBackups();
for (const backup of backups) {
  console.log(backup.filename, backup.manifest?.stats.sessionCount);
}

const sessions = await listSessions({ backupPath: '/path/to/cursor-backup.zip' });
```

Перевіряйте `restored.warnings`: пошкоджені записи пропускаються й не відновлюються. API також працює лише з резервними копіями Composer.

### Доступні функції

| Функція | Опис |
|---|---|
| `listSessions(config?)` | Список сеансів із пагінацією |
| `getSession(index, config?)` | Отримання сеансу за індексом або UUID |
| `searchSessions(query, config?)` | Пошук у сеансах |
| `exportSessionToJson(index, config?)` | Експорт сеансу в JSON |
| `exportSessionToMarkdown(index, config?)` | Експорт сеансу в Markdown |
| `exportAllSessionsToJson(config?)` | Експорт усіх сеансів у JSON |
| `exportAllSessionsToMarkdown(config?)` | Експорт усіх сеансів у Markdown |
| `migrateSession(config)` | Переміщення або копіювання підтримуваних сеансів Composer |
| `migrateWorkspace(config)` | Переміщення або копіювання підтримуваних сеансів Composer між робочими просторами |
| `createBackup(config?)` | Резервне копіювання історії Composer |
| `restoreBackup(config)` | Відновлення історії з резервної копії |
| `validateBackup(path)` | Перевірка цілісності резервної копії |
| `listBackups(directory?)` | Список доступних резервних копій |
| `getDefaultBackupDir()` | Шлях до типового каталогу резервних копій |
| `getDefaultDataPath()` | Шлях до даних Cursor для поточної платформи |
| `setDriver(name)` | Вибір драйвера SQLite |
| `getActiveDriver()` | Назва поточного драйвера SQLite |

### Налаштування

```typescript
import type { LibraryConfig } from 'cursor-history';

const config: LibraryConfig = {
  dataPath: '/path/to/Cursor/User',
  workspace: '/path/to/project',
  limit: 20,
  offset: 0,
  context: 2,
  sqliteDriver: 'node:sqlite',
  messageFilter: ['user', 'assistant']
};
```

`limit` і `offset` керують пагінацією, `context` задає кількість рядків контексту пошуку, а `messageFilter` — фільтр типів повідомлень. Параметр `backupPath` перемикає читання з поточних даних на архів. Межі читання й додаткові налаштування описано в [контракті сумісності](./compatibility.md).

### Обробка помилок

```typescript
import {
  listSessions,
  createBackup,
  restoreBackup,
  isDatabaseLockedError,
  isDatabaseNotFoundError,
  isSessionNotFoundError,
  isWorkspaceNotFoundError,
  isBackupError,
  isInvalidBackupError,
  isRestoreError,
  isBackupPublishedPermissionError,
  isRestoreRollbackError,
  validateMessageTypes
} from 'cursor-history';

const invalidTypes = validateMessageTypes(['invalid']);
console.log(invalidTypes);

try {
  await listSessions();
} catch (err) {
  if (
    isDatabaseLockedError(err) ||
    isDatabaseNotFoundError(err) ||
    isSessionNotFoundError(err) ||
    isWorkspaceNotFoundError(err)
  ) {
    console.error(err.message);
  } else {
    throw err;
  }
}

try {
  await createBackup({ outputPath: '/private/backups/cursor.zip' });
} catch (err) {
  if (isBackupPublishedPermissionError(err)) {
    console.error(err.details.pathIdentityVerified, err.details.outputPath);
  } else if (isBackupError(err) || isInvalidBackupError(err)) {
    console.error(err.message);
  } else {
    throw err;
  }
}

try {
  await restoreBackup({ backupPath: '/path/to/cursor-backup.zip' });
} catch (err) {
  if (isRestoreRollbackError(err)) {
    console.error(err.details.residualFiles);
  } else if (isRestoreError(err) || isInvalidBackupError(err)) {
    console.error(err.message);
  } else {
    throw err;
  }
}
```

Якщо базу заблоковано, закрийте Cursor і повторіть спробу. Якщо дані чи робочий простір не знайдено, перевірте їхнє розташування та шлях проєкту. За помилки дозволів уже створеної резервної копії перевірте `pathIdentityVerified`; не змінюйте права й не видаляйте неперевірені шляхи. `RESTORE_ROLLBACK_INCOMPLETE` означає, що після запису відновлення залишилося частковим. Зупиніть Cursor і виконайте ручне відновлення з надійної копії.

<a id="compatibility"></a>
## Сумісність і безпечні оновлення

[Compatibility and Data-Integrity Contract](./compatibility.md) є нормативним джерелом щодо ID, меж індексів, I/O робочих просторів, повноти джерел, походження часу, обмежень читання, дозволів та оновлень. У разі розбіжностей пріоритет має цей англомовний контракт.

- `Session.id` залишається рідним UUID Cursor. Порівняння побайтове й чутливе до регістру; фізичне розташування зберігається в окремих метаданих.
- Для робочих просторів спершу застосовується точний нормалізований збіг, потім — однозначний збіг суфікса з повних компонентів шляху. Вміст за межами вибраного простору типово не читається.
- `source: "global"` означає повні дані, придатні для заміни. `source: "workspace-fallback"` означає часткові дані, якими не слід перезаписувати повний архів. `resolvedSource`, `sources` і `resolution` описують фактичне походження.
- Споживачі, які інкрементально зберігають Store / об’єднані дані v0.17, мають перевірити шлях виправлення v0.18.0 і створити резервну копію свого архіву перед оновленням. Гарантія оновлення без зміни коду споживача обмежена переходом від архіву v0.16 лише з Composer до повного об’єднаного представлення на основі Composer. Нестабільні синтетичні Store ID v0.17 не зберігаються. Для неповних даних потрібні фіксація версії, повторне читання повних джерел або ручне перенесення.
- Не використовуйте максимальну часову позначку як межу інкрементальної синхронізації. Перевіряйте повноту та зміни вмісту й не замінюйте повну історію частковим результатом.
- У POSIX тимчасові каталоги знімків мають права `0700`, файли — `0600`; очищення виконується як після успіху, так і після помилки. Нові архіви типово мають `0600`, примусовий перезапис зберігає наявний режим. `backup --shared` запитує спільний доступ лише до кінцевого архіву. Захист ACL у Windows не заявляється як еквівалентна гарантія POSIX.
- Помилка дозволів або очищення після розміщення резервної копії за кінцевим шляхом не означає, що архів відсутній. Перевіряйте стан публікації та ідентифікацію шляху в типізованій помилці; не видаляйте неперевірені шляхи й не повторюйте операцію наосліп із `--force`.
- Відновлення відхиляє порожні списки, недопустимі шляхи, незадекларований вміст, повторні призначення та небезпечні посилання, а пошкоджені записи пропускає. `--force` не обходить перевірок. Помилка після запису не спричиняє автоматичного відкочування: залишковий стан повертається через `RESTORE_ROLLBACK_INCOMPLETE`.
- `manifest.version` резервної копії залишається `1.0.0`, а необов’язковий список Composer використовує незалежний `schemaVersion: 1`. Читання в межах робочого простору не розпаковує спільну глобальну базу й повертає часткове представлення. Старі архіви кількох робочих просторів без потрібного списку відхиляються з `BACKUP_WORKSPACE_SCOPE_METADATA_REQUIRED`.

<a id="roadmap"></a>
## План розвитку

Розширити резервне копіювання, відновлення та перенесення на нові джерела, читання яких уже підтримується. Це заплановані напрями; версію та дату випуску ще не визначено.

- [ ] **Резервне копіювання й відновлення Store / ACP**: включити бази Store, транскрипти Agent і пов’язані метадані до відновлюваних архівів, зберегти зв’язки між джерелами та перевірити повний цикл копіювання й відновлення.
- [ ] **Перенесення сеансів Store-only**: переміщувати або копіювати сеанси між робочими просторами, оновлювати прив’язки та посилання на шляхи й перевіряти, що Cursor знаходить ці сеанси.
- [ ] **Перенесення сеансів з об’єднаних джерел**: переміщувати або копіювати сеанси, наявні одночасно в Composer і Store, зі збереженням рідних ID та узгодженості джерел.

До реалізації цих можливостей [резервне копіювання та відновлення](#backup-restore) охоплюють лише Composer, а [перенесення](#migration) — лише сеанси Composer, які відповідають вимогам. Доступні розмови Store / ACP можна експортувати в Markdown / JSON, але це не архіви для відновлення.

Діліться сценаріями використання та відтворюваними прикладами сховищ через [GitHub Issues](https://github.com/S2thend/cursor-history/issues), щоб допомогти визначити пріоритети.

<a id="development"></a>
## Розробка

### Збирання й тестування

```bash
npm install
npm run build
npm run typecheck
npm run lint
npm test
npm run test:watch
```

Для pnpm доступні `pnpm install`, `pnpm build`, `pnpm typecheck`, `pnpm lint`, `pnpm test` і `pnpm test:watch`.

### Випуск у npm

Використовується npm trusted publishing через GitHub Actions, без секрету репозиторію `NPM_TOKEN`. Перед першим випуском прив’яжіть trusted publisher у npm до цього репозиторію, `npm-publish.yml` і середовища `npm-release-verification`, дозволивши дію `npm publish`. У GitHub створіть однойменне захищене середовище з обов’язковими рецензентами.

1. Оновіть і перевірте метадані з версіями та примітки до випуску, пройдіть перевірки й зафіксуйте чисту ревізію.
2. Переконайтеся, що тег версії ще не існує, і надішліть лише цей тег. Не надсилайте й не переміщуйте тег випуску до фіксації ревізії.
3. Процес перевіряє код і підтримувані середовища Node.js, пакує рівно один раз і пов’язує кандидата з ревізією та SHA-256. Завдання публікації зупиняється в захищеному середовищі до запиту токена OIDC.
4. Завантажте кандидата, ідентифікованого контрольною сумою, перевірте той самий артефакт за [процедурою перевірки випуску](./release-verification.md) і лише тоді схваліть середовище.
5. Опублікуйте ті самі збережені байти з npm provenance, без повторного збирання чи пакування.

Будь-яка невдала перевірка блокує публікацію. Не пересувайте тег випуску примусово без явного рішення. Якщо байти вже опубліковані, потрібна нова версія.

## Участь у проєкті

Повідомляйте про помилки й пропонуйте функції через [Issues](https://github.com/S2thend/cursor-history/issues/new). Для помилок додайте кроки відтворення, очікувану й фактичну поведінку, ОС і версію Node.js.

Для змін створіть fork і робочу гілку, внесіть правки, виконайте `npm test` та `npm run lint`, зробіть commit і push, а потім відкрийте [Pull Request](https://github.com/S2thend/cursor-history/pulls).

## Ліцензія

[MIT](../LICENSE)
