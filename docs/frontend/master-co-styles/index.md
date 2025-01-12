# Intro

作者從2004年就開始撰寫及維護CSS，並於2021年底釋出了一個以Virtual CSS為底的CSS框架

### Virtual CSS

> Virtual CSS is the concept of expressing CSS rules through class syntax.
> 

Virtual CSS 是一種藉由 Class 語法來表達 CSS 規則的概念

其原理就是將一個虛擬的樣式表保存在記憶體中，並即時與實際的 CSS 樣式表同步，核心是使用 Mutation Observer，會觀察 DOM 樹並透過一個特殊的規則引擎去遍歷每個 class，將其轉換為 CSS 並注入至 style。

![1](/img/frontend/master-co-styles/1.png)

### Traditional CSS

Inline Styles

```jsx
<div style="background-color: red;">Background will be red</div>
```

優點：簡單暴力、不必尋找給予樣式的來源、權重僅小於 !important

缺點：無法複用、無法根據斷點撰寫不同 CSS

class + .css 檔案/<style>標籤

```jsx
// index.html
<h1 class="title"></h1>

// style.css
.title {
    font-size: 1rem;
}
.title:hover {
    font-size: 1.5rem;
}
@media (min-width: 768px) {
    .title {
        font-size: 1.25rem;
    }
}
@media print {
    .title {
        display: none;
    }
}
```

優點：可複用、可控制權重.....太多了，可以說是現在最主流的寫法沒有之一

缺點：撰寫速度較慢

```jsx
<h1 class="font:16 font:24:hover font:20@sm hide@print"></h1>
```

### Atomic CSS

將 CSS 語法轉變為簡易的 class 並大量複用

```jsx
// index.html
<div class="D(f) W(100px)"></div>

// style.css
.D\(f\) {
  display: flex;
}

.W\(300px\) {
  width: 100px;
}
```

優點：大量複用重複CSS，在成熟的架構下開發相對快速

缺點：須仰賴其他工具、可能造成大量樣式未使用的狀況

### CSS-in-JS

在 JS 中撰寫 CSS

```jsx
// React
const Title = styled.h1`
  font-size: 1.5em;
  text-align: center;
  color: palevioletred;
`;

class App extends React.Component {
  render() {
    return <Title>Hello World!</Title>;
  }
}

// Vue
<template>
  <h1 :class="$style.title">Hello World</h1>
</template>

<style module>
.title {
  font-size: 1.5em;
  text-align: center;
  color: palevioletred;
}
</style>
```

優點：局部的樣式解決權重問題、可擴展性高、組件能帶著走

缺點：學習曲線高、需嚴謹規範

### 其他撰寫 CSS 時的痛點

**過早的抽象化**

除非有非常充裕的時間，通常在開發時較難找出每個區塊樣式的共同點和重複的部分，如果前期就大量進行了不準確的抽象化，容易降低頁面配合需求變動的靈活性，若樣式大量被使用也可能會有修改不便甚至改A壞B的隱憂。

**取名的時間成本**

假設我們要為一個樣式命名，通常我們會看：

- 這個區塊應該被稱為什麼？
- 先來看看內容有些什麼...
- 好，看起來像是 xxx

```jsx
// style.css
.xxx {
  font-size: 24px;
  /* ... */
}
```

**維護專案 CSS 所在的時間成本**

在維護專案時，尋找 CSS 容易耗費額外的時間，通常需要仰賴瀏覽器 devtool 和編輯器的搜尋功能來降低尋找時間。

**入門時的學習曲線高**

CSS-in-JS、Atomic CSS 本身都有自己的學習曲線，即使是使用傳統的 CSS 建構的專案，首次進入時也得花時間閱讀專案文件或靠自己觀察出 CSS 的檔案架構。

### CSS-in-Class

**CSS in Class 解決了...**

- 無須抽象化，直接寫就對了
- 寫出像是 CSS 的 class 名稱
- 不必查找大量 CSS 花費額外時間成本
- 學習曲線低

### 範例：

CDN

```jsx
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://unpkg.com/@master/normal.css" rel="stylesheet">
    <script src="https://unpkg.com/@master/style"></script>
    <script src="https://unpkg.com/@master/styles"></script>
</head>
<body>
    <h1 class="font:40 font:heavy font:italic m:50 text:center">Hello World</h1>
</body>
```

NPM

```jsx
// 安裝指令
npm install @master/styles @master/normal.css

// .js檔
import '@master/styles';

// .css檔
@import '@master/normal.css';
```

### 語法範例1

可讀性高的寫法

```jsx
<p class="text-align:center font-size:1rem background-color:red-50">Lorem ipsum dolor sit amet.</p>
```

簡潔的寫法

```jsx
<p class="text:center font:16 background:red-50">Lorem ipsum dolor sit amet.</p>
```

更簡潔的寫法

