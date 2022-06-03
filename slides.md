---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# SWRとSWRV

---

# 今日のひとこと

- 川崎のハードコアフェスに行ってきました
  - 11:45 ~ 21:00まで14バンドを堪能
  - 手足ブンブン丸が大量発生していて怖かったです

<video 
  src="/movie.mp4" 
  style="width:600px; height:300px;" type="video/mp4" controls></video>

---

# Agenda

- SWRとSWRVとは
- stale-while-revalidate
- SWRの概要/特徴
- SWRVの概要/特徴

---

# SWRとは

- HTTP RFC5861で提唱されたHTTPキャッシュ無効化戦略である `stale-while-revalidate` に由来している
- まずはキャッシュからデータを返し（stale）、次にフェッチリクエストを送り（revalidate）、最後に最新のデータを持ってくるという戦略
- ⭐ GitHub Star 22.5k

---

# SWRVとは

- VueのComposition APIで使えるリモートデータフェッチング
- 大本はSWR
- ⭐ GitHub Star 1.6k

---

# stale-while-revalidate

- Web領域では、HTTPヘッダーを使ってブラウザやプロキシにキャッシュの制御を指定するが、 `state-while-revalidate` ヘッダーはこのキャッシュ制御に選択肢を追加した
- キャッシュを行う意義
  - リソースの取得を高速化する
  - サーバーへの負荷を減らす

---

# stale-while-revalidate

- キャッシュの指定方法には大きく二つ
  - ブラウザはリクエストを発行せず、保持するキャッシュを使用する（ `Cache-Control` , `Expires` ）
  - ブラウザはリクエストを発行し、サーバーにキャッシュの有効性を確認してから、キャッシュを使用する（ `ETag` , `Last-Modified` ）

---

# stale-while-revalidate

- また、キャッシュは「再利用」を行う目的でありながら、ある一定の範囲で「更新」を行いたいという、相反するコントロールが求められる
- ここが、キャッシュ設計の難しい点

---

# stale-while-revalidate

> - ブラウザはリクエストを発行せず、保持するキャッシュを使用する（ `Cache-Control` , `Expires` ）
> - ブラウザはリクエストを発行し、サーバーにキャッシュの有効性を確認してから、キャッシュを使用する（ `ETag` , `Last-Modified` ）

上記2つの仕組みでは、「キャッシュは効かせたいが、なるべく新鮮なリソースを提供したい」などの要望に対処するのが難しかった<br>


=> そこで `stale-while-revalidate` の出番

---

# stale-while-revalidate

- 「キャッシュから表示するが、裏で非同期にキャッシュを更新しておく」という仕組み

---

# stale-while-revalidate

## max-age の例

```
Cache-Control: max-age=3600;
```

- fetchしたレスポンスは、3600sの間は *fresh* とみなされ、その期間はキャッシュ化される
- 3600sを過ぎると、キャッシュは *stale* とみなされ破棄し、次のリクエストでfetchが走る

---

# stale-while-revalidate

## max-age の例

```
Cache-Control: max-age=3600, stale-while-revalidate=360
```

- 3600s経過後、そこから360sは *stale* なキャッシュを引き続き使用する
- 一度 *stale* なキャッシュを使用したら、裏で非同期にfetchを行い、サーバーにキャッシュの鮮度を問い合わせる
- もしサーバーから新しいリソースをfetchしたら、そこに付与された新しいヘッダーに従ってキャッシュを更新する

---

# stale-while-revalidate

## max-age の例

つまり、 `max-age` のみ設定していた頃よりも、キャッシュを効かせつつ新鮮なリソースを保てる

---

# SWRの概要

```ts
import axios from 'axios'
import useSWR from 'swr'

const fetcher = url => axios.get(url).then(res => res.data)

function Profile() {
  const { data, error } = useSWR('/api/user', fetcher)

  if (error) return <div>failed to load</div>
  if (!data) return <div>loading...</div>
  return <div>hello {data.name}!</div>
}
```

- `useSWR` フックは `key` 文字列と `fetcher` 関数を受け取る
- `key` はデータの一意な識別子（通常はAPIのURL）で、 `fetcher` に渡される
- `fetcher` はデータを返す任意の非同期関数で、fetchやaxios、GraphQLなどを使うことができる
- リクエストの状態にもとづいて `data` と `error` の二つの値を返す

---

# SWRの特徴

- 高速なページナビゲーション
- 定期的なポーリング
- データの依存関係
- フォーカス時の再検証
- ネットワーク回復時の再検証
- ローカルキャッシュの更新
- スマートなエラーの再試行
- ページネーションとスクロールポジションの回復
- React Suspense

---

# SWRVの細かな違い

### swrv では `data`, `error`, `isValidating` が Ref でラップされている

```vue
<script setup>
import useSWRV from "swrv";

const { data } = useSWRV("/api/user", fetcher);

const greete = () => {
  if (!data.value) {
    return;
  }
  alert(`Hello, my name is ${data.value.name}.`);
};
</script>

<template>
  <div v-if="!data">Loading...</div>
  <div v-else>
    <h1>My name is {{ data.name }}.</h1>
    <button @click="greete">greete</button>
  </div>
</template>
```

---

# SWRVの細かな違い

### SWRVのバウンドミューテートで渡せる引数が違う

- ミューテートとは
  - SWRの場合 `mutate` に文字列を渡すことでrefetch(再取得)を実行できるよう指示すること
- バウンドミューテートとは
  - SWRの場合、一部のデータのみをrefetchするなど細かな設定ができる

---

# SWRVの細かな違い

### SWRVのバウンドミューテートで渡せる引数が違う

```vue
<script setup>
import useSWRV from "swrv";

const { data, mutate } = useSWRV("/api/user", fetcher);

const handleClick = () => {
  // 誤り: mutate({ name: "Bob" })
  mutate(() => ({ name: "Bob" }));
};
</script>
```

---

# SWRVの細かな違い

と、まあ他にも色々挙動の違いがあるので気になる方は以下をチェック！

https://zenn.dev/mascii/articles/difference-between-swr-and-swrv