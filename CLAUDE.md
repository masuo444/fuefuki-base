# MaSU SALON 運用ガイド

## 構成
- サイト: `/Users/masuo/Desktop/MaSU SALON/index.html`
- GitHub: `masuo444/fuefuki-base` → Vercel自動デプロイ (`fuefuki-base.vercel.app`)
- LINE Harness管理画面: `https://6b655d53.masu-salon-admin-b406a08d.pages.dev`
- LINE Harness API: `https://masu-salon.keisukendo414.workers.dev`
- LINE Bot: `@412jqsnb`

## APIキー（使用時はコードに直接書かない）
- LINE Harness API Key: `b406a08d4f5ab121e7e02bfb57e1d69e7cb4e7e5f706f2bf8d5695edbbfb926b`
- LINE Account ID: `41eb2d20-bebb-4ad5-b153-d1f0e2f2c3ef`
- Stripe Secret Key: Cloudflare Worker secretに設定済み

## タグ一覧
- `7b6d0092` = 参加者
- `93b9d6cf` = リピーター
- `b5c3a85d` = 日本酒好き
- `e70e47b0` = ワイン好き
- `54040d55` = 読書好き
- `53e9e726` = 国際交流好き
- `0b032a28` = VIP

---

## イベント作成コマンド

「次のイベントを作って」と言われたら以下を実行：

### 必要情報（ユーザーに確認）
1. イベント名（例：Sake Night）
2. 開催日時
3. 参加費（円）
4. 定員（人数）

### 実行手順

#### Step 1: Stripeで決済リンクを作成
```python
import subprocess, json

STRIPE_KEY = "sk_live_..." # ユーザーに確認

# 商品を作成
product = subprocess.run(['curl', '-s', '-X', 'POST',
    'https://api.stripe.com/v1/products',
    '-u', f'{STRIPE_KEY}:',
    '-d', f'name=【イベント名】参加費',
    '-d', 'description=キャンセルポリシー：3日前まで全額返金 / 前日50%返金 / 当日返金なし'
], capture_output=True, text=True)

# 価格を設定
# 決済リンクを作成
```

#### Step 2: LINE Harnessの自動返信を更新
```bash
curl -X PUT "https://masu-salon.keisukendo414.workers.dev/api/automations/{参加申込ルールID}" \
  -H "Authorization: Bearer b406a08d..." \
  -H "Content-Type: application/json" \
  -d '{"actions": [{"type":"send_message","message":"参加申込ありがとうございます！\n\n以下から参加費をお支払いください。\n{STRIPE_PAYMENT_LINK}\n\nキャンセルポリシー：\n・3日前まで：全額返金\n・前日：50%返金\n・当日：返金なし\n\n支払い完了後、詳細をお送りします。\n— MaSU SALON"}]}'
```

#### Step 3: 登録済みユーザーへイベント案内を一斉配信
```bash
curl -X POST "https://masu-salon.keisukendo414.workers.dev/api/broadcasts" \
  -H "Authorization: Bearer b406a08d..." \
  -H "Content-Type: application/json" \
  -d '{
    "lineAccountId": "41eb2d20-bebb-4ad5-b153-d1f0e2f2c3ef",
    "message": "【次回イベント案内】\n\nイベント名：{名前}\n日時：{日時}\n定員：{定員}名\n参加費：¥{金額}\n\nご参加希望の方はメニューの「参加申込」からどうぞ。\n\n— MaSU SALON",
    "scheduledAt": null
  }'
```

#### Step 4: リマインダーを設定（イベント3日前・前日・当日朝）
管理画面「リマインダ」から手動設定 or APIで設定。

---

## サイト更新コマンド

「サイトを更新して」と言われたら：
```bash
cd "/Users/masuo/Desktop/MaSU SALON"
# index.html を編集後
cp index.html masu-salon.html
git add index.html masu-salon.html
git commit -m "更新内容"
git push
# → Vercelが自動デプロイ
```

## キャンセルポリシー（全イベント共通）
- 3日前まで：全額返金
- 前日：50%返金
- 当日：返金なし
