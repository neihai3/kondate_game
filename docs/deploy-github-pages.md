# GitHub Pages デプロイ手順

## 前提

- GitHub にリポジトリが作成済み
- `index.html` と `meta.json` がルートに配置されている

---

## 手順

### 1. リポジトリに push

```bash
git add .
git commit -m "ゲーム実装"
git push origin main
```

### 2. GitHub Pages を有効化

1. GitHub のリポジトリページを開く
2. **Settings** → **Pages** に移動
3. **Source** で `Deploy from a branch` を選択
4. **Branch** で `main` / `/ (root)` を選択
5. **Save** をクリック

### 3. 公開を確認

数分待つと以下の URL でアクセスできるようになる：

```
https://<ユーザー名>.github.io/<リポジトリ名>/
```

### 4. プラットフォームに登録

公開 URL を game-plaza-proto 側の `game-config.ts` に追加する。

```ts
{
  name: 'your-game',
  url: 'https://<ユーザー名>.github.io/<リポジトリ名>/',
  origin: 'https://<ユーザー名>.github.io',
  metaUrl: 'https://<ユーザー名>.github.io/<リポジトリ名>/meta.json',
}
```

---

## トラブルシューティング

- **404 になる** → Pages の設定でブランチとフォルダが正しいか確認。反映まで数分かかることがある。
- **ゲームが動かない** → ブラウザの開発者ツール（Console）でエラーを確認。`file://` で動いたのに Pages で動かない場合は、Mixed Content（HTTP/HTTPS混在）を疑う。
