# Cursor History

<p align="center">
  <img src="readme-banner.png" alt="cursor-history : une interface pour tout votre historique Cursor. Composer, Agent transcripts et Store / ACP alimentent un CLI et une API Node.js unifiés." width="960">
</p>

[![npm version](https://img.shields.io/npm/v/cursor-history.svg)](https://www.npmjs.com/package/cursor-history)
[![npm downloads](https://img.shields.io/npm/dm/cursor-history.svg)](https://www.npmjs.com/package/cursor-history)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js](https://img.shields.io/badge/Node.js-20%2C%2022--26-green.svg)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0%2B-blue.svg)](https://www.typescriptlang.org/)

🇺🇸 [English](../README.md) | 🇨🇳 [中文](./readme_zh.md) | 🇫🇷 [Français](./readme_fr.md) | 🇪🇸 [Español](./readme_es.md) | 🇯🇵 [日本語](./readme_ja.md) | 🇰🇷 [한국어](./readme_ko.md) | 🇺🇦 [Українська](./readme_uk.md)

**Une interface pour tout votre historique Cursor.**

Les conversations Cursor peuvent être réparties entre espaces de travail, bases de données de l'IDE, transcriptions Agent et magasins de sessions CLI / ACP plus récents. `cursor-history` découvre les sources locales prises en charge et les rend accessibles via un CLI et une bibliothèque Node.js.

Recherchez dans le contenu des conversations de plusieurs espaces de travail, consultez les messages et l'activité des outils disponible, puis exportez en Markdown ou JSON. Sauvegardez et restaurez l'historique Composer, ou migrez les sessions Composer prises en charge lorsque vos projets changent d'emplacement.

**Vous avez déjà des mois d'historique Cursor ? Aucune capture préalable ni configuration d'index n'est nécessaire.** La recherche s'exécute localement, sans embeddings ni clé API.

**Vous préférez une interface serveur MCP ?** Connectez [cursor-history-mcp](https://github.com/S2thend/cursor-history-mcp#quick-start) pour exposer l'historique sous forme d'outils MCP. Les agents peuvent aussi appeler directement le CLI ou l'API Node.js de ce projet : choisissez l'interface adaptée à votre workflow. Les paquets sont publiés indépendamment ; vérifiez la [compatibilité et les informations de publication MCP](https://github.com/S2thend/cursor-history-mcp#compatibility).

## Démarrage rapide

```bash
npm install -g cursor-history

cursor-history list --all
cursor-history search "authentication"
cursor-history show 1
cursor-history export 1
cursor-history backup
```

Nécessite Node.js 20.x ou 22.x–26.x et un historique Cursor local existant. Pour essayer sans installation permanente : `npx cursor-history list --all`.

Les nombres utilisés avec `show` et `export` correspondent à la liste issue de la même source et du même périmètre d'espace de travail ; utilisez l'UUID de session pour les commandes conservées. `backup` archive les bases Composer, pas les bases Store ni les transcriptions.

[Comparaison](#comparison) · [Installation](#installation) · [Utilisation](#utilisation) · [Exemples de sortie](../README.md#example-output) · [API Bibliothèque](#api-bibliothèque) · [Feuille de route](#feuille-de-route) · [Compatibilité](#compatibilité-et-mises-à-niveau)

<a id="comparison"></a>

## Pourquoi choisir cursor-history ?

Choisissez `cursor-history` pour retrouver, consulter, conserver et réutiliser l'historique Cursor via un même CLI et une API Node.js.

- **Plusieurs sources Cursor, une interface** — Lisez les sources prises en charge : Composer, transcriptions Agent, Store / CLI et ACP.
- **De la recherche à la conservation** — Recherchez dans le texte des conversations de plusieurs espaces de travail, consultez les diff et activités d'outils disponibles, puis exportez en Markdown ou JSON. Sauvegardez et restaurez les données Composer, ou prévisualisez les migrations Composer admissibles avec `--dry-run`.
- **Adapté à votre workflow** — Utilisez l'API Node.js dans vos outils, connectez le [projet MCP distinct](https://github.com/S2thend/cursor-history-mcp#compatibility), ou suivez la [configuration de WSL](#où-cursor-stocke-les-données) pour lire les données Store sous Windows ou dans WSL.

| Critère | **cursor-history (ce projet)** | [deja-vu](https://github.com/vshulcz/deja-vu) | [cursaves](https://github.com/Callum-Ward/cursaves) | [johnlindquist/cursor-history](https://github.com/johnlindquist/cursor-history) |
|---|---|---|---|---|
| Orientation | Consultation et gestion Cursor ; CLI + API Node.js | Mémoire inter-agents | Synchronisation Git / S3 | Navigation, export, presse-papiers |
| Sources Cursor documentées | Composer, transcriptions Agent, Store / CLI, ACP | SQLite IDE, transcriptions CLI¹ | SQLite des espaces de travail/global | Non documenté |
| Recherche | Texte des conversations entre espaces de travail | Index inter-agents | Non documenté | Titres, recherche approximative |
| Conservation et transfert | Sauvegarde/restauration Composer ; migration Composer admissible | Synchronisation/transfert de mémoire | Restauration ; copie entre espaces | Non documenté |

Comparaison documentaire vérifiée le **2026-09-09**, à partir des README liés et du ¹ [registre Cursor de deja-vu](https://github.com/vshulcz/deja-vu/blob/main/docs/registry/cursor.md) ; aucun test comparatif d'exécution. « Non documenté » signifie que les sources consultées ne décrivent pas cette capacité, sans conclure à son absence.

Nos sauvegardes et restaurations couvrent uniquement les bases Composer. La migration concerne les sessions Composer admissibles, à l'exclusion des sessions Store seules ou à sources fusionnées. Les sessions Store / ACP et transcriptions lisibles peuvent être exportées, mais ces exports ne sont pas des archives de sauvegarde restaurables. Voir le [contrat de compatibilité](./compatibility.md).

## Pourquoi cet outil existe

Vous vous souvenez peut-être d'avoir résolu un problème avec Cursor, sans savoir dans quel projet, quelle session ou quelle interface. Une recherche par mot-clé dans l'historique local peut vous aider à retrouver cette conversation.

Le [CLI Agent de Cursor](https://cursor.com/docs/cli/reference/parameters) propose `agent ls`, `agent resume` et `agent --resume=<id>` pour retrouver ou reprendre des sessions CLI. `cursor-history` ajoute des workflows de lecture et de gestion de l'historique local pris en charge, à travers les espaces de travail et les formats de stockage.

| Votre besoin | Par où commencer |
|---|---|
| Reprendre une conversation dans Cursor Agent CLI | `agent ls` ou `agent --resume=<id>` de Cursor |
| Rechercher une expression dans les conversations de plusieurs espaces de travail | `cursor-history search "connection pool"` |
| Lire ou exporter une session locale découverte | `cursor-history show 1` ou `cursor-history export 1` |
| Sauvegarder ou restaurer l'historique Composer | `cursor-history backup` / `cursor-history restore` ([utilisation](#sauvegarde-et-restauration)) |
| Déplacer des sessions Composer prises en charge vers un autre espace de travail | `cursor-history migrate-session` ([utilisation](#migrer-des-sessions)) |
| Exploiter l'historique dans votre application | [API Node.js](#api-bibliothèque) |

## Plusieurs générations de stockage et interfaces Cursor

Différentes versions et interfaces de Cursor peuvent laisser plusieurs représentations locales sur la même machine. `cursor-history` découvre les sources prises en charge suivantes :

| Source | Fichiers locaux | Utilisation |
|---|---|---|
| Ancien format / Composer | `workspaceStorage/*/state.vscdb` et `globalStorage/state.vscdb` dans le répertoire utilisateur de Cursor | Enregistrements des espaces de travail et conversations globales |
| Transcriptions Agent | `~/.cursor/projects/**/agent-transcripts/**/*.jsonl` | Texte des conversations et appels d'outils présents dans la transcription |
| Store / CLI | `~/.cursor/chats/**/store.db` | Données de conversation Store par session |
| Sessions ACP | `~/.cursor/acp-sessions/**/store.db` | Données Store découvertes sous la racine ACP |

Consultez [Où Cursor stocke les données](#où-cursor-stocke-les-données) pour les chemins par plateforme, les racines personnalisées et WSL.

Ces représentations ne contiennent pas toujours les mêmes champs. Une transcription peut omettre des horodatages ou des résultats d'outils. Lorsqu'une base Store exploitable et une transcription coexistent dans le périmètre autorisé, la base fournit la conversation Store et la transcription reste une trace de provenance. Les informations de source et de provenance temporelle distinguent les données stockées, les valeurs inférées et les vues partielles.

La prise en charge de la lecture n'implique pas celle de la sauvegarde ou de la migration : les archives actuelles contiennent uniquement Composer, et les sessions Store-only ou issues de sources fusionnées ne peuvent pas être migrées. Une couverture plus large figure dans la [feuille de route](#feuille-de-route). Le [contrat de compatibilité et d'intégrité](./compatibility.md) précise les limites.

## Trois façons d'utiliser votre historique

### Le retrouver

```bash
cursor-history list --all
cursor-history search "connection pool"
cursor-history show 1
```

Retrouvez la conversation où vous avez déjà résolu le problème, même dans un autre espace de travail. L'historique local pris en charge reste consultable même s'il précède l'installation de cet outil.

### Le conserver

```bash
cursor-history export 1
cursor-history backup
cursor-history migrate-session 1 /path/to/new/workspace --dry-run
```

Exportez les sessions lisibles en Markdown ou JSON. Sauvegardez l'historique Composer et prévisualisez la migration des sessions Composer prises en charge avant de les déplacer.

### Le réutiliser

Utilisez directement la [bibliothèque Node.js](#api-bibliothèque), ou connectez le serveur distinct [cursor-history-mcp](https://github.com/S2thend/cursor-history-mcp) pour permettre à un assistant compatible MCP de rechercher dans votre historique de développement.

Des mois de décisions, de corrections, de prompts et d'activité des outils peuvent déjà se trouver sur disque. Retrouvez plus facilement ce contexte la prochaine fois que vous en aurez besoin.

## Fonctionnalités

- **Double interface** - Utilisable en tant qu'outil CLI ou importable comme bibliothèque dans vos projets Node.js
- **Liste des sessions** - Voir toutes les sessions de chat à travers les espaces de travail
- **Consulter les conversations** - Examiner le contenu disponible dans chaque source, notamment :
  - Réponses IA avec explications en langage naturel
  - **Affichage complet des diff** pour les modifications de fichiers avec coloration syntaxique
  - **Appels d'outils détaillés** montrant tous les paramètres (chemins de fichiers, motifs de recherche, commandes, etc.)
  - Raisonnement et réflexion de l'IA
  - Horodatages avec provenance explicite (stockée ou inférée)
- **Recherche** - Rechercher par mot-clé dans le contenu des conversations de plusieurs espaces de travail, avec correspondances surlignées
- **Export** - Sauvegarder les sessions en fichiers Markdown ou JSON
- **Migration** - Déplacer ou copier les sessions Composer prises en charge entre espaces de travail (ex. lors du renommage de projets)
- **Sauvegarde et restauration** - Sauvegarder les bases Composer et les restaurer si nécessaire
- **Multi-plateforme et prise en charge de WSL** - Fonctionne sur macOS, Windows, Linux et WSL. Depuis WSL, configurez le chemin des données pour lire les sessions Cursor Store stockées sous Windows ou dans WSL. Consultez la [configuration de WSL](#où-cursor-stocke-les-données).

## Installation

### Depuis NPM (Recommandé)

```bash
# Installation globale
npm install -g cursor-history

# Utiliser le CLI
cursor-history list
```

### Depuis les sources

```bash
# Cloner et compiler
git clone https://github.com/S2thend/cursor_chat_history.git
cd cursor_chat_history
npm install
npm run build

# Exécuter directement
node dist/cli/index.js list

# Ou lier globalement
npm link
cursor-history list
```

## Prérequis

- Node.js 20.x ou 22.x–26.x (Node 21 n'est pas pris en charge ; Node.js 22.5+ est recommandé pour SQLite intégré)
- Historique local existant de Cursor IDE ou Agent CLI dans un format pris en charge

## Configuration du pilote SQLite

cursor-history supporte deux pilotes SQLite pour une compatibilité maximale :

| Pilote | Description | Limite de capacité Node.js |
|--------|-------------|----------------------------|
| `node:sqlite` | Module intégré ; sélectionné uniquement s'il fournit toutes les API requises | Lecture dès 22.5 ; sauvegarde en ligne dès 22.16.0 et 23.8.0 |
| `better-sqlite3` | Binding natif et repli automatique lorsqu'il est capable | Versions majeures 20 et 22–26 |

### Sélection automatique du pilote

cursor-history sélectionne par opération et vérifie les capacités réelles, pas seulement si le
module s'importe :

1. il préfère **node:sqlite** lorsque toutes les API requises sont disponibles ;
2. sinon il utilise un **better-sqlite3** installé et capable.

Un pilote forcé ne se replie jamais sur l'autre : une capacité manquante produit une erreur typée
et une solution exploitable.

### Sélection manuelle du pilote

Vous pouvez forcer un pilote spécifique en utilisant la variable d'environnement :

```bash
# Forcer better-sqlite3
CURSOR_HISTORY_SQLITE_DRIVER=better-sqlite3 cursor-history list

# Forcer node:sqlite (doit fournir toutes les API requises par l'opération)
CURSOR_HISTORY_SQLITE_DRIVER=node:sqlite cursor-history list
```

### Déboguer la sélection du pilote

Pour voir quel pilote est utilisé :

```bash
DEBUG=cursor-history:* cursor-history list
```

### Contrôle du pilote via l'API bibliothèque

Lors de l'utilisation de cursor-history comme bibliothèque, vous pouvez contrôler le pilote par programmation :

```typescript
import { setDriver, getActiveDriver, listSessions } from 'cursor-history';

// Forcer un pilote spécifique avant toute opération
setDriver('better-sqlite3');

// Vérifier quel pilote est actif
const driver = getActiveDriver();
console.log(`Pilote utilisé : ${driver}`);

// Ou configurer via LibraryConfig
const result = await listSessions({
  sqliteDriver: 'node:sqlite'  // Forcer node:sqlite pour cet appel
});
```

## Utilisation

### Lister les sessions

```bash
# Lister les sessions récentes (par défaut : 20)
cursor-history list

# Lister toutes les sessions
cursor-history list --all

# Lister avec les IDs composer (pour les outils externes)
cursor-history list --ids

# Limiter les résultats
cursor-history list -n 10

# Lister uniquement les espaces de travail
cursor-history list --workspaces
```

### Voir une session

```bash
# Afficher une session par numéro d'index
cursor-history show 1

# Afficher avec messages tronqués (aperçu rapide)
cursor-history show 1 --short

# Afficher le texte complet de réflexion/raisonnement de l'IA
cursor-history show 1 --think

# Afficher les détails complets des appels d'outils (commandes, contenu, résultats)
cursor-history show 1 --tool

# Afficher les messages d'erreur complets (non tronqués à 300 caractères)
cursor-history show 1 --error

# Filtrer par type de message (user, assistant, tool, thinking, error)
cursor-history show 1 --only user
cursor-history show 1 --only user,assistant
cursor-history show 1 --only tool,error

# Combiner les options
cursor-history show 1 --short --think --tool --error
cursor-history show 1 --only user,assistant --short

# Sortie en JSON
cursor-history show 1 --json
```

### Rechercher

```bash
# Rechercher un mot-clé
cursor-history search "react hooks"

# Limiter les résultats
cursor-history search "api" -n 5

# Ajuster le contexte autour des correspondances
cursor-history search "error" --context 100
```

### Exporter

```bash
# Exporter une seule session en Markdown
cursor-history export 1

# Exporter vers un fichier spécifique
cursor-history export 1 -o ./mon-chat.md

# Exporter en JSON
cursor-history export 1 --format json

# Exporter toutes les sessions vers un répertoire
cursor-history export --all -o ./exports/

# Écraser les fichiers existants
cursor-history export 1 --force
```

### Migrer des sessions

La migration prend en charge les sessions Composer éligibles. Les sessions Store-only, issues de sources fusionnées ou ambiguës sont refusées ; utilisez `--dry-run` pour prévisualiser une migration.

```bash
# Déplacer une seule session vers un autre espace de travail
cursor-history migrate-session 1 /chemin/vers/nouveau/projet

# Déplacer plusieurs sessions (indices ou IDs séparés par des virgules)
cursor-history migrate-session 1,3,5 /chemin/vers/projet

# Copier au lieu de déplacer (garde l'original)
cursor-history migrate-session --copy 1 /chemin/vers/projet

# Prévisualiser ce qui se passerait sans effectuer de changements
cursor-history migrate-session --dry-run 1 /chemin/vers/projet

# Déplacer toutes les sessions d'un espace de travail vers un autre
cursor-history migrate /ancien/projet /nouveau/projet

# Copier toutes les sessions (sauvegarde)
cursor-history migrate --copy /projet /sauvegarde/projet

# Forcer la fusion avec les sessions existantes à la destination
cursor-history migrate --force /ancien/projet /projet/existant
```

### Sauvegarde et restauration

Les archives contiennent les données Composer `state.vscdb`. Elles n'incluent ni les bases Store, ni les transcriptions Agent, ni les magasins de sessions ACP. Exportez les sessions lisibles de ces sources en Markdown ou JSON pour obtenir une copie portable ; ces exports ne sont pas des archives de sauvegarde restaurables.

```bash
# Créer une sauvegarde de l'historique Composer
cursor-history backup

# Créer une sauvegarde vers un fichier spécifique
cursor-history backup -o ~/ma-sauvegarde.zip

# Écraser une sauvegarde existante
cursor-history backup --force

# Lister les sauvegardes disponibles
cursor-history list-backups

# Lister les sauvegardes dans un répertoire spécifique
cursor-history list-backups -d /chemin/vers/sauvegardes

# Restaurer depuis une sauvegarde
cursor-history restore ~/cursor-history-backups/backup.zip

# Restaurer vers un emplacement personnalisé
cursor-history restore backup.zip --target /cursor/data/personnalisé

# Forcer l'écrasement des données existantes
cursor-history restore backup.zip --force

# Voir les sessions d'une sauvegarde sans restaurer
cursor-history list --backup ~/backup.zip
cursor-history show 1 --backup ~/backup.zip
cursor-history search "requête" --backup ~/backup.zip
cursor-history export 1 --backup ~/backup.zip
```

### Options globales

```bash
# Sortie en JSON (fonctionne avec toutes les commandes)
cursor-history --json list

# Utiliser un chemin de données Cursor personnalisé
cursor-history --data-path ~/.cursor-alt list

# Filtrer par espace de travail
cursor-history --workspace /chemin/vers/projet list
```

## Ce que vous pouvez voir

En parcourant votre historique de chat, vous verrez :

- **Conversations complètes** - Tous les messages échangés avec Cursor AI
- **Chaque message rendu** - Chaque message résolu est affiché une fois dans l'ordre ; les doublons consécutifs ne sont pas pliés, afin que les appels d'outils, la provenance et les données de tokens distincts ne soient jamais masqués
- **Horodatages** - L'heure directement stockée d'un message quand elle est disponible (format HH:MM:SS) ; les messages sans heure directement stockée n'affichent pas d'horodatage plutôt qu'un repli fabriqué
- **Sessions résolues entre piles** - Quand le même UUID existe dans Composer et Store, cursor-history conserve les identités Composer compatibles et produit une vue à provenance explicite. La portée de l'espace de travail est appliquée avant la lecture du contenu : une source connue hors frontière n'est pas ouverte et rend la vue partielle ; les sources permises suivent la politique canonique de backbone et d'enrichissement, pas une fusion aveugle champ par champ.
- **Actions des outils IA** - Vue détaillée de ce que Cursor AI a fait :
  - **Modifications/écritures de fichiers** - Affichage complet des diff avec coloration syntaxique montrant exactement ce qui a changé
  - **Lectures de fichiers** - Chemins de fichiers et aperçus du contenu (utilisez `--tool` pour le contenu complet)
  - **Opérations de recherche** - Motifs, chemins et requêtes de recherche utilisés
  - **Commandes terminal** - Texte complet des commandes
  - **Listages de répertoires** - Chemins explorés
  - **Erreurs d'outils** - Opérations échouées/annulées affichées avec l'indicateur de statut ❌ et les paramètres
  - **Décisions utilisateur** - Indique si vous avez accepté (✓), rejeté (✗), ou en attente (⏳) les opérations d'outils
  - **Erreurs** - Messages d'erreur avec mise en évidence emoji ❌ (extraits de `toolFormerData.additionalData.status`)
- **Raisonnement IA** - Voir le processus de réflexion de l'IA derrière les décisions (utilisez `--think` pour le texte complet)
- **Artefacts de code** - Diagrammes Mermaid, blocs de code, avec coloration syntaxique
- **Explications en langage naturel** - Explications IA combinées avec le code pour un contexte complet

### Options d'affichage

- **Vue par défaut** - Messages complets avec réflexion tronquée (200 car.), lectures de fichiers (100 car.) et erreurs (300 car.)
- **Mode `--short`** - Tronque les messages utilisateur et assistant à 300 caractères pour un scan rapide
- **Drapeau `--think`** - Affiche le texte complet de raisonnement/réflexion IA (non tronqué)
- **Drapeau `--tool`** - Affiche les détails complets des appels d'outils : commandes, contenu et résultats
- **Drapeau `--error`** - Affiche les messages d'erreur complets au lieu de l'aperçu de 300 caractères
- **Drapeau `--only <types>`** - Filtre les messages par type : `user`, `assistant`, `tool`, `thinking`, `error` (séparés par des virgules)

## Où Cursor stocke les données

| Plateforme | Stockage Composer | Stockage Store |
|---|---|---|
| macOS | `~/Library/Application Support/Cursor/User/` | `~/.cursor/` |
| Windows | `%APPDATA%/Cursor/User/` | `%USERPROFILE%\.cursor\` |
| Linux / WSL | `~/.config/Cursor/User/` | `~/.cursor/` |

L'outil découvre et lit automatiquement les deux stockages. Le fichier `store.db` de chaque session est la source principale des messages Store. Une fois les capacités du pilote et l'infrastructure de lecture par instantané disponibles, la transcription peut servir de repli si la base manque, ne contient aucun message exploitable ou présente une corruption ou une erreur de lecture des données sources. Les erreurs de capacité du pilote ou d'infrastructure d'instantané sont fatales et ne déclenchent pas ce repli. Une base exploitable reste la seule source principale Store ; une transcription coexistante est conservée uniquement comme provenance supplantée.

Utilisez `--data-path <path>` ou `CURSOR_DATA_PATH` pour un répertoire Cursor personnalisé, et `CURSOR_STORE_ROOT` pour configurer séparément la racine Store. Cette racine ou ses sous-répertoires `chats`, `projects` ou `acp-sessions` sont acceptés et normalisés vers la même racine.

Sous WSL, les données Store côté Windows sont généralement montées dans `/mnt/c/Users/<windows-user>/.cursor`. Exemple : `CURSOR_STORE_ROOT=/mnt/c/Users/<windows-user>/.cursor cursor-history list --all`. Utilisez plutôt `~/.cursor` côté WSL pour les sessions créées par un agent Cursor exécuté dans cette distribution.

Ne réutilisez pas des `node_modules` natifs installés sous Windows pour exécuter le CLI sous WSL. Installez les dépendances avec Node.js pour Linux dans un répertoire de dépendances WSL distinct avant les tests ou compilations côté Linux. `cursor-history` n'installe et ne supprime jamais de dépendances automatiquement.

## API Bibliothèque

En plus du CLI, vous pouvez utiliser cursor-history comme bibliothèque dans vos projets Node.js :

```typescript
import {
  listSessions,
  getSession,
  searchSessions,
  exportSessionToMarkdown
} from 'cursor-history';

// Lister toutes les sessions avec pagination
const result = await listSessions({ limit: 10 });
console.log(`Trouvé ${result.pagination.total} sessions`);

for (const session of result.data) {
  console.log(`${session.id}: ${session.messageCount} messages`);
}

// Obtenir une session spécifique (index à base zéro)
const session = await getSession(0);
console.log(session.messages);

// Rechercher dans toutes les sessions
const results = await searchSessions('authentication', { context: 2 });
for (const match of results) {
  // Index dans le tableau complet, décalage UTF-16 et ligne source complète.
  console.log(match.messageIndex, match.offset, match.match);
}

// Exporter en Markdown
const markdown = await exportSessionToMarkdown(0);
```

### API de migration

```typescript
import { migrateSession, migrateWorkspace } from 'cursor-history';

// Déplacer une session vers un autre espace de travail
const moveResults = await migrateSession({
  sessions: 3,  // index ou ID
  destination: '/chemin/vers/nouveau/projet'
});
console.log(moveResults);

// Copier plusieurs sessions (garde les originaux)
const copyResults = await migrateSession({
  sessions: [1, 3, 5],
  destination: '/chemin/vers/projet',
  mode: 'copy'
});
console.log(copyResults);

// Migrer toutes les sessions entre espaces de travail
const workspaceResult = await migrateWorkspace({
  source: '/ancien/projet',
  destination: '/nouveau/projet'
});
console.log(`Migré ${workspaceResult.successCount} sessions`);
```

### API de sauvegarde

```typescript
import {
  createBackup,
  restoreBackup,
  validateBackup,
  listBackups,
  getDefaultBackupDir,
  listSessions
} from 'cursor-history';

// Créer une sauvegarde
const result = await createBackup({
  outputPath: '~/ma-sauvegarde.zip',
  force: true,
  onProgress: (progress) => {
    console.log(`${progress.phase}: ${progress.filesCompleted}/${progress.totalFiles}`);
  }
});
console.log(`Sauvegarde créée : ${result.backupPath}`);
console.log(`Sessions : ${result.manifest.stats.sessionCount}`);

// Valider une sauvegarde
const validation = await validateBackup('~/backup.zip');
if (validation.status === 'valid') {
  console.log('La sauvegarde est valide');
} else if (validation.status === 'warnings') {
  console.log('La sauvegarde a des avertissements :', validation.corruptedFiles);
}

// Restaurer depuis une sauvegarde
const restoreResult = await restoreBackup({
  backupPath: '~/backup.zip',
  force: true
});
console.log(`Restauré ${restoreResult.filesRestored} fichiers`);
// Consultez restoreResult.warnings : les entrées corrompues sont ignorées, jamais restaurées.

// Lister les sauvegardes disponibles
const backups = await listBackups();  // Scanne ~/cursor-history-backups/
for (const backup of backups) {
  console.log(`${backup.filename}: ${backup.manifest?.stats.sessionCount} sessions`);
}

// Lire les sessions depuis une sauvegarde sans restaurer
const sessions = await listSessions({ backupPath: '~/backup.zip' });
```

### Fonctions disponibles

| Fonction | Description |
|----------|-------------|
| `listSessions(config?)` | Lister les sessions avec pagination |
| `getSession(index, config?)` | Obtenir une session complète par index |
| `searchSessions(query, config?)` | Rechercher dans les sessions |
| `exportSessionToJson(index, config?)` | Exporter une session en JSON |
| `exportSessionToMarkdown(index, config?)` | Exporter une session en Markdown |
| `exportAllSessionsToJson(config?)` | Exporter toutes les sessions en JSON |
| `exportAllSessionsToMarkdown(config?)` | Exporter toutes les sessions en Markdown |
| `migrateSession(config)` | Déplacer/copier des sessions vers un autre espace de travail |
| `migrateWorkspace(config)` | Déplacer/copier toutes les sessions entre espaces de travail |
| `createBackup(config?)` | Sauvegarder l'historique Composer |
| `restoreBackup(config)` | Restaurer l'historique depuis une sauvegarde |
| `validateBackup(path)` | Valider l'intégrité d'une sauvegarde |
| `listBackups(directory?)` | Lister les fichiers de sauvegarde disponibles |
| `getDefaultBackupDir()` | Obtenir le chemin du répertoire de sauvegarde par défaut |
| `getDefaultDataPath()` | Obtenir le chemin des données Cursor spécifique à la plateforme |
| `setDriver(name)` | Définir le pilote SQLite ('better-sqlite3' ou 'node:sqlite') |
| `getActiveDriver()` | Obtenir le nom du pilote SQLite actuellement actif |

### Options de configuration

```typescript
import type { MessageType } from 'cursor-history';

interface LibraryConfig {
  dataPath?: string;       // Chemin personnalisé des données Cursor
  workspace?: string;      // Filtrer par chemin d'espace de travail
  limit?: number;          // Limite de pagination
  offset?: number;         // Décalage de pagination
  context?: number;        // Lignes de contexte de recherche
  backupPath?: string;     // Lire depuis un fichier de sauvegarde au lieu des données en direct
  sqliteDriver?: 'better-sqlite3' | 'node:sqlite';  // Forcer un pilote SQLite spécifique
  messageFilter?: MessageType[];  // Filtrer les messages par type (user, assistant, tool, thinking, error)
}
```

### Gestion des erreurs

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
  isBackupPublishedPermissionError,
  isRestoreRollbackError,
  isRestoreError,
  isInvalidBackupError,
  validateMessageTypes
} from 'cursor-history';

try {
  const result = await listSessions();
} catch (err) {
  if (isDatabaseLockedError(err)) {
    console.error('Base de données verrouillée - fermez Cursor et réessayez');
  } else if (isDatabaseNotFoundError(err)) {
    console.error('Données Cursor non trouvées');
  } else if (isSessionNotFoundError(err)) {
    console.error('Session non trouvée');
  } else if (isWorkspaceNotFoundError(err)) {
    console.error('Espace de travail non trouvé - ouvrez d\'abord le projet dans Cursor');
  }
}

try {
  await createBackup({ outputPath: '/private/backups/cursor.zip' });
} catch (err) {
  if (isBackupPublishedPermissionError(err)) {
    if (err.details.pathIdentityVerified) {
      console.error('La sauvegarde publiée vérifiée nécessite une correction de mode :', err.details.outputPath);
    } else {
      // Le point de validation est franchi, mais ce chemin n'est pas fiable. Ne changez pas son mode ici.
      console.error("Le chemin de sauvegarde publié nécessite une récupération d'identité :", err.details.outputPath);
    }
  }
}

// Valider les valeurs de filtre non typées avant de les passer à une opération de lecture
const invalidTypes = validateMessageTypes(['invalid']);
if (invalidTypes.length > 0) {
  console.error('Types de filtre invalides :', invalidTypes);
}

// Erreurs spécifiques aux sauvegardes
try {
  await createBackup();
} catch (err) {
  if (isBackupError(err)) {
    console.error('Échec de la sauvegarde :', err.message);
  } else if (isInvalidBackupError(err)) {
    console.error('Fichier de sauvegarde invalide');
  } else if (isRestoreError(err)) {
    console.error('Échec de la restauration :', err.message);
  }
}

try {
  await restoreBackup({ backupPath: '/private/backups/cursor.zip', force: true });
} catch (err) {
  if (isRestoreRollbackError(err)) {
    // Ce sont des chemins relatifs au manifeste, jamais des localisateurs physiques privés.
    console.error('Récupération manuelle requise pour :', err.details.residualFiles);
  }
}
```

## Compatibilité et mises à niveau

> **Contrat de compatibilité :** le document canonique en anglais
> [Compatibility and Data-Integrity Contract](./compatibility.md) définit l'identité stable, la
> portée et la base des indices, la frontière d'E/S par espace de travail, la fidélité/provenance,
> les horodatages inférés, les limites de lecture, les permissions de sauvegarde et les exemples
> CLI/bibliothèque vérifiés. En cas de divergence, ce contrat fait autorité.
>
> Les consommateurs incrémentaux de la bibliothèque doivent épingler v0.16 jusqu'à validation de
> v0.18.0 avant une mise à niveau depuis v0.17. Le chemin sans modification du
> consommateur est garanti pour les archives v0.16 Composer uniquement ; il ne promet pas de
> conserver les ID Store synthétiques instables de v0.17.
>
> v0.18.0 corrige directement les coordonnées publiques de recherche de v0.16/v0.17 et ajoute
> l'index zéro-basé aux exports JSON comme nouvelle métadonnée. Le contrat canonique définit aussi
> le point de publication et les erreurs typées de permissions ou de nettoyage après publication ;
> un chemin résiduel non vérifié ne doit jamais être supprimé à l'aveugle. La
> restauration ignore les entrées corrompues et refuse les chemins, destinations dupliquées ou
> liens non sûrs avant toute écriture ; `--force` ne désactive pas ces contrôles. Après toute
> publication, un échec ne tente aucun retour arrière automatique : il préserve chaque fichier et
> renvoie `RESTORE_ROLLBACK_INCOMPLETE` ; arrêtez Cursor et restaurez une sauvegarde fiable.

### Compatibilité de la v0.18

- Tous les ID de session, y compris les UUID canoniques, conservent le comportement de la v0.16 :
  recherche, regroupement et association sont sensibles à la casse et comparent les octets
  exactement. Réutilisez l'orthographe renvoyée ; une variante de casse est un ID distinct.
- Une migration avec `--workspace` ne lit hors périmètre que les métadonnées nécessaires, lie les
  clés physiques exactes et prépare tout le lot avant la première écriture. Une cible ambiguë ou
  inéligible annule le lot sans modification.
- Lors de la fusion Composer/Store, les tours Store actifs placés au début, au milieu ou à la fin
  apparaissent une seule fois ; les branches latérales restent exclues et les anciens ID Composer ne
  changent pas.
- À date égale, les lignes Composer conservent l'ordre de découverte de la v0.16 fondé sur
  `String.localeCompare()` dans le même environnement pris en charge.
- Le manifeste de sauvegarde conserve `manifest.version: "1.0.0"` ; l'inventaire facultatif utilise
  son propre `schemaVersion: 1`. Consultez [compatibility.md](compatibility.md) pour le contrat
  normatif.

## Feuille de route

Étendre la sauvegarde, la restauration et la migration aux nouvelles sources déjà prises en charge en lecture. Il s'agit d'orientations prévues, sans engagement sur une version ou une date de publication.

- [ ] **Sauvegarde et restauration Store / ACP** : inclure les bases Store, les transcriptions Agent et les métadonnées associées dans des archives restaurables, en préservant les relations entre sources et en validant le cycle sauvegarde-restauration.
- [ ] **Migration des sessions Store-only** : déplacer ou copier ces sessions entre espaces de travail, mettre à jour les associations et références de chemins, puis vérifier leur découverte par Cursor.
- [ ] **Migration des sessions issues de sources fusionnées** : déplacer ou copier les sessions présentes à la fois dans Composer et Store, en préservant leurs identifiants natifs et la cohérence des sources.

En attendant, la [sauvegarde et la restauration](#sauvegarde-et-restauration) couvrent Composer, et la [migration](#migrer-des-sessions) prend en charge les sessions Composer éligibles. Les sessions Store / ACP lisibles peuvent être exportées en Markdown ou JSON, mais ces exports ne sont pas des archives de sauvegarde restaurables.

Partagez vos cas d'usage et des exemples de stockage reproductibles via [GitHub Issues](https://github.com/S2thend/cursor-history/issues) pour aider à prioriser ces travaux.

## Développement

### Compiler depuis les sources

```bash
npm install
npm run build
```

### Exécuter les tests

```bash
npm test              # Exécuter tous les tests
npm run test:watch    # Mode surveillance
```

### Publier sur npm

Les versions utilisent la publication de confiance npm via GitHub Actions. Aucun secret de dépôt
`NPM_TOKEN` n'est utilisé. Avant la première publication :

1. Configurez l'éditeur de confiance du paquet npm pour ce dépôt exact, saisissez
   `npm-publish.yml` comme nom du workflow (le fichier se trouve dans
   `.github/workflows/npm-publish.yml`), définissez l'environnement `npm-release-verification` et
   autorisez l'action `npm publish`.
2. Créez l'environnement GitHub `npm-release-verification`, imposez des mainteneurs désignés comme
   réviseurs et empêchez tout contournement non révisé conformément à la politique du dépôt.

Pour chaque version :

1. Mettez à jour et validez toutes les métadonnées versionnées et les notes de version, terminez les
   contrôles documentés, puis figez une révision propre.
2. Vérifiez que le tag n'existe pas, puis poussez uniquement ce tag (par exemple,
   `git push origin v0.18.0`). Ne poussez ni ne déplacez un tag avant que la révision soit figée.
3. Le workflow valide les sources et tous les environnements pris en charge, empaquette une seule
   fois et lie le candidat à sa révision et à son SHA-256. Une fois ces contrôles réussis, la vraie
   tâche `publish` s'arrête sur l'environnement protégé `npm-release-verification` avant de demander
   son jeton OIDC.
4. Téléchargez ce candidat adressé par sa somme et effectuez les contrôles privés de l'artefact exact
   décrits dans [release-verification.md](release-verification.md). N'approuvez l'environnement
   qu'après leur réussite.
5. L'approbation publie exactement ces octets conservés avec la provenance npm, sans nouvelle
   compilation ni nouvel empaquetage.

Tout échec de source, de runtime, d'artefact ou de vérification privée bloque la publication. Ne
forcez jamais silencieusement le déplacement d'un tag : corrigez explicitement un candidat non
publié et utilisez une nouvelle version si des octets ont déjà été publiés.

## Contribuer

Nous accueillons les contributions de la communauté ! Voici comment vous pouvez aider :

### Signaler des problèmes

- **Rapports de bugs** : [Ouvrez une issue](https://github.com/S2thend/cursor_chat_history/issues/new) avec les étapes pour reproduire, le comportement attendu vs réel, et votre environnement (OS, version Node.js)
- **Demandes de fonctionnalités** : [Ouvrez une issue](https://github.com/S2thend/cursor_chat_history/issues/new) décrivant la fonctionnalité et son cas d'utilisation

### Soumettre des Pull Requests

1. Forkez le dépôt
2. Créez une branche de fonctionnalité (`git checkout -b feature/ma-fonctionnalite`)
3. Faites vos modifications
4. Exécutez les tests et le linting (`npm test && npm run lint`)
5. Committez vos modifications (`git commit -m 'Ajoute ma fonctionnalité'`)
6. Poussez vers votre fork (`git push origin feature/ma-fonctionnalite`)
7. [Ouvrez une Pull Request](https://github.com/S2thend/cursor_chat_history/pulls)

### Configuration de l'environnement de développement

```bash
git clone https://github.com/S2thend/cursor_chat_history.git
cd cursor_chat_history
npm install
npm run build
npm test
```

## Licence

MIT
