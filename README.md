# RTA in Japan NodeCG レイアウト Stream Deck plugin

- NodeCGの `rtainjapan-layouts` を Stream Deck から操作するためのプラグイン

## しくみ

- NodeCGの `socket.io` に接続し、`message` で `startTimer` / `stopTimer`などのメッセージを送信
- NodeCGのレプリカントを購読し、走者名・状態をキーに表示
- NodeCG再起動などに備えて「再同期」アクションを用意
- すべてのアクションは1つの共有NodeCGクライアント（`src/nodecg/client.ts`）を介して動作

## アクション

| アクション                   | 動作                                     | 設定               |
| ---------------------------- | ---------------------------------------- | ------------------ |
| タイマー開始                 | `startTimer`                             | —                  |
| タイマー停止                 | `stopTimer`                              | —                  |
| ファンアート 表示開始        | `startTweetQueue`                        | —                  |
| ファンアート 表示停止        | `stopTweetQueue`                         | —                  |
| 走者 完走                    | `completeRunner {index, forfeit: false}` | 走者 index         |
| 走者 リタイア                | `completeRunner {index, forfeit: true}`  | 走者 index         |
| 走者 再開                    | `resumeRunner(index)`                    | 走者 index         |
| 走者名表示                   | 表示のみ（押下で操作なし）               | 走者 index         |
| 再同期                       | レプリカント再取得                       | —                  |
| メッセージ送信（上級者向け） | 任意の `messageName` + JSON content      | メッセージ名・内容 |

### 接続設定

プラグイン全体（Global Settings）で共有。

- NodeCG URL(既定値: `http://localhost:9090`)
- Bundle 名 (既定値: `rtainjapan-layouts`)
- 認証キー (既定値: なし)
  - nodecgのconfigで、loginを有効にする場合に使用

## NodeCG 連携の契約

- プラグインが利用するレプリカントは本体側と合わせる必要があるため、本体側が変わった場合は、`src/nodecg/replicants.ts` のローカル型と各アクションを対応させること。

- 送信メッセージ: `startTimer` / `stopTimer` /
  `completeRunner { index, forfeit }` / `resumeRunner(index)` /
  `startTweetQueue` / `stopTweetQueue`
- 購読レプリカント:
  - `timer`（`raw` / `formatted` / `timerState` / `results[]`）
  - `current-run`（`runners[].name`）
  - `tweet-queue-playing`（boolean）
- バンドル名: `rtainjapan-layouts`（Global Settings で変更可）

## 開発

```sh
npm install
npm run test:types   # 型チェック
npm run build        # jp.rtainjapan.layouts.sdPlugin/bin/plugin.cjs を生成
npm run watch        # 変更監視ビルド
```

Stream Deck で読み込むには、[Stream Deck CLI](https://docs.elgato.com/streamdeck/cli/intro) で `jp.rtainjapan.layouts.sdPlugin` をリンクするか、`*.streamDeckPlugin` としてパッケージすること。

```sh
npx @elgato/cli link jp.rtainjapan.layouts.sdPlugin
npx @elgato/cli restart jp.rtainjapan.layouts
```

## 補足

- プロパティインスペクターは `sdpi-components`（CDN）を利用しているため、<br>完全なオフライン運用が必要な場合はライブラリを同梱（vendoring）すること。
