# マッピング

モデルの構造を記載に資料についてはSP-APIのOpenAPIスペックを参照

## マッピング

### 前提

```ts
// 注文詳細

// 単一注文
const order: OrderItems = "getOrder: /orders/2026-01-01/orders/{orderId}";

// 最初の注文アイテム
const orderItem: OrderItem = order.orderItems[0];

// 最初の決済情報
const payment: PaymentExecution = order.payment.paymentExecutions[0];

// 最初のパッケージ
const package: OrderPackage = order.packages[0];
```

### 注文詳細

#### 基本情報

- Amazon注文番号: `order.orderId`
- Amazonステータス: `order.fulfilment.fulfilmentStatus`
- 注文番号: `order.orderId`
- 注文日時: `order.createdTime`
- ステータス: `order.fulfilment.fulfilmentStatus`
- Amazon警告: 説明が欲しいです **検討中**


#### 注文主情報

- 名前: `order.buyer.buyerName`
- 郵便番号: `order.recipient.deliveryAddress.postalCode`
- 住所: `#{order.recipient.deliveryAddress.stateOrRegion} #{order.recipient.deliveryAddress.city} #{order.recipient.deliveryAddress.districtOrCounty}` または `order.recipient.deliveryAddress.addressLine1`
- メールアドレス: `order.buyer.buyerEmail`


#### お届け先情報

- 名前: `order.buyer.buyerName`
- 郵便番号: `order.recipient.deliveryAddress.postalCode`
- 住所: `#{order.recipient.deliveryAddress.stateOrRegion} #{order.recipient.deliveryAddress.city} #{order.recipient.deliveryAddress.districtOrCounty}` または `order.recipient.deliveryAddress.addressLine1`
- 電話番号: `order.recipient.deliveryAddress.phone`
- 配送希望日: `order.fulfilment.deliverByWindow`（範囲）
- 配送希望時間帯: B2Cの場合は検知できません **検討中**


#### 商品情報

- 商品名: `orderItem.product.title`
- Amazon商品管理番号: `orderItem.product.sellerSku` とかでしょうか **検討中**
- Amazon掲載ステータス: 👇

```ts
// Listing API を使用
const marketplaceId: string = "...";
const item: Item = await fetch("/listings/2021-08-01/items/{sellerId}/{sku}");

const japanMarketplaceStatus: ItemSummaryByMarketplace = item?.summaries
  .filter(s => s.marketplaceId === marketplaceId)[0];

const status: string = japanMarketplaceStatus?.status ?? '非掲載'
```

- ~~EC注文フラグ~~
- ~~EC画像フラグ~~
- ~~Amazon商品連携ステータス~~
- Amazon商品連携ステータス: **検討中** 仕組みがわかりません
- ~~(楽天）画像連携ステータス~~
- Amazon画像連携ステータス: **検討中** 仕組みがわかりません
- 販売価格: `orderItem.product.price.unitPrice`(税抜)


#### スペック情報

- ~~商品識別番号~~
- ~~インストアコード~~
- ~~新中区分~~
- ~~番手~~
- ~~メーカー名~~
- ~~モデル名~~
- ~~シャフト~~
- ~~番手~~
- ~~ロフト~~
- ~~硬さ~~
- ~~長さ~~
- ~~バランス~~
- ~~総重量~~
- ~~販売ランク~~
- ~~リシャフトフラグ~~
- ~~ヘッドカバー~~
- ~~リグリップ~~
- ~~付属品~~


#### 決済情報

- 支払方法: `payment.paymentMethod`
- 商品金額: `payment.paymentAmount.amount`
- ~~送料、ラッピング料、決済手数料~~
- ポイント利用額: 👇

```ts
const pointsMonetaryValue = order?.orderItems
  ?.map(i => i.expense.pointsCost.pointsGranted.pointsMonetaryValue.amount)
  .reduce((total, next) => total + next, 0) ?? 0;
```

- クーポン利用総額: 👇

```ts
/** @type Array<{ promotionId: string, description: string }> */
const promotions: ItemPromotionBreakdown[] = order.orderItems.flatMap(i => i.promotion.breakdowns);
// プロモーションの詳細情報の取得方法は知りません
```

- 合計金額: `order.proceeds.grandTotal`


#### 備考

- ...


#### 配送内容

- 出荷日: `package.shipTime`
- 配送伝票番号: **検討中** Shipment API を用いて生成された伝票情報を読み込むか、手動で設定


### 注文一覧

#### 前提

```ts
const orders: SearchOrdersResponse = await fetch("/orders/2026-01-01/orders")
  .then(res => res.json)
  .then(data => data.orders);
const order: Order = orders[0];
const orderFinancialEvents:  = await fetch(`/finances/v0/orders/${order.orderId}/financialEvents`)
  .then(res => res.json)
  .then(data => data.payload.);
```

#### 絞り込み

```ts
// 注文IDと送付電話番号と絞り込みキーワードとPOS注文ステータスでの
// 絞り込みはアプリサーバーで行う
type AmazonFulfilmentStatuses = 'PENDING' | 'SHIPPED' | 'CANCELLED' ...;
interface OrdersSearchQueryParamaters {
  lastUpdatedBefore: DateTime,                    // 範囲（最古）
  lastUpdatedAfter: DateTime,                     // 範囲（最新）
  fulfilmentStatus: AmazonFulfilmentStatuses[],   // アマゾンステータス
  maxResultsPerPage: 20 | 50 | 100,               // ページサイズ
  paginationToken: string,                        // （ページサイズを指定した場合必須）
}

const queryParams = new URLSearchParams({...} as OrdersSearchQueryParameters);
const orders = await fetch(`https://foo.bar/baz?${queryParams}`);
```

#### 一覧

- ~~画像~~ (POS)
- 注文日時: `order.createdTime`
- アマゾン注文番号: `order.orderId`
- 注文番号: `order.orderId`
- ~~店舗~~: 店舗の概念はない
- AMAZONステータス: `order.fulfilment.fulfilmentStatus`
- ~~ステータス~~
- ~~単品識別番号~~
- 注文主名: `order.buyer.buyerName`
- 発送日: **検討中**

## 参考
