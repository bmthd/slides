---
marp: true
title: TanStack DB
description: TanStack DB の紹介
style: |
  section {
    background: #cbdcf7;
    color: #2c3e50;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    padding: 60px 80px 80px 80px;
    display: flex;
    flex-direction: column;
  }
  
  h1 {
    color: #2d3748;
    text-shadow: none;
    border-bottom: 3px solid #2d3748;
    padding-bottom: 10px;
    position: absolute;
    top: 60px;
    left: 80px;
    right: 80px;
    margin: 0;
  }
  
  h2 {
    color: #2d3748;
    border-left: 4px solid #2d3748;
    padding-left: 15px;
    position: absolute;
    top: 60px;
    left: 80px;
    right: 80px;
    margin: 0;
  }
  
  section:has(h1) {
    padding-top: 160px;
  }
  
  section:has(h2) {
    padding-top: 140px;
  }
  
  ul {
    line-height: 1.8;
  }
  
  li {
    margin-bottom: 8px;
  }
  
  strong {
    color: #1a365d;
    font-weight: bold;
  }
  
  code {
    background: #1F1F1F;
    padding: 2px 6px;
    border-radius: 4px;
    color: #CCCCCC;
    font-family: 'Cascadia Code', 'Fira Code', monospace;
  }
  
  pre {
    background: #1F1F1F;
    border-radius: 8px;
    padding: 20px;
    border-left: 4px solid #2d3748;
  }
  
  pre code {
    background: transparent;
    color: #CCCCCC;
  }
  
  .language-ts .hljs-keyword,
  .language-tsx .hljs-keyword {
    color: #C586C0;
  }
  
  .language-ts .hljs-string,
  .language-tsx .hljs-string {
    color: #CE9178;
  }
  
  .language-ts .hljs-function,
  .language-tsx .hljs-function {
    color: #DCDCAA;
  }
  
  .language-ts .hljs-variable,
  .language-tsx .hljs-variable {
    color: #9CDCFE;
  }
  
  .language-ts .hljs-comment,
  .language-tsx .hljs-comment {
    color: #6A9955;
  }
  
  .language-ts .hljs-type,
  .language-tsx .hljs-type {
    color: #4EC9B0;
  }
  
  .language-ts .hljs-number,
  .language-tsx .hljs-number {
    color: #B5CEA8;
  }
  
  .language-ts .hljs-title,
  .language-tsx .hljs-title {
    color: #DCDCAA;
  }
  
  .language-ts .hljs-params,
  .language-tsx .hljs-params {
    color: #9CDCFE;
  }
  
  blockquote {
    border-left: 4px solid #2d3748;
    padding-left: 15px;
    font-style: italic;
    opacity: 0.9;
  }
---

<h1 style="position: static; margin: 0 auto; max-width: fit-content; text-align: center;">TanStack DB ~状態管理の新しい考え方~</h1>

---

## TanStackファミリー

<div style="display: flex; align-items: center; gap: 40px;">
  <div style="flex: 1; text-align: center;">
    <img src="tanstack-logo.png" alt="TanStack Logo" style="width: 240px; height: auto;" />
  </div>
  <div style="flex: 2;">
    <ul>
      <li><strong>TanStack Query</strong> - Server State Management</li>
      <li><strong>TanStack Router</strong> - Type-Safe Routing</li>
      <li><strong>TanStack Table</strong> - Headless Table Building</li>
      <li><strong>TanStack Form</strong> - Headless Form Building</li>
      <li><strong>TanStack DB</strong> - Client-Side Database ← <strong>NEW!</strong></li>
    </ul>
  </div>
</div>

---

# TanStack DBとは？

* フロントエンドに **永続化層(DB)** を設けてコンポーネントからクエリする
* **クエリファーストなAPI**
* React だけでなく **Vue / Svelte / Solid** など多数のUIフレームワークをサポート

---

# 設計思想

**Client Side Querying**
  → データ取得の主導権をフロント側へ

---

# Reactでの使い方の例

```ts
export const drawCalcCollection = createCollection(
  localStorageCollectionOptions({
    storageKey: "tcg-tool-draw-calculations",
    id: "draw-calculations",
    getKey: (item) => item.id,
    schema: drawCalcSchema
  }),
);
```

* Collectionという単位でデータの永続化を定義
* ZodやValibotなどでランタイムバリデーションありの型定義可能

---

```ts
const { data } = useLiveQuery((q) =>
  q
    .from({ calculations: calculationsCollection })
    .where(({ calculations }) => eq(calculations.id, calculationId)),
);
```

