# Cursor History

<p align="center">
  <img src="readme-banner.png" alt="cursor-history：すべての Cursor 履歴をひとつのインターフェースへ。Composer、Agent transcripts、Store / ACP を統合した CLI と Node.js API。" width="960">
</p>

[![npm version](https://img.shields.io/npm/v/cursor-history.svg)](https://www.npmjs.com/package/cursor-history)
[![npm downloads](https://img.shields.io/npm/dm/cursor-history.svg)](https://www.npmjs.com/package/cursor-history)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js](https://img.shields.io/badge/Node.js-20%2C%2022--26-green.svg)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0%2B-blue.svg)](https://www.typescriptlang.org/)

🇺🇸 [English](../README.md) | 🇨🇳 [中文](./readme_zh.md) | 🇫🇷 [Français](./readme_fr.md) | 🇪🇸 [Español](./readme_es.md) | 🇯🇵 [日本語](./readme_ja.md) | 🇰🇷 [한국어](./readme_ko.md) | 🇺🇦 [Українська](./readme_uk.md)

**すべての Cursor 履歴を、ひとつのインターフェースで。**

Cursor の会話は、ワークスペース、IDE のデータベース、Agent のトランスクリプト、新しい CLI / ACP のセッションストアなどに分散していることがあります。`cursor-history` は対応するローカルソースを自動検出し、ひとつの CLI と Node.js ライブラリからアクセスできるようにします。

ワークスペースを横断して会話本文を検索し、メッセージや記録されているツール操作を確認して、Markdown または JSON にエクスポートできます。Composer 履歴のバックアップ・復元や、プロジェクト移動時の対応する Composer セッションの移行も可能です。

**すでに数か月分の Cursor 履歴がありますか？事前の収集やインデックスの設定は不要です。** 検索はローカルで実行され、埋め込みモデルや API キーも必要ありません。

<a id="quick-start"></a>
**MCP サーバー経由で接続したいですか？** [cursor-history-mcp](https://github.com/S2thend/cursor-history-mcp#quick-start) を使うと、履歴機能を MCP ツールとして公開できます。エージェントは本プロジェクトの CLI や Node.js API を直接呼び出すこともできます。ワークフローに合うインターフェースを選んでください。パッケージは個別にリリースされるため、[MCP のストレージ互換性とリリース情報](https://github.com/S2thend/cursor-history-mcp#compatibility)を確認してください。

## クイックスタート

```bash
npm install -g cursor-history

cursor-history list --all
cursor-history search "authentication"
cursor-history show 1
cursor-history export 1
cursor-history backup
```

Node.js 20.x または 22.x–26.x と、既存のローカル Cursor 履歴が必要です。グローバルインストールせずに試す場合は `npx cursor-history list --all` を実行してください。

`show` と `export` の番号は、同じデータソースとワークスペース範囲で取得した一覧に対応します。保存して再利用するコマンドにはセッション UUID を使ってください。`backup` の対象は Composer データベースであり、Store データベースやトランスクリプトは含みません。

[インストール](#installation) · [使い方](#usage) · [出力例](../README.md#example-output) · [ライブラリ API](#library-api) · [ロードマップ](#roadmap) · [互換性](#compatibility)

## このツールが必要な理由

Cursor で問題を解決した記憶はあっても、どのプロジェクト、セッション、画面で相談したのか思い出せないことがあります。ローカル履歴のキーワード検索で、その会話を見つけられます。

Cursor の [Agent CLI](https://cursor.com/docs/cli/reference/parameters) には、CLI セッションを探したり再開したりする `agent ls`、`agent resume`、`agent --resume=<id>` があります。`cursor-history` は、ワークスペースや保存形式をまたいで、対応するローカル履歴を読み取り、管理するための機能を提供します。

| やりたいこと | 使用する機能 |
|---|---|
| Cursor Agent CLI で会話を再開 | Cursor の `agent ls` または `agent --resume=<id>` |
| ワークスペースを横断して会話本文を検索 | `cursor-history search "connection pool"` |
| 検出したローカルセッションを表示・エクスポート | `cursor-history show 1` / `cursor-history export 1` |
| Composer 履歴をバックアップ・復元 | [バックアップと復元](#backup-restore) |
| 対応する Composer セッションを別のワークスペースへ移動 | [セッションの移行](#migration) |
| 自作アプリで履歴を利用 | [Node.js API](#library-api) |

## Cursor の複数世代のストレージに対応

Cursor のバージョンや利用するインターフェースによって、同じマシン上に異なる形式の履歴が残る場合があります。次の対応ソースを検出します。

| ソース | ローカルファイル | 用途 |
|---|---|---|
| 旧形式 / Composer | Cursor ユーザーデータ配下の `workspaceStorage/*/state.vscdb`、`globalStorage/state.vscdb` | ワークスペースの記録とグローバルな会話データ |
| Agent トランスクリプト | `~/.cursor/projects/**/agent-transcripts/**/*.jsonl` | 記録された会話本文とツール呼び出し |
| Store / CLI | `~/.cursor/chats/**/store.db` | セッションごとの Store 会話データ |
| ACP セッション | `~/.cursor/acp-sessions/**/store.db` | ACP ルート配下で検出される Store データ |

プラットフォーム別のパス、カスタムルート、WSL の設定は[保存場所](#storage)を参照してください。

形式によって含まれる情報は異なり、トランスクリプトにはタイムスタンプやツールの実行結果がない場合があります。許可された範囲内に利用可能な Store データベースとトランスクリプトが共存する場合、会話の主ソースはデータベースとなり、トランスクリプトは来歴として保持されます。ソースと時刻の来歴情報により、保存された値、推定値、不完全な表示を区別できます。

読み取りへの対応は、バックアップや移行への対応を意味しません。現在のバックアップは Composer のみを含み、Store-only および複数ソースを統合したセッションは移行できません。対応拡大は[ロードマップ](#roadmap)に記載しています。詳細は[互換性とデータ整合性の契約](./compatibility.md)を参照してください。

## 履歴を活用する三つの方法

### 見つける

```bash
cursor-history list --all
cursor-history search "connection pool"
cursor-history show 1
```

別のワークスペースにある会話でも、以前に解決した問題を探せます。ツールのインストール前に作成された、対応形式のローカル履歴も検索対象です。

### 保存する

```bash
cursor-history export 1
cursor-history backup
cursor-history migrate-session 1 /path/to/new/workspace --dry-run
```

読み取り可能な会話を Markdown または JSON にエクスポートし、Composer 履歴をバックアップできます。対応する Composer セッションを移動する前に、移行内容をプレビューできます。

### 再利用する

[Node.js ライブラリ](#library-api)を直接使うか、独立した [cursor-history-mcp](https://github.com/S2thend/cursor-history-mcp) サーバーに接続すると、MCP 対応アシスタントから既存の開発履歴を検索できます。

過去の判断、修正、プロンプト、ツール操作はすでにディスクに残っているかもしれません。次に必要になったとき、その文脈を見つけやすくします。

## 主な機能

- CLI と Node.js ライブラリの両方から利用できます。
- ワークスペースを横断してセッションを列挙し、会話本文をキーワード検索できます。
- ソースに含まれるメッセージ、差分、ツール引数・結果、AI の思考、時刻の来歴を確認できます。
- Markdown / JSON へのエクスポートに対応しています。
- 対応する Composer セッションを移動・コピーできます。
- Composer データベースのバックアップと復元に対応しています。
- macOS、Windows、Linux で利用できます。

<a id="installation"></a>
## インストール

npm からグローバルインストールします。

```bash
npm install -g cursor-history
cursor-history list
```

pnpm を使う場合：

```bash
pnpm add -g cursor-history
cursor-history list
```

ソースからビルドする場合：

```bash
git clone https://github.com/S2thend/cursor-history.git
cd cursor-history
npm install
npm run build
node dist/cli/index.js list
npm link
```

`npm link` の後は `cursor-history list` を直接実行できます。pnpm では `pnpm install` と `pnpm build` を使用してください。

## 動作要件

- Node.js 20.x または 22.x–26.x。Node 21 は非対応です。組み込み SQLite の読み取りには Node.js 22.5+ が必要です。
- Cursor IDE または Agent CLI で作成された、対応形式の既存ローカル履歴。

## SQLite ドライバーの設定

| ドライバー | 選択条件 | Node.js の対応範囲 |
|---|---|---|
| `node:sqlite` | 操作に必要な API がすべて利用可能な場合に優先 | 読み取りは 22.5 以降、オンラインバックアップは 22.16.0 / 23.8.0 以降 |
| `better-sqlite3` | インストール済みで必要な機能を備える場合の自動フォールバック | 対応メジャーバージョンは 20、22–26 |

ドライバーは操作ごとに実際の能力を確認して選択します。強制指定したドライバーからの自動切り替えはありません。必要な機能がない場合は、原因と対処方法を含む型付きエラーを返します。Store のスナップショット基盤の障害を、トランスクリプトへのフォールバックで隠すことはありません。

次の例は順に、ドライバーの強制指定とデバッグ出力です。

```bash
CURSOR_HISTORY_SQLITE_DRIVER=better-sqlite3 cursor-history list
CURSOR_HISTORY_SQLITE_DRIVER=node:sqlite cursor-history list
DEBUG=cursor-history:* cursor-history list
```

ライブラリからの設定：

```typescript
import { setDriver, getActiveDriver, listSessions } from 'cursor-history';

setDriver('better-sqlite3');
console.log(getActiveDriver());

const result = await listSessions({ sqliteDriver: 'node:sqlite' });
```

<a id="usage"></a>
## 使い方

### セッション一覧

既定では最近の 20 件を表示します。`--all` は全件、`--ids` は ID 付き、`-n` は件数指定、`--workspaces` はワークスペース一覧です。

```bash
cursor-history list
cursor-history list --all
cursor-history list --ids
cursor-history list -n 10
cursor-history list --workspaces
```

`list --workspaces` は範囲指定なしの検出であり、`--workspace` と併用できません。出力からパスを選び、以降のコマンドで同じ `--workspace` を指定してください。

### セッションの表示

`--short` はメッセージを短縮し、`--think` は思考全文、`--tool` はツール呼び出しの詳細、`--error` はエラー全文を表示します。`--only` はメッセージ種別をカンマ区切りで指定します。

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

自然言語の回答に構造化ツール呼び出しが含まれる場合は、`assistant` と `tool` の両方に一致します。ツールのみの記録は `tool` にだけ一致します。元のソースに存在しない内容は復元できません。

### 検索

`-n` で結果件数、`--context` で一致箇所の前後に表示する文字数を指定します。

```bash
cursor-history search "react hooks"
cursor-history search "api" -n 5
cursor-history search "error" --context 100
```

### エクスポート

既定は Markdown です。`--format json` で JSON、`--all` で全セッションを出力します。`--force` は既存の出力を上書きします。

```bash
cursor-history export 1
cursor-history export 1 -o ./my-chat.md
cursor-history export 1 --format json
cursor-history export --all -o ./exports/
cursor-history export 1 --force
```

<a id="migration"></a>
### セッションの移行

移行は条件を満たす Composer セッションに対応します。Store-only、統合ソース、曖昧なセッションは拒否されます。まず `--dry-run` でプレビューしてください。`--copy` は元の会話を残します。

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
### バックアップと復元

バックアップには Composer の `state.vscdb` データが含まれます。Store データベース、Agent トランスクリプト、ACP セッションストアは含まれません。それらの読み取り可能な会話は Markdown / JSON にエクスポートできますが、エクスポートは復元可能なバックアップではありません。

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

`restore --force` は検証済みの復元先を上書きします。整合性やパスの検査を無効にはしません。`--backup` を指定すると復元せずにアーカイブ内の履歴を読めます。

### グローバルオプション

```bash
cursor-history --json list
cursor-history --data-path /path/to/Cursor/User list
cursor-history --workspace /path/to/project list
cursor-history --workspace /path/to/project --include-cross-workspace-sources show 1
```

成功時の JSON は標準出力、致命的な JSON エラーは標準エラー出力に書かれ、終了コードは非ゼロになります。`--workspace` はメンバーシップだけでなく本文の読み取り範囲も制限します。`--include-cross-workspace-sources` は選択済み UUID の補完ソースに限って読み取り範囲を広げます。

## 表示できる内容

ソースに存在する会話、差分、ファイルの読み取り内容、検索条件、ターミナルコマンド、ツール結果・エラー、AI の思考、コードブロックや Mermaid 図を表示します。解決済みの各メッセージを順序どおりに一度ずつ表示し、連続する重複を省略して別々のツール操作や来歴を隠すことはありません。

時刻には `timestampSource` が付与され、推定時刻は概算として区別されます。同一 UUID が Composer と Store に存在する場合、対応する Composer ID を保ちつつ許可されたソースを統合します。既知の範囲外ソースは開かれず、その表示は不完全であることが明示されます。

<a id="storage"></a>
## Cursor のデータ保存場所

| プラットフォーム | Composer | Store |
|---|---|---|
| macOS | `~/Library/Application Support/Cursor/User/` | `~/.cursor/` |
| Windows | `%APPDATA%/Cursor/User/` | `%USERPROFILE%\.cursor\` |
| Linux / WSL | `~/.config/Cursor/User/` | `~/.cursor/` |

両方のストレージを自動検出します。Store ではセッションごとの `store.db` が主ソースです。必要なドライバー機能とスナップショット環境が正常な場合、データベースの欠落、利用可能なメッセージの欠如、ソースの破損・読み取り障害に対してトランスクリプトをフォールバックに使えます。ドライバー能力やスナップショット基盤の障害は致命的エラーとなります。利用可能なデータベースがある場合、トランスクリプトは置き換えられたソースの来歴としてのみ残ります。

`--data-path <path>` または `CURSOR_DATA_PATH` で Cursor データディレクトリを指定します。`CURSOR_STORE_ROOT` は Store のルートを独立して指定します。ルート自体のほか、`chats`、`projects`、`acp-sessions` 子ディレクトリも受け付け、同じルートに正規化します。

WSL から Windows 側の Store を読む場合、通常は `/mnt/c/Users/<windows-user>/.cursor` を使います。例：`CURSOR_STORE_ROOT=/mnt/c/Users/<windows-user>/.cursor cursor-history list --all`。WSL 内の Cursor agent が作成した履歴には、その環境の `~/.cursor` を指定してください。

Windows でインストールしたネイティブ `node_modules` を WSL で再利用しないでください。Linux 用 Node.js を使い、独立した WSL の依存関係ディレクトリでインストールしてからテストやビルドを実行します。ツールが依存関係を自動インストール・削除することはありません。

<a id="library-api"></a>
## ライブラリ API

CLI のほか、Node.js からも利用できます。保存して再利用する参照には、一覧が返した正確な UUID を使ってください。

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

ライブラリの読み取りインデックスは 0 始まり、CLI とライブラリの移行セレクターは 1 始まりです。インデックスはデータソース、ワークスペース範囲、一覧の状態に依存する一時的な値です。

v0.18.0 の検索結果では、`messageIndex` は完全な `session.messages` 内の位置、`offset` は元のメッセージ全文内の UTF-16 コード単位オフセット、`match` と文脈は元の完全な行を示します。v0.16/v0.17 の座標を保存していた場合は再計算してください。JSON エクスポートには 0 始まりの `index` が追加されます。

### 移行 API

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

### バックアップ API

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

`restored.warnings` を確認してください。破損した項目はスキップされ、復元されません。API の対象も Composer バックアップに限られます。

### 利用可能な関数

| 関数 | 説明 |
|---|---|
| `listSessions(config?)` | ページ分割付きでセッションを列挙 |
| `getSession(index, config?)` | インデックスまたは UUID でセッションを取得 |
| `searchSessions(query, config?)` | セッションを横断検索 |
| `exportSessionToJson(index, config?)` | セッションを JSON にエクスポート |
| `exportSessionToMarkdown(index, config?)` | セッションを Markdown にエクスポート |
| `exportAllSessionsToJson(config?)` | 全セッションを JSON にエクスポート |
| `exportAllSessionsToMarkdown(config?)` | 全セッションを Markdown にエクスポート |
| `migrateSession(config)` | 対応する Composer セッションを移動・コピー |
| `migrateWorkspace(config)` | ワークスペース間で対応する Composer セッションを移動・コピー |
| `createBackup(config?)` | Composer の会話履歴をバックアップ |
| `restoreBackup(config)` | バックアップから履歴を復元 |
| `validateBackup(path)` | バックアップの整合性を検証 |
| `listBackups(directory?)` | 利用可能なバックアップを列挙 |
| `getDefaultBackupDir()` | 既定のバックアップディレクトリを取得 |
| `getDefaultDataPath()` | プラットフォーム別の Cursor データパスを取得 |
| `setDriver(name)` | SQLite ドライバーを指定 |
| `getActiveDriver()` | 使用中の SQLite ドライバー名を取得 |

### 設定

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

`limit` と `offset` はページ分割、`context` は検索前後の行数、`messageFilter` は種別フィルターです。`backupPath` を指定するとライブデータの代わりにアーカイブを読みます。詳細な読み取り上限と追加設定は[互換性契約](./compatibility.md)を参照してください。

### エラー処理

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

データベースのロック時は Cursor を閉じて再試行してください。データやワークスペースが見つからない場合は、保存場所とプロジェクトパスを確認します。公開済みバックアップの権限エラーでは `pathIdentityVerified` を確認し、検証できないパスを根拠なく chmod・削除しないでください。`RESTORE_ROLLBACK_INCOMPLETE` は書き込み後の部分的な復元を示します。Cursor を停止し、信頼できるバックアップから手動で復旧してください。

<a id="compatibility"></a>
## 互換性と安全なアップグレード

[Compatibility and Data-Integrity Contract](./compatibility.md) が、ID、インデックスの範囲、ワークスペース I/O 境界、ソースの完全性、時刻の来歴、読み取り上限、権限、アップグレードの正式な仕様です。記述に差がある場合は、この英語の契約を優先してください。

- `Session.id` は Cursor のネイティブ UUID のままです。バイト単位かつ大文字・小文字を区別して照合し、物理的な保存先は別のメタデータで扱います。
- ワークスペースは正規化後の完全一致を優先し、その後で曖昧でない完全なパス要素の接尾辞を認めます。範囲外の本文は既定で開きません。
- `source: "global"` は完全で置き換え可能なデータ、`source: "workspace-fallback"` は完全な保存データを上書きすべきでない部分データです。`resolvedSource`、`sources`、`resolution` が実際の来歴を示します。
- v0.17 の Store / 統合データを増分保存している利用側は、v0.18.0 の修正経路を検証し、下流アーカイブをバックアップしてから更新してください。変更不要の保証は v0.16 Composer-only アーカイブから完全な Composer ベースの統合表示への移行に限定されます。v0.17 の不安定な合成 Store ID は維持されません。不完全なデータはバージョン固定、完全なソースからの再取得、または手動移行が必要です。
- 増分同期の境界に最大タイムスタンプを使わないでください。完全性と内容の変化を確認し、完全な履歴を不完全な結果で置き換えないでください。
- POSIX の一時スナップショットはディレクトリ `0700`、ファイル `0600` で作成され、成功・失敗時に片付けられます。新しいアーカイブは既定で `0600`、強制上書きは既存モードを維持します。`backup --shared` は最終アーカイブのみ共有権限を要求します。Windows の ACL 保護を POSIX と同じ保証とはしていません。
- バックアップ公開後の権限・片付けエラーは、アーカイブが存在しないことを意味しません。型付きエラーの公開状態とパス識別情報を確認し、未検証のパスを削除したり `--force` で盲目的に再試行したりしないでください。
- 復元は空の一覧、不正なパス、未宣言のペイロード、重複した宛先、安全でないリンクを拒否し、破損項目をスキップします。`--force` は検査を回避しません。書き込み後の失敗では自動ロールバックせず、`RESTORE_ROLLBACK_INCOMPLETE` の残存情報を返します。
- バックアップの `manifest.version` は `1.0.0` を維持し、追加の Composer 一覧は独立した `schemaVersion: 1` を使います。ワークスペース指定時は共有グローバルデータベースを展開せず、部分的な表示を返します。必要な一覧を持たない旧式の複数ワークスペースアーカイブは `BACKUP_WORKSPACE_SCOPE_METADATA_REQUIRED` で拒否します。

<a id="roadmap"></a>
## ロードマップ

読み取りに対応済みの新しいソースへ、バックアップ・復元と移行の範囲を広げます。以下は計画中の方向性であり、リリースバージョンや日付は未定です。

- [ ] **Store / ACP のバックアップと復元**：Store データベース、Agent トランスクリプト、関連メタデータを復元可能なアーカイブに含め、ソース関係の保持と往復の整合性を検証する。
- [ ] **Store-only セッションの移行**：ワークスペース間で移動・コピーし、ワークスペースとの関連付けとパスを更新して、Cursor から検出できることを確認する。
- [ ] **統合ソースのセッション移行**：Composer と Store の両方に存在するセッションを、ネイティブ ID とソース間の整合性を保って移動・コピーする。

実装までは[バックアップと復元](#backup-restore)は Composer のみ、[移行](#migration)は条件を満たす Composer セッションのみが対象です。読み取り可能な Store / ACP 会話は Markdown / JSON にエクスポートできますが、復元用アーカイブにはなりません。

[GitHub Issues](https://github.com/S2thend/cursor-history/issues) で利用例や再現可能なストレージの例を共有し、優先順位付けにご協力ください。

<a id="development"></a>
## 開発

### ビルドとテスト

```bash
npm install
npm run build
npm run typecheck
npm run lint
npm test
npm run test:watch
```

pnpm の場合は `pnpm install`、`pnpm build`、`pnpm typecheck`、`pnpm lint`、`pnpm test`、`pnpm test:watch` が使えます。

### npm へのリリース

GitHub Actions の npm trusted publishing を使用し、リポジトリの `NPM_TOKEN` secret は使いません。初回は npm の trusted publisher をこのリポジトリと `npm-publish.yml`、環境 `npm-release-verification` に関連付け、`npm publish` を許可します。GitHub 側にも同名の保護環境と必須レビュー担当者を設定してください。

1. バージョン付きメタデータとリリースノートを更新・検証し、リリース検査を完了して、クリーンなリビジョンを固定します。
2. タグが存在しないことを確認し、そのバージョンタグだけを push します。リビジョン固定前にリリースタグを push・移動しないでください。
3. ワークフローはソースと対応ランタイムを検証し、一度だけパッケージ化して、リビジョンと SHA-256 に紐付けます。公開ジョブは OIDC トークン要求前に保護環境で停止します。
4. チェックサムで識別される候補をダウンロードし、[リリース検証手順](./release-verification.md)に従って同一アーティファクトを検証した後で環境を承認します。
5. 保存された同一バイト列を npm provenance 付きで公開します。再ビルドや再パッケージ化はしません。

いずれかの検証が失敗した場合は公開できません。リリースタグを無断で強制移動しないでください。公開済みのバイト列がある場合は新しいバージョンが必要です。

## コントリビューション

不具合や機能要望は [Issues](https://github.com/S2thend/cursor-history/issues/new) へお寄せください。不具合には再現手順、期待する動作と実際の動作、OS と Node.js のバージョンを含めてください。

変更を提案する場合は、リポジトリを fork し、作業ブランチで変更を行い、`npm test` と `npm run lint` を実行してからコミット・push し、[Pull Request](https://github.com/S2thend/cursor-history/pulls) を作成してください。

## ライセンス

[MIT](../LICENSE)
