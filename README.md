# 讓導覽鈕隨著swiper輪播項目縮放時改變位置
<br>
以swiper為範例，讓使用絕對定位的左右兩側navigation按鈕，能隨RWD改變正在輪播的主item(active)改變視窗時，左右兩側的navigation按鈕能更隨主item移動。
<br>

---
demo：https://icemakerhuang.github.io/swiper-navigationRWD/
<br>

---
code：<br>
```js
    // 決定按鈕離當下輪播卡片(active)有多遠
    // 往外拓展的比例（3% 就填 0.03）
    const EXPAND_RATIO = 0.05;

    // === afterLayout：等畫面真的排好再動手量 ===
    // 有瀏覽器和 Swiper 需要時間把東西排好（例如置中、位移）
    // 用「兩次 requestAnimationFrame」等到「下一下一幀」再執行，
    // 這樣量到的數字才是「畫面已經準備好」的正確位置
    const afterLayout = (fn) => requestAnimationFrame(() => requestAnimationFrame(fn));

    // === updateNavOffset：把左右按鈕放在對的位置 ===
    // 按鈕要放在「主item（.swiper-slide-active）」左右兩邊
    function updateNavOffset() {
      // 1) 找到外層容器（定位的基準）和目前在中間的那張卡片（active）
      const container = document.querySelector('.mySwiper');
      const active = container?.querySelector('.swiper-slide-active');
      if (!container || !active) return; // 找不到就先不做

      // 2) 量「容器」和「active 卡片」在螢幕上的位置與大小（會包含 CSS scale）
      const cRect = container.getBoundingClientRect();
      const sRect = active.getBoundingClientRect();

      // 3) 計算「往外一點點」的距離（px）
      //    這是用「active 卡片的寬度 * EXPAND_RATIO」得到的
      const expandPx = sRect.width * EXPAND_RATIO;

      // 4) 決定左右按鈕要放在哪個「X 座標」
      //    左按鈕：落在 active 左邊再往「左」移 expandPx 距離
      const leftTargetX = sRect.left - expandPx;
      //    右按鈕：落在 active 右邊再往「右」移 expandPx 距離
      const rightTargetX = sRect.right + expandPx; // 這些是根據螢幕的絕對位置

      // 5) 把「螢幕的絕對位置」換算成「容器裡的百分比」
      //    因為按鈕的 CSS 用的是 left:% / right:%，要跟容器寬度比
      const leftOffsetPercent  = ((leftTargetX  - cRect.left)  / cRect.width) * 100;
      const rightOffsetPercent = ((cRect.right - rightTargetX) / cRect.width) * 100;

      
      // 6) 做保護：數字只能在 0% ~ 100% 之間，不要超出容器
      const leftPct  = Math.max(0, Math.min(100, leftOffsetPercent));
      const rightPct = Math.max(0, Math.min(100, rightOffsetPercent));

      // 7) 把結果塞進 CSS 變數，讓樣式自己更新位置
      container.style.setProperty('--offset-left',  `${leftPct}%`);
      container.style.setProperty('--offset-right', `${rightPct}%`);
    }

    // === 建立 Swiper（輪播） ===
    // 這裡只在「需要重新排版」的時候更新按鈕位置：
    // 1. 第一次準備好（init）
    // 2. 容器大小改變（resize / update）
    // 不在滑動途中更新，按鈕就不會在動畫中亂跑
    const swiper = new Swiper('.mySwiper', {
      loop: true,
      slidesPerView: 'auto',
      centeredSlides: true,
      navigation: { nextEl: '.swiper-button-next', prevEl: '.swiper-button-prev' },

      // 這兩個設定：當父層大小改變時，Swiper 會自己重新計算
      observer: true,
      observeParents: true,

      on: {
        // 初始化完 → 再多等兩幀，確定畫面真的置中後，才量位置
        init() { afterLayout(updateNavOffset); },
        // 大小有變（例如視窗縮放、父層變窄）→ 再等兩幀後重算
        resize() { afterLayout(updateNavOffset); },
        update() { afterLayout(updateNavOffset); }
      }
    });

    // === 這裡處理「視窗大小改變」的情況 ===
    // 例：把瀏覽器拖到另一個螢幕、打開 DevTools 讓頁面變窄
    // 我們等兩幀再量，確保尺寸真的固定下來
    let resizeRaf = 0;
    window.addEventListener('resize', () => {
      if (resizeRaf) cancelAnimationFrame(resizeRaf);
      resizeRaf = afterLayout(updateNavOffset);
    });

    // === 這裡再多一層保險：容器自己變寬或變窄也要重算 ===
    // ResizeObserver 會在「容器的內容區大小」改變時叫我們
    // 例如：父層排版變了、打開 DevTools 擠壓了容器
    const containerEl = document.querySelector('.mySwiper');
    if (window.ResizeObserver && containerEl) {
      const ro = new ResizeObserver(() => {
        afterLayout(updateNavOffset);
      });
      ro.observe(containerEl);
    }
```
<br>

---
本範例使用ai協作(歡迎自由取用)<br>
如有錯誤歡迎修正，感謝您
