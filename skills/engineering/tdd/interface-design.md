# テスト容易性を高めるインターフェース設計

優れたインターフェースは、テストを自然なものにします：

1. **依存関係は自ら生成せず、外部から受け取る**

   ```typescript
   // テストしやすい例
   function processOrder(order, paymentGateway) {}

   // テストしにくい例
   function processOrder(order) {
     const gateway = new StripeGateway();
   }
   ```

2. **副作用を起こさず、結果を戻り値として返す**

   ```typescript
   // テストしやすい例
   function calculateDiscount(cart): Discount {}

   // テストしにくい例
   function applyDiscount(cart): void {
     cart.total -= discount;
   }
   ```

3. **公開インターフェースの露出面積を小さくする**
   - メソッドの数を減らす ＝ 必要なテストケースが減少する
   - パラメータをシンプルにする ＝ テストデータの準備（セットアップ）が容易になる