```jsx
<p class="t:center f:16 bg:red-50">Lorem ipsum dolor sit amet.</p>
```

### 語法範例2

可讀性高的寫法

```jsx
<p class="animation-name:fade animation-duration:1s animatino-timing-function:ease transition-property:opacity transition-duration:.3s margin-top:1rem margin-bottom:1rem margin-right:2rem margin-left:2.5rem">Lorem ipsum dolor sit amet.</p>
```

簡潔的寫法

```jsx
<p class="animation:fade;1s;ease transition:opacity;.3s margin:16;32;16;40">Lorem ipsum dolor sit amet.</p>
```

更簡潔的寫法

```jsx
<p class="@fade;1s;ease ~opacity;.3s m:16;32;16;40">Lorem ipsum dolor sit amet.</p>
```

@為animation簡寫；~為transition簡寫；;則將不同的CSS屬性區分開來

### !important

```jsx
<p class="text:right text:center!">I will be centered!</p>
```

### 偽元素、偽類別

```jsx
<button class="bg:blue-47 bg:blue-54:hover bg:blue-68:active bg:blue-54:focus">Click Me!</button>
```

### calc, var

可讀性高的寫法

```jsx
<div class="width:calc(100%-60px) height:var(--size)">Guess my height!</div>
```

簡潔的寫法

```jsx
<div class="w:calc(100%-60px) h:$(size)">Guess my height!</div>
```

### 顏色

可讀性高的寫法

```jsx
<div class="font:var(--blue-60)">Blue with 0.5 opacity</div>
```

簡潔的寫法

```jsx
<div class="font:blue-60">Blue with 0.5 opacity</div>
```

### 斷點

可以這樣寫

```jsx
<div class="grid-cols:4@<768 grid-cols:5@<1280 grid-cols:7 gap:16">
  <div class="bg:red p:4">1</div>
  <div class="bg:orange p:4">2</div>
  <div class="bg:yellow p:4">3</div>
  <div class="bg:green p:4">4</div>
  <div class="bg:blue p:4">5</div>
  <div class="bg:aqua p:4">6</div>
  <div class="bg:purple p:4">7</div>
</div>
```

或是使用框架定義好的寬度

```jsx
<div class="grid-cols:4@<sm grid-cols:5@<lg grid-cols:7 gap:16">
  <div class="bg:red p:4">1</div>
  <div class="bg:orange p:4">2</div>
  <div class="bg:yellow p:4">3</div>
  <div class="bg:green p:4">4</div>
  <div class="bg:blue p:4">5</div>
  <div class="bg:aqua p:4">6</div>
  <div class="bg:purple p:4">7</div>
</div>
```

### 複用樣式

```jsx
// src\classes\outline-button.js
export const OUTLINE_BUTTON_CLASS = `
    b:1px;solid;#000 p:12 r:4 bg:aqua
`;

// Vue (以Vue Cli為例)
// some-random-component.vue
<template>
  <div>
    <button :class="OUTLINE_BUTTON_CLASS">Hover me!</button>
  </div>
</template>

<script>
import { OUTLINE_BUTTON_CLASS } from "../classes/outline-button";
export default {
  name: "HelloWorld",
  data() {
    return {
      OUTLINE_BUTTON_CLASS: OUTLINE_BUTTON_CLASS,
    };
  },
};
</script>

// main.js
import "@master/normal.css";
import "@master/styles";

// --------
// React (以create-react-app為例)
// src/App.js
import { OUTLINE_BUTTON_CLASS } from "./classes/outline-button";
import "./App.css";

function App() {
  return (
    <div className="App">
      <button className={OUTLINE_BUTTON_CLASS}>Imma leave my door open</button>
    </div>
  );
}

export default App;

// index.js
import "@master/normal.css";
import "@master/styles";
```

### 優缺點

優點

- 非常輕量，所需的資源總容量不超過13KB
- 極低的導入成本，可不依賴打包工具
- CSS撰寫非常快速
- 可與其他 CSS 框架並行（須注意原本專案是不是就有 reset.css 或 normalize.css）

缺點

- 核心需仰賴 JavaScript 驅動，在不允許 JS 的瀏覽器會完全失效
- 複用樣式需要仰賴打包工具，部分架構如 Laravel 目前尚不知複用架構的做法，且若複用樣式相較傳統 CSS 寫法我覺得沒那麼好
- 尚無法完全捨棄 <style> 或 .css，如若需要自定義 keyframes 或 CSS 變數依然需要用原生的寫法寫
- 若標籤的樣式過多，可能造成可讀性差的問題

適合

- 走快速開發且注重載入效能的頁面
- 沒有 Webpack 或其他打包工具的專案

不適合

- 需支援舊瀏覽器的頁面
- 已有高度模組化 CSS 的專案

可嘗試的情境

- 一次性的活動頁
- 小型專案