* DrizzleのようなSQLライクなセレクター
* 型安全なクエリビルダー
* クエリで絞り込まれた箇所のみの部分的な**Subscribe**を実現

---

## コンポーネントで使う

```tsx
const CalculationItem = ({ calculationId }: { calculationId: string }) => {
  const { data } = useLiveQuery((q) =>
    q.from({ calculations: calculationsCollection })
      .where(({ calculations }) => eq(calculations.id, calculationId)),
  );
  const firstCalculation = data[0];
  if (!firstCalculation) return <FallbackItem />;
  return (
    <Card.Root>
      <Card.Header>
        {firstCalculation.name}
      </Card.Header>
      <Card.Body>
        {firstCalculation.result}
      </Card.Body>
    </Card.Root>
  );
}
```

---

## 派生Collectionも可能

```ts
// 条件をあらかじめ指定
export const drawCalcHistoryCollection = createLiveQueryCollection((q) =>
  q.from({ calculations: drawCalcCollection })
  .orderBy(({ calculations }) => calculations.updatedAt, "desc"),
);

// 引数ありの動的なCollection
export const createDrawCalcByGameCollection = (gameTemplate: string) =>
  createLiveQueryCollection((q) =>
    q.from({ calculations: drawCalcCollection })
    .where(({ calculations }) => eq(calculations.gameTemplate, gameTemplate))
    .orderBy(({ calculations }) => calculations.updatedAt, "desc"),
  );
```

---

<h1 style="position: static; margin: 0 auto; max-width: fit-content; text-align: center;">TanStack DB、なにが美味しい？</h1>

<!-- * 永続化層を **自由に差し替え可能**
* オンメモリだけでなく、Electric SQL
* RLS（Row Level Security）と組み合わせることで都度問い合わせすることなくクライアントが自由にデータを加工可能 -->

---

## 永続化層を **自由に差し替え可能**

* localStorage
* インメモリ
* REST API
* Electric SQL
* TrailBase

Supabase, IndexedDB, DuckDB WasmなどもAdapterを用意することで利用可能になるかも


---

# バックエンド実装が不要に？

* **フロントエンド中心のデータ設計** が可能
* RLS（Row Level Security）を組み合わせれば都度問い合わせの必要なし
* 特定の データベースSaaS の実装に依存せずに済む

---

# 状態管理のベストプラクティスを変える？

* Zustand / Jotai のような状態を
  → **SQLライクなクエリ + スキーマ** に置き換え可能
* 単なる状態管理にとどまらず、データの保管にも利用できる
* **認知負荷を大幅に削減** できるかも！

---

# おまけ

<div style="margin-top: 100px;">
  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 40px; align-items: start;">
    <div>
      <p style="margin-bottom: 30px; font-size: 1.1em; margin-top: 20px;">Claude Codeと一緒に既存のアプリケーションをTanStack DBに載せ替えしてみた</p>
      <img src="image-2.png" alt="サービス画像" style="width: 100%; height: auto; max-height: 330px; object-fit: contain; border-radius: 8px;" />
    </div>
    <div style="display: flex; flex-direction: column; justify-content: flex-start; padding-top: 20px;">
      <div style="display: flex; align-items: center; gap: 20px; margin-bottom: 20px;">
        <img src="image-1.png" alt="QRコード" style="width: 140px; height: auto; border-radius: 8px;" />
        <div>
          <h3 style="margin-bottom: 8px; color: #2d3748;">できたもの</h3>
          <p style="font-size: 0.85em; color: #2d3748; line-height: 1.3;">https://tcg-tool.pages.dev/<br>draw-calc-db</p>
        </div>
      </div>
      <div style="display: flex; align-items: center; gap: 20px;">
        <img src="image.png" alt="GitHub PR" style="width: 140px; height: auto; border-radius: 8px;" />
        <div>
          <h3 style="margin-bottom: 8px; color: #2d3748;">ソースコード見たい方向け</h3>
          <p style="font-size: 0.85em; color: #2d3748; line-height: 1.3;">https://github.com/bmthd/<br>tcg-tool/pull/3/files</p>
        </div>
      </div>
    </div>
  </div>
</div>

---

# まとめ

* 「**フロントエンド中心のデータ設計**」を加速させる存在
* 全てのServer Stateの抽象化層になりうる
* フロントエンドの状態管理だけでなくバックエンド設計にも影響を与える可能性大
