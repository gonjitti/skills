# 良いテストと悪いテスト

## 良いテスト (Good Tests)

**統合テストスタイル (Integration-style)**: 内部パーツのモックではなく、実際のインターフェースを介してテストします。

```typescript
// 良い例: 観測可能な振る舞いをテストしている
test("有効なカートでチェックアウトできること", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

特徴：

- ユーザーや呼び出し元が気にする「振る舞い」をテストしている
- 公開API（Public API）のみを使用している
- 内部的なリファクタリングを行っても生存し続ける（テストが壊れない）
- 実装方法（HOW）ではなく、何を行うか（WHAT）を説明している
- 1つのテストにつき、論理的なアサーションは1つに留めている

## 悪いテスト (Bad Tests)

**実装詳細のテスト**: 内部のコード構造に密結合しています。

```typescript
// 悪い例: 実装の詳細をテストしている
test("チェックアウト時に paymentService.process が呼び出されること", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

警告サイン（レッドフラグ）：

- 内部の連携モジュールをモックしている
- 非公開（private）メソッドをテストしている
- 呼び出し回数や呼び出し順序をアサート（検証）している
- 振る舞いは変わっていないのに、リファクタリングを行うとテストが壊れる
- テスト名が振る舞い（WHAT）ではなく実装手順（HOW）を記述している
- インターフェースを介さず、外部の手段で状態を検証している

```typescript
// 悪い例: インターフェースをバイパスして状態を検証している
test("createUser を実行するとデータベースに保存されること", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// 良い例: インターフェースを介して検証している
test("createUser を実行すると、ユーザーを取得できるようになること", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```
