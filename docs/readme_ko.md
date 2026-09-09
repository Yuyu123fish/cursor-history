# Cursor History

<p align="center">
  <img src="readme-banner.png" alt="cursor-history: 모든 Cursor 기록을 하나의 인터페이스로. Composer, Agent transcripts, Store / ACP를 통합 CLI와 Node.js API로 연결합니다." width="960">
</p>

[![npm version](https://img.shields.io/npm/v/cursor-history.svg)](https://www.npmjs.com/package/cursor-history)
[![npm downloads](https://img.shields.io/npm/dm/cursor-history.svg)](https://www.npmjs.com/package/cursor-history)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js](https://img.shields.io/badge/Node.js-20%2C%2022--26-green.svg)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0%2B-blue.svg)](https://www.typescriptlang.org/)

🇺🇸 [English](../README.md) | 🇨🇳 [中文](./readme_zh.md) | 🇫🇷 [Français](./readme_fr.md) | 🇪🇸 [Español](./readme_es.md) | 🇯🇵 [日本語](./readme_ja.md) | 🇰🇷 [한국어](./readme_ko.md) | 🇺🇦 [Українська](./readme_uk.md)

**모든 Cursor 기록을 하나의 인터페이스로.**

Cursor 대화는 워크스페이스, IDE 데이터베이스, Agent 트랜스크립트, 새로운 CLI / ACP 세션 저장소 등에 나뉘어 있을 수 있습니다. `cursor-history`는 지원되는 로컬 소스를 자동으로 찾아 하나의 CLI와 Node.js 라이브러리로 접근할 수 있게 합니다.

여러 워크스페이스의 대화 본문을 검색하고, 메시지와 기록된 도구 활동을 살펴본 뒤 Markdown 또는 JSON으로 내보낼 수 있습니다. Composer 기록을 백업·복원하거나, 프로젝트를 옮길 때 지원되는 Composer 세션을 마이그레이션할 수도 있습니다.

**이미 몇 달 치 Cursor 기록이 있나요? 사전 수집이나 인덱스 설정은 필요하지 않습니다.** 검색은 로컬에서 실행되며 임베딩이나 API 키가 필요 없습니다.

<a id="quick-start"></a>
**MCP 서버 인터페이스를 선호하시나요?** [cursor-history-mcp](https://github.com/S2thend/cursor-history-mcp#quick-start)를 연결하면 기록 기능을 MCP 도구로 제공할 수 있습니다. 에이전트는 이 프로젝트의 CLI나 Node.js API를 직접 호출할 수도 있습니다. 워크플로에 맞는 인터페이스를 선택하세요. 패키지는 별도로 출시되므로 [MCP 저장소 호환성과 릴리스 안내](https://github.com/S2thend/cursor-history-mcp#compatibility)를 확인하세요.

## 빠른 시작

```bash
npm install -g cursor-history

cursor-history list --all
cursor-history search "authentication"
cursor-history show 1
cursor-history export 1
cursor-history backup
```

Node.js 20.x 또는 22.x–26.x와 기존 로컬 Cursor 기록이 필요합니다. 전역 설치 없이 사용해 보려면 `npx cursor-history list --all`을 실행하세요.

`show`와 `export`의 숫자는 같은 데이터 소스와 워크스페이스 범위에서 얻은 목록을 기준으로 합니다. 저장해 두고 다시 실행할 명령에는 세션 UUID를 사용하세요. `backup`은 Composer 데이터베이스를 보관하며 Store 데이터베이스나 트랜스크립트는 포함하지 않습니다.

[도구 비교](#comparison) · [설치](#installation) · [사용법](#usage) · [출력 예시](../README.md#example-output) · [라이브러리 API](#library-api) · [로드맵](#roadmap) · [호환성](#compatibility)

<a id="comparison"></a>

## cursor-history를 선택하는 이유

`cursor-history`는 하나의 CLI와 Node.js API로 Cursor 기록을 찾고, 살펴보고, 보존하고, 재사용하려는 경우에 적합합니다.

- **여러 Cursor 소스를 하나의 인터페이스로** — 지원되는 Composer, Agent 트랜스크립트, Store / CLI, ACP 소스를 읽습니다.
- **검색부터 보존까지** — 여러 워크스페이스의 대화 본문을 검색하고, 사용 가능한 diff와 도구 활동을 확인하며, Markdown / JSON으로 내보냅니다. Composer 데이터를 백업·복원하고, 조건을 충족하는 Composer 세션의 마이그레이션을 `--dry-run`으로 미리 확인할 수 있습니다.
- **기존 작업 흐름에 통합** — 직접 만든 도구에서 Node.js API를 사용하거나 별도 [MCP 연동 프로젝트](https://github.com/S2thend/cursor-history-mcp#compatibility)를 연결할 수 있습니다. [WSL 설정](#storage)에 따라 Windows 쪽이나 WSL 내부의 Store 데이터도 읽을 수 있습니다.

| 비교 항목 | **cursor-history(이 프로젝트)** | [deja-vu](https://github.com/vshulcz/deja-vu) | [cursaves](https://github.com/Callum-Ward/cursaves) | [johnlindquist/cursor-history](https://github.com/johnlindquist/cursor-history) |
|---|---|---|---|---|
| 주요 용도 | Cursor 기록 조회·관리; CLI + Node.js API | 에이전트 간 기억 공유 | Git / S3 동기화 | 탐색·내보내기·클립보드 |
| 문서에 명시된 Cursor 소스 | Composer, Agent 트랜스크립트, Store / CLI, ACP | IDE SQLite, CLI 트랜스크립트¹ | 워크스페이스·전역 SQLite | 문서에 명시되지 않음 |
| 검색 | 여러 워크스페이스의 대화 본문 검색 | 에이전트 간 인덱스 검색 | 문서에 명시되지 않음 | 제목 유사 검색 |
| 기록 보존·이동 | Composer 백업·복원; 조건을 충족하는 Composer 마이그레이션 | 기억 동기화·인계 | 스냅샷 복원·워크스페이스 간 복사 | 문서에 명시되지 않음 |

문서 확인일: **2026-09-09**. 링크된 README와 ¹ [deja-vu의 Cursor 형식 문서](https://github.com/vshulcz/deja-vu/blob/main/docs/registry/cursor.md)를 기준으로 비교했으며, 각 도구를 실행한 비교 테스트는 아닙니다. “문서에 명시되지 않음”은 확인한 자료에 설명이 없다는 뜻이며, 미지원으로 단정하지 않습니다.

이 프로젝트의 백업·복원은 Composer 데이터베이스만 대상으로 합니다. 마이그레이션은 조건을 충족하는 Composer 세션만 지원하며, Store 전용 세션이나 여러 소스를 병합한 세션은 제외됩니다. 읽을 수 있는 Store / ACP 세션과 트랜스크립트는 내보낼 수 있지만, 내보낸 파일은 복원 가능한 백업이 아닙니다. [호환성 계약](./compatibility.md)을 참고하세요.

## 이 도구가 필요한 이유

Cursor로 문제를 해결했던 기억은 있어도 어느 프로젝트, 세션 또는 화면에서 대화했는지 떠오르지 않을 수 있습니다. 로컬 기록에서 키워드를 검색하면 그 대화를 다시 찾을 수 있습니다.

Cursor의 [Agent CLI](https://cursor.com/docs/cli/reference/parameters)는 CLI 세션을 찾거나 재개하는 `agent ls`, `agent resume`, `agent --resume=<id>`를 제공합니다. `cursor-history`는 워크스페이스와 저장 형식을 넘나들며 지원되는 로컬 기록을 읽고 관리하는 기능을 제공합니다.

| 하고 싶은 작업 | 시작 방법 |
|---|---|
| Cursor Agent CLI에서 대화 재개 | Cursor의 `agent ls` 또는 `agent --resume=<id>` |
| 여러 워크스페이스의 대화 본문에서 문구 검색 | `cursor-history search "connection pool"` |
| 발견한 로컬 세션 조회·내보내기 | `cursor-history show 1` / `cursor-history export 1` |
| Composer 기록 백업·복원 | [백업과 복원](#backup-restore) |
| 지원되는 Composer 세션을 다른 워크스페이스로 이동 | [세션 마이그레이션](#migration) |
| 자신의 애플리케이션에서 기록 활용 | [Node.js API](#library-api) |

## 여러 세대의 Cursor 저장 형식 지원

Cursor 버전과 사용 인터페이스에 따라 같은 컴퓨터에 여러 형태의 로컬 기록이 남을 수 있습니다. 다음과 같은 지원 소스를 찾습니다.

| 소스 | 로컬 파일 | 용도 |
|---|---|---|
| 이전 형식 / Composer | Cursor 사용자 데이터 디렉터리의 `workspaceStorage/*/state.vscdb`, `globalStorage/state.vscdb` | 워크스페이스 기록과 전역 대화 데이터 |
| Agent 트랜스크립트 | `~/.cursor/projects/**/agent-transcripts/**/*.jsonl` | 기록된 대화 본문과 도구 호출 |
| Store / CLI | `~/.cursor/chats/**/store.db` | 세션별 Store 대화 데이터 |
| ACP 세션 | `~/.cursor/acp-sessions/**/store.db` | ACP 루트 아래에서 발견한 Store 데이터 |

플랫폼별 경로, 사용자 지정 루트 및 WSL 설정은 [데이터 저장 위치](#storage)를 참고하세요.

형식마다 포함하는 필드가 다릅니다. 트랜스크립트에는 타임스탬프나 도구 실행 결과가 없을 수 있습니다. 허용된 범위 안에 사용 가능한 Store 데이터베이스와 트랜스크립트가 함께 있으면 데이터베이스가 Store 대화의 주 소스가 되고 트랜스크립트는 출처 정보로 남습니다. 소스와 시간의 출처 정보를 통해 저장된 값, 추정값, 불완전한 뷰를 구분할 수 있습니다.

읽기 지원이 백업이나 마이그레이션 지원을 의미하지는 않습니다. 현재 백업은 Composer만 포함하며, Store-only 세션과 여러 소스를 병합한 세션은 마이그레이션할 수 없습니다. 지원 확대는 [로드맵](#roadmap)에 있습니다. 자세한 범위는 [호환성 및 데이터 무결성 계약](./compatibility.md)을 참고하세요.

## 기록을 활용하는 세 가지 방법

### 찾기

```bash
cursor-history list --all
cursor-history search "connection pool"
cursor-history show 1
```

다른 워크스페이스에 있더라도 이전에 문제를 해결했던 대화를 찾을 수 있습니다. 도구를 설치하기 전에 만들어진 지원 형식의 로컬 기록도 검색할 수 있습니다.

### 보관하기

```bash
cursor-history export 1
cursor-history backup
cursor-history migrate-session 1 /path/to/new/workspace --dry-run
```

읽을 수 있는 세션을 Markdown 또는 JSON으로 내보내고 Composer 기록을 백업하세요. 지원되는 Composer 세션을 옮기기 전에 마이그레이션 내용을 미리 확인할 수 있습니다.

### 재사용하기

[Node.js 라이브러리](#library-api)를 직접 사용하거나 별도 서버인 [cursor-history-mcp](https://github.com/S2thend/cursor-history-mcp)를 연결하면 MCP 지원 어시스턴트가 기존 개발 기록을 검색할 수 있습니다.

몇 달 동안의 결정, 수정, 프롬프트, 도구 활동이 이미 디스크에 남아 있을 수 있습니다. 다음에 필요할 때 그 맥락을 더 쉽게 찾으세요.

## 주요 기능

- CLI로 실행하거나 Node.js 라이브러리로 가져와 사용할 수 있습니다.
- 여러 워크스페이스의 세션을 나열하고 대화 본문을 키워드로 검색합니다.
- 소스에 남아 있는 메시지, 변경 diff, 도구 인수·결과, AI의 사고 과정과 시간 출처를 확인합니다.
- Markdown / JSON 내보내기를 지원합니다.
- 지원되는 Composer 세션을 이동·복사합니다.
- Composer 데이터베이스를 백업·복원합니다.
- **크로스 플랫폼 및 WSL 지원** - macOS, Windows, Linux, WSL에서 사용할 수 있습니다. WSL에서 데이터 경로를 설정하면 Windows 쪽이나 WSL 내부에 저장된 Cursor Store 세션을 읽을 수 있습니다. [WSL 설정](#storage)을 참고하세요.

<a id="installation"></a>
## 설치

npm으로 전역 설치:

```bash
npm install -g cursor-history
cursor-history list
```

pnpm을 사용하는 경우:

```bash
pnpm add -g cursor-history
cursor-history list
```

소스에서 빌드:

```bash
git clone https://github.com/S2thend/cursor-history.git
cd cursor-history
npm install
npm run build
node dist/cli/index.js list
npm link
```

`npm link` 이후에는 `cursor-history list`를 직접 실행할 수 있습니다. pnpm을 사용한다면 `pnpm install`과 `pnpm build`를 실행하세요.

## 요구 사항

- Node.js 20.x 또는 22.x–26.x. Node 21은 지원하지 않습니다. 내장 SQLite 읽기 기능은 Node.js 22.5+에서 사용할 수 있습니다.
- Cursor IDE 또는 Agent CLI에서 생성한, 지원 형식의 기존 로컬 기록.

## SQLite 드라이버 설정

| 드라이버 | 선택 조건 | Node.js 지원 범위 |
|---|---|---|
| `node:sqlite` | 작업에 필요한 모든 API가 있을 때 우선 사용 | 읽기는 22.5부터, 온라인 백업은 22.16.0 / 23.8.0부터 |
| `better-sqlite3` | 설치되어 있고 필요한 기능을 제공할 때 자동 대체 | 지원 메이저 버전은 20, 22–26 |

드라이버는 작업마다 실제 기능을 확인해 선택합니다. 강제로 지정한 드라이버는 다른 드라이버로 자동 전환하지 않습니다. 필요한 기능이 없으면 원인과 해결 방법이 담긴 타입이 지정된 오류를 반환합니다. Store 스냅샷 기반 시설의 장애를 트랜스크립트로 대체해 숨기지 않습니다.

다음은 드라이버 강제 지정과 디버그 출력 예시입니다.

```bash
CURSOR_HISTORY_SQLITE_DRIVER=better-sqlite3 cursor-history list
CURSOR_HISTORY_SQLITE_DRIVER=node:sqlite cursor-history list
DEBUG=cursor-history:* cursor-history list
```

라이브러리에서 설정:

```typescript
import { setDriver, getActiveDriver, listSessions } from 'cursor-history';

setDriver('better-sqlite3');
console.log(getActiveDriver());

const result = await listSessions({ sqliteDriver: 'node:sqlite' });
```

<a id="usage"></a>
## 사용법

### 세션 목록

기본값은 최근 20개입니다. `--all`은 전체, `--ids`는 ID 표시, `-n`은 개수 지정, `--workspaces`는 워크스페이스 목록입니다.

```bash
cursor-history list
cursor-history list --all
cursor-history list --ids
cursor-history list -n 10
cursor-history list --workspaces
```

`list --workspaces`는 범위를 지정하지 않는 탐색이므로 `--workspace`와 함께 사용할 수 없습니다. 출력에서 경로를 고른 다음 이후 명령에 같은 `--workspace` 값을 사용하세요.

### 세션 보기

`--short`는 메시지를 줄여 표시합니다. `--think`는 사고 과정 전체, `--tool`은 도구 호출 상세, `--error`는 오류 전체를 표시합니다. `--only`에는 메시지 종류를 쉼표로 구분해 지정합니다.

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

자연어 답변에 구조화된 도구 호출이 포함되면 `assistant`와 `tool` 필터에 모두 일치합니다. 도구만 포함된 기록은 `tool`에만 일치합니다. 원본 소스에 없는 내용은 복원할 수 없습니다.

### 검색

`-n`으로 결과 개수, `--context`로 일치한 부분 앞뒤에 표시할 문자 수를 지정합니다.

```bash
cursor-history search "react hooks"
cursor-history search "api" -n 5
cursor-history search "error" --context 100
```

### 내보내기

기본 형식은 Markdown입니다. `--format json`은 JSON, `--all`은 모든 세션을 내보냅니다. `--force`는 기존 출력을 덮어씁니다.

```bash
cursor-history export 1
cursor-history export 1 -o ./my-chat.md
cursor-history export 1 --format json
cursor-history export --all -o ./exports/
cursor-history export 1 --force
```

<a id="migration"></a>
### 세션 마이그레이션

조건을 충족하는 Composer 세션만 지원합니다. Store-only, 병합된 소스, 모호한 세션은 거부됩니다. 먼저 `--dry-run`으로 내용을 확인하세요. `--copy`는 원본 대화를 남깁니다.

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
### 백업과 복원

백업에는 Composer의 `state.vscdb` 데이터가 포함됩니다. Store 데이터베이스, Agent 트랜스크립트, ACP 세션 저장소는 포함하지 않습니다. 이들 소스의 읽을 수 있는 대화는 Markdown / JSON으로 내보낼 수 있지만, 내보낸 파일은 복원 가능한 백업 아카이브가 아닙니다.

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

`restore --force`는 검증된 복원 대상을 덮어쓰며 무결성 또는 경로 검사를 해제하지 않습니다. `--backup`으로 아카이브를 지정하면 복원하지 않고 기록을 읽을 수 있습니다.

### 전역 옵션

```bash
cursor-history --json list
cursor-history --data-path /path/to/Cursor/User list
cursor-history --workspace /path/to/project list
cursor-history --workspace /path/to/project --include-cross-workspace-sources show 1
```

성공한 JSON 결과는 표준 출력, 치명적인 JSON 오류는 표준 오류 출력으로 기록되며 종료 코드는 0이 아닙니다. `--workspace`는 소속뿐 아니라 본문을 읽는 범위도 제한합니다. `--include-cross-workspace-sources`는 이미 선택된 UUID의 보완 소스에 한해 읽기 범위를 넓힙니다.

## 확인할 수 있는 내용

소스에 존재하는 대화, diff, 파일 읽기 내용, 검색 조건, 터미널 명령, 도구 결과·오류, AI의 사고 과정, 코드 블록과 Mermaid 다이어그램을 표시합니다. 해석된 각 메시지는 순서대로 한 번씩 표시합니다. 연속으로 반복되는 메시지를 생략해 서로 다른 도구 작업이나 출처가 가려지지 않도록 합니다.

시간에는 `timestampSource`가 제공되고, 추정 시각은 대략적인 값으로 구분됩니다. 같은 UUID가 Composer와 Store에 모두 있으면 호환되는 Composer ID를 보존하며 허용된 소스를 통합합니다. 범위 밖에 있는 것으로 알려진 소스는 열지 않으며 결과가 불완전하다고 표시합니다.

<a id="storage"></a>
## Cursor 데이터 저장 위치

| 플랫폼 | Composer | Store |
|---|---|---|
| macOS | `~/Library/Application Support/Cursor/User/` | `~/.cursor/` |
| Windows | `%APPDATA%/Cursor/User/` | `%USERPROFILE%\.cursor\` |
| Linux / WSL | `~/.config/Cursor/User/` | `~/.cursor/` |

두 저장소를 자동으로 찾습니다. Store에서는 세션별 `store.db`가 주 소스입니다. 필요한 드라이버 기능과 스냅샷 환경이 정상이라면 데이터베이스 누락, 사용 가능한 메시지 없음, 소스 손상·읽기 실패 시 트랜스크립트를 대체 소스로 사용할 수 있습니다. 드라이버 기능이나 스냅샷 기반 시설의 문제는 치명적인 오류입니다. 사용 가능한 데이터베이스가 있으면 트랜스크립트는 대체된 소스의 출처 정보로만 남습니다.

`--data-path <path>` 또는 `CURSOR_DATA_PATH`로 Cursor 데이터 디렉터리를 지정하세요. `CURSOR_STORE_ROOT`로 Store 루트를 별도로 설정할 수 있습니다. 루트 자체나 `chats`, `projects`, `acp-sessions` 하위 디렉터리를 받아 같은 루트로 정규화합니다.

WSL에서 Windows 쪽 Store 데이터를 읽을 때는 보통 `/mnt/c/Users/<windows-user>/.cursor`를 사용합니다. 예: `CURSOR_STORE_ROOT=/mnt/c/Users/<windows-user>/.cursor cursor-history list --all`. WSL 안에서 실행한 Cursor agent가 만든 기록은 해당 환경의 `~/.cursor`를 사용하세요.

Windows에서 설치한 네이티브 `node_modules`를 WSL에서 재사용하지 마세요. Linux Node.js를 사용해 별도 WSL 의존성 디렉터리에 설치한 후 테스트나 빌드를 실행해야 합니다. 도구는 의존성을 자동 설치하거나 삭제하지 않습니다.

<a id="library-api"></a>
## 라이브러리 API

CLI 외에 Node.js에서도 사용할 수 있습니다. 저장해 두고 다시 사용하는 참조에는 목록에서 반환된 정확한 UUID를 사용하세요.

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

라이브러리 읽기 인덱스는 0부터, CLI와 라이브러리 마이그레이션 선택자는 1부터 시작합니다. 인덱스는 데이터 소스, 워크스페이스 범위, 목록 상태에 따라 달라지는 임시 값입니다.

v0.18.0 검색 결과의 `messageIndex`는 전체 `session.messages` 배열 안의 위치이며, `offset`은 원본 메시지 전체에서의 UTF-16 코드 단위 위치입니다. `match`와 문맥은 원본의 온전한 줄을 나타냅니다. v0.16/v0.17의 좌표를 저장했다면 다시 계산하세요. JSON 내보내기에는 0부터 시작하는 `index`가 추가됩니다.

### 마이그레이션 API

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

### 백업 API

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

`restored.warnings`를 확인하세요. 손상된 항목은 건너뛰며 복원하지 않습니다. API 역시 Composer 백업만 대상으로 합니다.

### 사용 가능한 함수

| 함수 | 설명 |
|---|---|
| `listSessions(config?)` | 페이지를 나누어 세션 목록 조회 |
| `getSession(index, config?)` | 인덱스 또는 UUID로 세션 조회 |
| `searchSessions(query, config?)` | 여러 세션에서 검색 |
| `exportSessionToJson(index, config?)` | 세션을 JSON으로 내보내기 |
| `exportSessionToMarkdown(index, config?)` | 세션을 Markdown으로 내보내기 |
| `exportAllSessionsToJson(config?)` | 모든 세션을 JSON으로 내보내기 |
| `exportAllSessionsToMarkdown(config?)` | 모든 세션을 Markdown으로 내보내기 |
| `migrateSession(config)` | 지원되는 Composer 세션 이동·복사 |
| `migrateWorkspace(config)` | 워크스페이스 간 지원되는 Composer 세션 이동·복사 |
| `createBackup(config?)` | Composer 대화 기록 백업 |
| `restoreBackup(config)` | 백업에서 기록 복원 |
| `validateBackup(path)` | 백업 무결성 검증 |
| `listBackups(directory?)` | 사용 가능한 백업 목록 조회 |
| `getDefaultBackupDir()` | 기본 백업 디렉터리 경로 조회 |
| `getDefaultDataPath()` | 플랫폼별 Cursor 데이터 경로 조회 |
| `setDriver(name)` | SQLite 드라이버 지정 |
| `getActiveDriver()` | 현재 SQLite 드라이버 이름 조회 |

### 설정

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

`limit`과 `offset`은 페이지 분할, `context`는 검색 문맥의 줄 수, `messageFilter`는 종류 필터입니다. `backupPath`를 설정하면 라이브 데이터 대신 아카이브를 읽습니다. 읽기 한도와 추가 설정은 [호환성 계약](./compatibility.md)을 참고하세요.

### 오류 처리

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

데이터베이스가 잠겨 있으면 Cursor를 닫고 다시 시도하세요. 데이터나 워크스페이스를 찾지 못하면 저장 위치와 프로젝트 경로를 확인하세요. 이미 생성된 백업의 권한 오류에서는 `pathIdentityVerified`를 확인하고, 검증되지 않은 경로의 권한을 변경하거나 삭제하지 마세요. `RESTORE_ROLLBACK_INCOMPLETE`는 쓰기 이후 복원이 부분적으로 진행된 상태임을 나타냅니다. Cursor를 중지하고 신뢰할 수 있는 백업에서 수동으로 복구하세요.

<a id="compatibility"></a>
## 호환성과 안전한 업그레이드

[Compatibility and Data-Integrity Contract](./compatibility.md)는 ID, 인덱스 범위, 워크스페이스 I/O 경계, 소스 완전성, 시간 출처, 읽기 한도, 권한 및 업그레이드에 대한 공식 규약입니다. 설명이 다르면 이 영문 계약을 우선합니다.

- `Session.id`는 Cursor의 네이티브 UUID를 유지합니다. 바이트 단위로 대소문자를 구분해 비교하며 실제 저장 위치는 별도 메타데이터로 다룹니다.
- 워크스페이스는 정규화한 정확한 경로 일치를 우선하고, 이후 모호하지 않은 완전한 경로 구성 요소의 접미사 일치를 허용합니다. 범위 밖 본문은 기본적으로 읽지 않습니다.
- `source: "global"`은 완전하고 교체에 사용할 수 있는 데이터입니다. `source: "workspace-fallback"`은 저장된 완전한 데이터를 덮어쓰면 안 되는 부분 데이터입니다. `resolvedSource`, `sources`, `resolution`은 실제 출처를 나타냅니다.
- v0.17의 Store / 병합 데이터를 증분 저장하는 사용자는 v0.18.0의 수정 경로를 검증하고 하위 아카이브를 백업한 다음 업데이트해야 합니다. 소비자 코드 변경 없이 업그레이드할 수 있다는 보장은 v0.16 Composer-only 아카이브에서 완전한 Composer 기반 병합 뷰로 전환하는 경우로 제한됩니다. v0.17의 불안정한 합성 Store ID는 유지하지 않습니다. 불완전한 데이터는 버전 고정, 완전한 소스에서 재수집 또는 수동 마이그레이션이 필요합니다.
- 최대 타임스탬프를 증분 동기화 경계로 사용하지 마세요. 완전성과 내용 변화를 확인하고, 완전한 기록을 불완전한 결과로 교체하지 마세요.
- POSIX 임시 스냅샷 디렉터리는 `0700`, 파일은 `0600`이며 성공과 실패 모두에서 정리합니다. 새 아카이브의 기본 권한은 `0600`이고 강제 덮어쓰기는 기존 모드를 유지합니다. `backup --shared`는 최종 아카이브에만 공유 권한을 요청합니다. Windows ACL 보호를 POSIX와 동일하게 보장하지는 않습니다.
- 백업이 최종 경로에 배치된 뒤 권한 또는 정리 오류가 발생해도 아카이브가 없다는 뜻은 아닙니다. 오류의 게시 상태와 경로 식별 정보를 확인하고, 검증되지 않은 경로를 삭제하거나 `--force`로 무작정 재시도하지 마세요.
- 복원은 빈 목록, 잘못된 경로, 선언되지 않은 페이로드, 중복 대상, 안전하지 않은 링크를 거부하고 손상된 항목을 건너뜁니다. `--force`는 검사를 우회하지 않습니다. 쓰기 후 실패 시 자동 롤백하지 않고 `RESTORE_ROLLBACK_INCOMPLETE`로 잔여 상태를 반환합니다.
- 백업의 `manifest.version`은 `1.0.0`을 유지하며 선택적 Composer 목록은 별도의 `schemaVersion: 1`을 사용합니다. 워크스페이스 범위를 지정한 읽기는 공유 전역 데이터베이스를 추출하지 않으며 부분 뷰를 반환합니다. 필요한 목록이 없는 구형 다중 워크스페이스 아카이브는 `BACKUP_WORKSPACE_SCOPE_METADATA_REQUIRED`로 거부합니다.

<a id="roadmap"></a>
## 로드맵

이미 읽기를 지원하는 새 소스로 백업·복원과 마이그레이션 범위를 확대합니다. 다음은 계획된 방향이며 릴리스 버전이나 날짜는 아직 정해지지 않았습니다.

- [ ] **Store / ACP 백업과 복원**: Store 데이터베이스, Agent 트랜스크립트, 관련 메타데이터를 복원 가능한 아카이브에 포함하고 소스 관계 보존과 백업·복원 왕복 일관성을 검증합니다.
- [ ] **Store-only 세션 마이그레이션**: 워크스페이스 간 세션을 이동·복사하고 워크스페이스 연결 및 경로 참조를 갱신한 뒤 Cursor가 세션을 발견하는지 확인합니다.
- [ ] **병합 소스 세션 마이그레이션**: Composer와 Store 양쪽에 존재하는 세션을 네이티브 ID와 소스 간 일관성을 유지하며 이동·복사합니다.

구현 전까지 [백업과 복원](#backup-restore)은 Composer만, [마이그레이션](#migration)은 조건을 충족하는 Composer 세션만 지원합니다. 읽을 수 있는 Store / ACP 대화는 Markdown / JSON으로 내보낼 수 있지만 복원용 아카이브는 아닙니다.

[GitHub Issues](https://github.com/S2thend/cursor-history/issues)에 사용 사례와 재현 가능한 저장소 예시를 공유해 우선순위 결정에 도움을 주세요.

<a id="development"></a>
## 개발

### 빌드와 테스트

```bash
npm install
npm run build
npm run typecheck
npm run lint
npm test
npm run test:watch
```

pnpm에서는 `pnpm install`, `pnpm build`, `pnpm typecheck`, `pnpm lint`, `pnpm test`, `pnpm test:watch`를 사용할 수 있습니다.

### npm 릴리스

GitHub Actions를 통한 npm trusted publishing을 사용하며 저장소의 `NPM_TOKEN` secret은 사용하지 않습니다. 최초 설정 시 npm trusted publisher를 이 저장소, `npm-publish.yml`, `npm-release-verification` 환경에 연결하고 `npm publish` 작업을 허용합니다. GitHub에도 같은 이름의 보호 환경과 필수 검토자를 설정하세요.

1. 버전이 포함된 메타데이터와 릴리스 노트를 갱신·검증하고 릴리스 검사를 완료한 뒤 깨끗한 리비전을 고정합니다.
2. 버전 태그가 없는지 확인하고 해당 태그만 push합니다. 리비전 고정 전에 릴리스 태그를 push하거나 옮기지 마세요.
3. 워크플로는 소스와 지원 런타임을 검증하고 한 번만 패키징하여 리비전 및 SHA-256과 연결합니다. 공개 작업은 OIDC 토큰을 요청하기 전에 보호 환경에서 멈춥니다.
4. 체크섬으로 식별되는 후보를 내려받아 [릴리스 검증 절차](./release-verification.md)에 따라 동일한 아티팩트를 검증한 뒤 환경을 승인합니다.
5. 보관된 동일 바이트를 npm provenance와 함께 공개합니다. 다시 빌드하거나 패키징하지 않습니다.

어느 검증이든 실패하면 공개할 수 없습니다. 릴리스 태그를 조용히 강제 이동하지 마세요. 이미 공개한 바이트가 있다면 새 버전이 필요합니다.

## 기여하기

버그나 기능 요청은 [Issues](https://github.com/S2thend/cursor-history/issues/new)에 남겨 주세요. 버그에는 재현 단계, 기대 동작과 실제 동작, OS와 Node.js 버전을 포함해 주세요.

변경을 제안하려면 저장소를 fork하고 작업 브랜치에서 수정한 뒤 `npm test`와 `npm run lint`를 실행하세요. 커밋·push 후 [Pull Request](https://github.com/S2thend/cursor-history/pulls)를 만들어 주세요.

## 라이선스

[MIT](../LICENSE)
