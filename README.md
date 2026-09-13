# otameshi-game-sample

Game Plaza プラットフォームに載せる外部ゲームのテンプレート。  
このリポジトリをフォークして、ゲームを実装する。

## 構成

```
index.html   ... ゲーム本体（HTML + CSS + JS の単一ファイル）
meta.json    ... ゲームのメタ情報（プラットフォームが読み取る）
```

---

## AI実装ガイド

> このセクションは AI エージェント向けの仕様書です。  
> ゲーム実装を依頼された場合、この仕様に従って実装してください。

### 実装手順

1. `index.html` 内の「ゲームロジック」セクション（コメントで囲まれた部分）を書き換えてゲームを実装する
2. `meta.json` のタイトル・説明文を実装したゲームに合わせて更新する

### 触らないもの

- `GamePlazaSDK` オブジェクトの実装部分は変更しない（そのまま利用する）
- ビルドツール・バンドラーは導入しない（単一 HTML で完結させる）

### プラットフォーム連携仕様

`GamePlazaSDK` がプラットフォーム（親ウィンドウ）との postMessage 通信を担う。  
ゲームロジックからは以下の API を呼ぶだけでよい。

#### ライフサイクル

1. ゲームが iframe として読み込まれる
2. プラットフォームから `game_start` メッセージが届く → `GamePlazaSDK.init` のコールバックが発火
3. ゲーム終了時に `sendResult()` または `sendClose()` を呼ぶ
4. プラットフォーム側は受け取った結果をもとにトースト通知を表示し、iframe を閉じる

#### API

**`GamePlazaSDK.init(callback)`** — ゲーム開始コールバックを登録

```js
GamePlazaSDK.init(({ gameType, players, localPlayer }) => {
  // gameType: string - ゲーム識別子
  // players: Array<{ userName: string, uuid: string, isLocal: boolean }>
  // localPlayer: { userName, uuid, isLocal: true } - 操作中プレイヤー
});
```

**`GamePlazaSDK.sendResult(winnerId, scores?)`** — ゲーム結果を送信（ゲーム終了）

```js
GamePlazaSDK.sendResult(localPlayer.uuid, { score: 100 }); // 勝敗あり＋スコア送信
GamePlazaSDK.sendResult(null); // 引き分け（スコアなし）
```

- `winnerId`: `string | null` — 勝者の UUID。引き分けは `null`
- `scores`: `Record<string, number>` — 任意のスコア情報（省略可）

プラットフォーム側の挙動：
- `winnerId` をもとに勝敗のトースト通知を表示する
- `scores` は省略しても問題ない（送ればプラットフォーム側で将来的に利用される）
- スコアを競うゲームの場合は `scores` に含めて送ると良い

**`GamePlazaSDK.sendClose()`** — 結果なしでゲームを閉じる（キャンセル・中断時）

```js
GamePlazaSDK.sendClose();
```

プラットフォーム側は結果通知を出さずに iframe を閉じる。ゲームを途中でやめる場合に使う。

#### プロパティ（init 後に参照可能）

| プロパティ | 型 | 説明 |
|---|---|---|
| `GamePlazaSDK.players` | `Array` | 全プレイヤー |
| `GamePlazaSDK.localPlayer` | `Object \| null` | 操作中のプレイヤー |
| `GamePlazaSDK.gameType` | `string \| null` | ゲーム識別子 |

### meta.json 仕様

```json
{
  "title": "ゲームタイトル",
  "description": "ゲームの説明文（改行は \\n）"
}
```

### 制約

- 外部 CDN の利用は可（Canvas ライブラリ等）
- フレームワーク不要（Vanilla JS 推奨、ただし必要なら単一ファイル内で完結する範囲で使用可）
- レスポンシブ対応すること（iframe サイズが可変のため）

---

## 開発者向けガイド

> このセクションは人間の開発者向けです。  
> ゲームの内容を AI に依頼する場合、ここは読まなくて大丈夫です。

### ゲームの作り方

1. このリポジトリをフォーク（またはコピー）する
2. AI に作りたいゲームを伝える（README の「AI実装ガイド」を AI が読んで実装してくれる）
3. ローカルで動作確認
4. デプロイ

AI へのプロンプト例：

```
このリポジトリのREADMEを読んで、じゃんけんゲームを実装して
```

```
README の仕様に従って、15パズルを作って。タイマー付きで。
```

```
このリポジトリのREADMEを読んで、スネークゲームを実装して。制限時間60秒でスコアを競う形式で。
```

ゲームロジック以外（プラットフォーム連携・メッセージ通信）は AI が仕様書を見て実装するので、開発者はゲームの内容だけ考えれば OK。

### ローカル確認

`index.html` をブラウザで直接開くだけで動作確認できる（`file://` でOK）。

外部リソースを追加した場合など、静的サーバーが必要になったら：

```bash
npx serve .
```

プラットフォームからの `game_start` メッセージなしでも UI は表示される。

### デプロイ

GitHub Pages など任意の静的ホスティングにデプロイし、プラットフォーム側の `game-config.ts` に URL を登録する。

詳細手順: [docs/deploy-github-pages.md](docs/deploy-github-pages.md)
