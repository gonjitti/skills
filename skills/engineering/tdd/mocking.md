# モックを使用するタイミング

モック化（Mocking）は、**システムの境界（system boundaries）** でのみ行ってください：

- 外部API（決済、メール送信など）
- データベース（場合による — テスト用のデータベースを用意する方を優先します）
- 時間 / 乱数
- ファイルシステム（場合による）

以下はモック化しないでください：

- あなた自身が作成したクラス / モジュール
- 内部の連携モジュール（internal collaborators）
- あなた自身がコントロール（編集）できるすべてのもの

## モックしやすさを考慮した設計 (Designing for Mockability)

システムの境界では、モック化しやすいインターフェースを設計します：

**1. 依存性注入 (Dependency Injection) を使用する**

外部の依存関係を内部でインスタンス化するのではなく、外部から渡すようにします：

```typescript
// モックしやすい例
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// モックしにくい例
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

**2. 汎用的なデータ取得関数ではなく、SDKスタイルのインターフェースを優先する**

条件分岐ロジックを含む1つの汎用的な関数を作る代わりに、外部操作ごとに専用の関数を作成します：

```typescript
// 良い例: 各関数が個別にモック可能です
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// 悪い例: モックの内部で条件分岐ロジックを書く必要があります
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

SDKアプローチの利点：
- 各モックが特定のデータ構造を1つだけ返せばよくなります
- テストのセットアップで条件分岐ロジックを書く必要がなくなります
- テストがどのエンドポイントを実行しているかが明確になります
- エンドポイントごと型安全性が確保されます
