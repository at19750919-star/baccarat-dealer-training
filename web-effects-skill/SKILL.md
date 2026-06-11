---
name: web-effects
description: >-
  Make web UI look polished, premium, or impressive — add depth, glass, glow,
  and motion with pure HTML/CSS/SVG (no libraries). Use when the user wants a
  page/component to feel "high-end", "designed", "有設計感/質感", or asks for
  glassmorphism, liquid glass, neon/glow, animated gradient text, ambient
  background, grain/noise, spotlight/tilt/magnetic hover, glowing borders, or
  similar effects. Also covers a non-destructive "snapshot → restore" pattern
  for preset/effect pickers so applying then cancelling an effect never
  corrupts the user's original values.
---

# Web 視覺特效 / 質感打底

純 HTML/CSS/SVG，零套件。分兩部分：**(1) 零互動的氛圍質感**（打開就成立、手機也行，CP 值最高）、**(2) 互動特效**。最後附一個常被忽略的 **非破壞性預設** 寫法。

> 完整可玩示範在同資料夾 `fx-cookbook.html`（每個效果都有實時樣本 + 複製 CSS）。需要時直接打開參考。

## 心法
- **低互動的專案，質感來自「光、材質、字體、緩慢的環境動畫」，不是 hover。** 手機沒有 hover。
- 想讓畫面「不像 AI 罐頭」：給它**明確的美術方向**（顏色/字體/材質一致），而不是堆預設值。
- **最便宜的高級感 = 顆粒質感膜 + 頂部聚光**。先上這兩個。

## 1. 零互動氛圍質感（優先用這些）

**① 顆粒質感膜（最關鍵）**——整頁鋪一層極淡雜訊：
```css
.grain{position:fixed;inset:0;z-index:50;pointer-events:none;opacity:.5;mix-blend-mode:overlay;
  background-image:url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E")}
```

**② 頂部聚光 + 暗角**——給畫面一個光源方向，立刻立體：
```css
.light{position:fixed;inset:0;pointer-events:none;
  background:radial-gradient(120% 80% at 50% -10%,rgba(255,255,255,.10),transparent 55%),
             radial-gradient(140% 120% at 50% 50%,transparent 60%,rgba(0,0,0,.55))}
```

**③ 網格光暈背景（緩慢漂移）**：
```css
.mesh{position:fixed;inset:-25%;z-index:-1;filter:blur(75px);opacity:.5;
  background:radial-gradient(38% 38% at 22% 26%,#6d54e0,transparent),
            radial-gradient(34% 34% at 82% 22%,#1f9bb8,transparent),
            radial-gradient(40% 40% at 72% 82%,#c64f86,transparent);
  animation:drift 22s ease-in-out infinite alternate}
@keyframes drift{50%{transform:translate(3%,-3%) scale(1.08)}}
```

**④ 漸層流光標題（文字自動掃光）**：
```css
h1{background:linear-gradient(100deg,#fff 20%,#b79bff 45%,#7ee0ff 60%,#fff 80%);
  background-size:200% auto;-webkit-background-clip:text;background-clip:text;
  -webkit-text-fill-color:transparent;animation:shine 6s linear infinite}
@keyframes shine{to{background-position:-200% center}}
```
變體：彩虹流動（換成 7 色漸層、首尾同色才能無縫循環）；霓虹呼吸（`animation:pulse 1.6s ease-in-out infinite;@keyframes pulse{50%{opacity:.45}}`）。

**⑤ 緩慢呼吸浮動**：`animation:float 5s ease-in-out infinite;@keyframes float{50%{transform:translateY(-8px)}}`（多個元素用負 delay 錯開）。

**⑥ 自動旋轉漸層光邊（AI 感）**：
```css
@property --angle{syntax:'<angle>';initial-value:0deg;inherits:false}
.glow::before{content:"";position:absolute;inset:-2px;z-index:-1;border-radius:inherit;
  background:conic-gradient(from var(--angle),#7c5cff,#22d3ee,#ff5fa2,#7c5cff);
  animation:rot 5s linear infinite;filter:blur(6px)}
@keyframes rot{to{--angle:360deg}}
```

**⑦ 點陣背景紋理**：`background-image:radial-gradient(rgba(255,255,255,.10) 1px,transparent 1px);background-size:18px 18px`

**⑧ 捲動浮現**（唯一互動＝捲動）：`.reveal{opacity:0;transform:translateY(20px);transition:.7s}` + `IntersectionObserver` 加 `.in{opacity:1;transform:none}`。

## 2. 玻璃 / 液態玻璃

**玻璃磚（glassmorphism）核心配方 = 半透明 + 背景模糊 + 白高光邊 + 落地陰影**。⚠️ 背後要有**模糊的彩色光斑**才看得出毛玻璃感，純黑底直接放玻璃看不出來。
```css
.glass{background:linear-gradient(145deg,rgba(255,255,255,.13),rgba(255,255,255,.04));
  backdrop-filter:blur(16px);border:1px solid rgba(255,255,255,.16);
  box-shadow:inset 0 1px 1px rgba(255,255,255,.5),0 12px 34px -10px rgba(0,0,0,.6)}
```

**黑色液態玻璃（會真的折射背後內容）**——`backdrop-filter` 套 SVG 位移濾鏡。純 CSS `blur` 只是霧面、做不出「液態」。
```html
<svg width="0" height="0"><filter id="liquid">
  <feTurbulence type="fractalNoise" baseFrequency="0.012" numOctaves="2" seed="8" result="n"/>
  <feGaussianBlur in="n" stdDeviation="1.4" result="nb"/>
  <feDisplacementMap in="SourceGraphic" in2="nb" scale="40" xChannelSelector="R" yChannelSelector="G"/>
</filter></svg>
```
```css
.liquid{background:rgba(8,8,12,.28);backdrop-filter:blur(2px) url(#liquid);
  border:1px solid rgba(255,255,255,.16);
  box-shadow:inset 0 1.5px 1px rgba(255,255,255,.55),inset 0 -10px 24px rgba(0,0,0,.55),0 24px 60px -16px rgba(0,0,0,.7)}
```
> 相容性：折射在 **Chrome/Edge** 最完整；Safari 退成霧面、Firefox 不支援位移。實務上做「能折射就折射、不行就霧面」的降級，並提醒使用者。

## 3. 互動特效（桌面 hover；JS 都只有幾行）
- **游標聚光卡**：`onmousemove` 把座標寫進 `--x/--y`，`::after` 用 `radial-gradient(... at var(--x) var(--y))` 跟著走。
- **3D 傾斜 + 反光**：依游標算 `rotateX/Y`，反光層 `radial-gradient` 往反方向移；父層 `perspective`、本體 `transform-style:preserve-3d`。
- **磁吸按鈕**：`onmousemove` 把按鈕 `translate` 一個游標方向的分量，`mouseleave` 歸零；加 `transition:transform .15s`。
- **液態融合 metaball**：容器 `filter:url(#goo)`（`feGaussianBlur` + `feColorMatrix` 拉對比門檻），裡面放幾顆圓，靠近就黏合。
- **故障文字**：同字疊紅/青兩層 `::before/::after`，`clip-path:inset()` + 微位移抖動。
- **點擊噴彩**：點擊生成小方塊，隨機角度速度、每幀 `vy+=.35` 重力、`opacity` 漸減後移除。

## 4. 非破壞性「快照 → 還原」（做效果/預設選擇器時必用）
**問題**：套用預設時直接覆寫使用者原本的值（顏色/陰影…），清除時又沒還原 → 取消後原值跑掉。
**解法**：第一次「從無效果 → 套第一個」時拍快照，清除時還原；連續換效果不重拍。
```js
function applyPreset(p){
  if (active === null) snapshot = { color, shadow, stroke /* …使用者原本的值 */ };
  // …套用 p…
  active = p.name;
}
function clearPreset(){
  if (snapshot){ ({color, shadow, stroke} = snapshot); snapshot = null; } // 還原
  else { /* 回出廠預設 */ }
  active = null;
}
```
延伸：把「單一重點色」的效果做成**參數化** `recolor(accentColor)`，給一個色盤就能整組換色（neon 邊框/光暈最適合）。

## 使用建議
1. 先問畫面要什麼調性（暗色精品？日系清透？科技感？）再選效果，不要全部疊。
2. 低互動專案：第 1 區（氛圍質感）為主，至少上 ①顆粒 + ②聚光。
3. 字體有個性（襯線大標 / Space Grotesk）對「設計感」幫助極大。
4. 要看實際長相與可複製碼，打開 `fx-cookbook.html`。
