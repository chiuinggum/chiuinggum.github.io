---
layout: post
title: "淺談計算複雜性理論"
date: 2024-06-15
categories: algorithm
---

- [1. 前言](#1-前言)
  - [1-1. 難度的概念](#1-1-難度的概念)
  - [1-2. 回顧 - 多項式時間 (polynomial time)](#1-2-回顧---多項式時間-polynomial-time)
  - [1-3. 準備 - 歸約 (reduction)](#1-3-準備---歸約-reduction)
  - [1-4. 準備 - 決定性問題 vs. 最佳化問題](#1-4-準備---決定性問題-vs-最佳化問題)
  - [1-5. 準備 - 確定性算法 vs. 非確定性算法](#1-5-準備---確定性算法-vs-非確定性算法)
- [2. P 問題與 NP 問題](#2-p-問題與-np-問題)
  - [2-1. P 問題](#2-1-p-問題)
  - [2-2. NP 問題](#2-2-np-問題)
    - [2-2-1. NP 問題的例子](#2-2-1-np-問題的例子)
  - [2-3. P 是 NP 的子集](#2-3-p-是-np-的子集)
  - [2-4. 運用歸約證明 P 和 NP 間的關係](#2-4-運用歸約證明-p-和-np-間的關係)
- [3. NP-Hard 問題](#3-np-hard-問題)
  - [3-1. 非 NP 的 NP-Hard 問題 - 以停機問題為例](#3-1-非-np-的-np-hard-問題---以停機問題為例)
- [4. NP-Complete 問題](#4-np-complete-問題)
  - [4-1. NP-Complete 問題中的重要例子 - SAT Problem](#4-1-np-complete-問題中的重要例子---sat-problem)
- [5. P = NP 猜想](#5-p-np-猜想)

## **1. 前言**

我們在大學課程中接觸到的演算法，大多屬於多項式時間（polynomial time）可解的類型，意即給定大小為 $$n$$ 的輸入，此算法的最壞情況的時間複雜度 (worst-case time complexity) 為 $$O(n^k)$$。然而，有很多的演算法問題，是目前找不到方法可以在多項式時間內解[^1]。很明顯地，「可以用多項式時間解的問題」和「不確定是否可以用多項式時間解的問題」，這兩種問題的「難度」應該不同。在「不確定是否可以用多項式時間解的問題」這一類中，又應該進一步細分以幫助我們更好地分類問題。因此學者在研究這些問題的過程中，希望能用一種方式將這些問題依照「難度」分類，以方便進行討論。

這些難度分類包括 「P 問題 (P problems)」以及「NP 問題 (NP problems)」。你或許還聽過「NP 完備問題 (NP-Complete problems)」或是「NP 困難問題 (NP-Hard problems )」等。不過，即便全部沒聽過也沒關係，我們都將在本文當中討論到這些概念。

本文想討論的是這些**難度分類的標準為何**，以及我們應該**如何判斷一個問題是隸屬於哪個分類**。在正式開始討論這些難度分類之前，我們會先釐清我所說的難度為何、回顧需要用到的重要概念，以及準備好學習這些分類所需的工具。

### **1-1. 難度的概念**

一個問題的「難度」在有明確定義之前是抽象的。有人可能會說問題越大，則問題越難，但其實並非如此。例如，給定一個有序數列，尋找其中某個數字的問題，即便數列很大，仍可以用二分搜尋法在對數時間內解決，因此這類問題的難度並不高。「問題的難度」仍然應該回到時間複雜度來衡量，像是以「一個問題可不可以用多項式時間解」當作判斷標準，來決定一個問題的難度分類。

需要小心的是，這個難度分類是類似一個級距，例如 80 - 89 分的人有 13 個，這並不代表這 13 個人的分數完全一樣。即使兩個問題都被歸類為 NP，其實際難度可能不同，但在複雜度分類中，它們都屬於 NP 級別。（現在不知道什麼是 NP 沒關係，只要知道他是一個分類就好）

### **1-2. 回顧 - 多項式時間 (polynomial time)**

如果不熟悉時間複雜度 (time complexity) 或是大 $$O$$ 符號 (Big-O notation)，建議可以先去了解一下這兩個概念，再回來看這篇文會比較好理解。如果你已經知道它們是什麼，透過[下圖](https://www.bigocheatsheet.com/)喚醒一下記憶，知道哪些時間複雜度是多項式時間就夠了：

![Big-O Complexity Chart](/assets/images/2024-06-15-introduction-to-complexity/Big-O-Complexity-Chart.png)

**為什麼要熟悉多項式時間？**\\
在多項式時間內可以被解決的事都是簡單的，這對於後續判定難度的方式很重要。我們也說，多項式時間的算法是有效率的。

{:.highlight-blockquote}

> - <span class="highlight">$$O(n^k)$$, $$O(n\log n)$$, $$O(n)$$, $$O(\log n)$$ 和 $$O(1)$$ 都是有效率的。</span>

### **1-3. 準備 - 歸約 (reduction)**

我們說「問題 A 可歸約成問題 B (problem A is reducible to problem B)」，意即存在一個多項式時間算法可以將一組「A 問題輸入」轉換成一組等價的[^2]「B 問題輸入」。

當我們「將問題 A 歸約成問題 B (problem A reduces to problem B; reduction from problem A to problem B)」時，可以將其關係表示為：$$A \le_p B$$（$$p$$ 代表 polynomial）

這個關係有幾個特性：

- 問題 B 的難度至少和問題 A 的難度一樣。如果知道怎麼解決問題 B，就一定知道怎麼解決問題 A。
- 遞移性 (transitivity)：已知問題 A 可以歸約成問題 B，如果問題 B 可以歸約成問題 C，則問題 A 也可以歸約成問題 C。

這樣講可能還是有點抽象，我們來看一些例子：

1. 路徑問題 $$\le$$ 連通問題

   - 路徑問題：給定一個圖 G 和兩個點 s 和 t，判斷是否存在從 s 到 t 的路徑
   - 連通問題：給定一個圖 G，判斷 G 是否連通，意即是否存在從任一點到任一其他點的路徑
   - 歸約過程：若能解決「連通問題」，我們可以透過檢查 G 的聯通性來解決「路徑問題」。若我們只考慮 s 和 t 之間是否存在一條路徑來判斷圖是否連通，這樣的方式可以轉化成連通性判定。或者，我們可以透過在圖中加入額外邊來使問題轉化。如果新圖是連通的，則從 s 到 t 存在一條路徑。

1. 子集合問題 $$\le$$ 背包問題

   - 子集合問題：子集合問題是判斷是否能挑出一組數字，讓它們加總起來等於某個特定值
   - 背包問題：給定一組物品，每個物品有重量和價值，找到在重量限制下最有價值的組合
   - 歸約過程：我們可以把每個數字當作一個「物品」，這些物品的重量和價值都設為這個數字本身。接著設定背包的最大重量為這個特定值，並要求「裝滿這個背包、且總價值正好等於物品總重量」，這就等價於原本的子集合問題。

_為什麼要了解什麼是歸約？_\\
當我們在分類問題難度的時候，經常會使用到「歸約」這個技巧。歸約講白話一點就是「轉換」。由於我們很難從零證明某個問題隸屬於某個難度，因此我們常會拿一些已經被證明難度為何的問題，來幫我們找出一個新問題的難度。\\
也就是說，當我們面對一個不熟悉的問題（不知道該如何分類的問題）時，可以把這個問題轉換成一個比較熟悉的問題（已知分類為何的問題）。透過這樣的技巧，就可以輕鬆地證明出某個問題的難度。

{:.highlight-blockquote}

> - <span class="highlight">若 A 可以歸約成 B，那麼 A 不會比 B 難，則 A 的難度 $$\le$$ B 的難度</span>
> - <span class="highlight">歸約存在遞移性</span>

### **1-4. 準備 - 決定性問題 vs. 最佳化問題**

我們可以將問題分類成「決定性問題 (decision problem)」以及「最佳化問題 (optimization problem)」。決定性問題的答案只有「是 (yes)」 或是「否 (no)」，而最佳化問題的答案是最佳化的結果。

所有的最佳化問題都可以被轉換成決定性問題。直接看例子：

1. 最短路徑問題 (shortest problem)

   - 最佳化問題：找出圖中的最短路徑
   - 決定性問題：是否存在一條路徑總長小於 500[^3]？

1. 旅行推銷員問題 (traveling salesperson problem)

   - 最佳化問題：找出圖中可以拜訪每個城市一次的最短路徑
   - 決定性問題：是否存在可以拜訪每個城市一次的路徑，其總長小於 500？

仔細看的話，可以發現每個最佳化問題，都存在相對應的決定性問題，使得該決定性問題可以歸約成最佳化問題。

_為什麼要了解什麼是決定性問題以及什麼是最佳化問題？_\\
在討論 P 問題和 NP 問題時，我們都只討論決定性問題，原因是決定性問題較容易分析和比較[^4]。雖然很多問題最初是最佳化問題，但由於最佳化問題可以轉換成決定性問題，能夠解出決定性問題，通常也代表可以解出相關更複雜的最佳化問題。而決定性問題的關鍵在於幫助我們了解一個問題的難度。

{:.highlight-blockquote}

> - <span class="highlight">討論 P 問題和 NP 問題時，都只討論決定性問題。</span>
> - <span class="highlight">最佳化問題可以轉換成決定性問題。</span>

### **1-5. 準備 - 確定性算法 vs. 非確定性算法**

#### **1-5-1. 確定性算法 (deterministic algorithm)**

我們平常寫的演算法，像是解 Leetcode、考試題目的程式，大多都是「確定性算法」。這類演算法的特性是：**每次給它一樣的輸入，總是會產生相同的輸出**，中間的執行流程也是固定的。

![deterministic-algorithm-0](/assets/images/2024-06-15-introduction-to-complexity/deterministic-algorithm-0.png){: width="250" }
_黃色執行完後，如果 x = 0 ，則執行藍色；如果 x ≠ 0，則執行粉色_

#### **1-5-2. 非確定性算法 (non-deterministic algorithm)**

非確定性算法是一種**理論模型**，真實世界的電腦並無法真正執行它。但在計算理論中，它是個非常強大的工具，用來定義問題的複雜度。

這類算法的特性是：在某些步驟中可以**同時分岔出多條執行路徑**，每條路徑代表一種可能的選擇，最終是否接受輸入，取決於是否存在**至少一條成功的路徑**。因此，**對同樣的輸入，可能產生不同的執行流程與輸出。**

![non-deterministic-algorithm-0](/assets/images/2024-06-15-introduction-to-complexity/non-deterministic-algorithm-0.png){: width="250" }
_黃色執行完後，不曉得會執行藍色，或執行粉色。_

我們可以把它想成兩個階段：

1. **猜測 (guessing)**：產生一個可能的解
2. **驗證 (verification)**：檢查這個解是否正確

在討論複雜度時，我們假設這個算法能夠「總是猜對答案」，也就是說，它可以神奇地直接選中正確的解。**我們不在乎這個解是怎麼被猜出來的**，這點聽起來可能有些違反直覺，畢竟我們平常總是被訓練要釐清「怎麼解題」。但這正是非確定性模型的魅力：**它只關心能不能驗證答案，不關心怎麼找到它**。

你可以先把它想成一個「神奇海螺」--**不管你問它什麼問題，它總能立刻告訴你正確答案**。

_為什麼要了解什麼是確定性算法、什麼是非確定性算法？_\\
因為在後續定義問題難度（如 P、NP、NP-Complete）時，這兩種算法模型會扮演核心角色。

{:.highlight-blockquote}

> - <span class="highlight">確定性算法即一般的演算法。</span>
> - <span class="highlight">非確定性算法包含猜答案和驗證，且假設總是會猜對答案。</span>

接下來我們會看到，這些概念如何成為定義「P vs NP」問題的基礎。

---

## **2. P 問題與 NP 問題**

### **2-1. P 問題**

首先，我們先討論最好理解的 P 問題 (P problems)，其全名為 “Polynomial Time Problems”。我們先直接看完整定義：

> P 問題為那些可以花**多項式時間**用**確定性**算法**解出來**的決定性問題。\\
> P problems are **decision problems** that can be **solved in polynomial time** by a **deterministic** algorithm.

記得，確定性算法就是一般的演算法，而決定性問題就是一個是非題而已。這些概念都沒什麼了不起，但這個又臭又長的定義，第一次看到可能還是會有點頭昏眼花，所以可以先把它想成：

> P 問題為那些可以**花多項式時間解出來**的問題。\\
> P problems are problems that can be **solved in polynomial time**.

總之，**P 問題就是可以快速被解出來的問題**，P 問題就是我們已經找到有效率的算法能解出來的問題。

{:.highlight-blockquote}

> - <span class="highlight">P 問題是容易解的問題。</span>
> - <span class="highlight">P 問題可以在多項式時間內被解出來。</span>

### **2-2. NP 問題**

接著，我們來看讓人有點頭疼的 NP 問題，其全名為 “Non-Deterministic Polynomial Time Problems”。注意，NP 指的並非 Non-Polynomial，要小心一開始很容易搞錯。一樣直接看完整定義：

> NP 問題為那些可以花**多項式時間**用**非確定性**算法解出來的**決定性問題**。\\
> NP problems are **decision problems** that can be **solved in polynomial** time by a **non-deterministic** algorithm.

可以用非確定性算法解的意思是：我們可以從神奇海螺那邊獲得一個答案後，驗證它是正確的答案。因此，這個定義的意思即為，在我們從神奇海螺那邊獲得答案之後，可以花多項式時間驗證這個答案。如同前面提到的，如何獲得這個答案並不重要，所以其實也可以簡單想成：

> NP 問題為那些可以花**多項式時間**用確定性算法**驗證答案**的決定性問題。\\
> NP problems are **decision problems** whose solution can be **verified in polynomial time**.

也就是：

> NP 問題為那些可以花**多項式時間驗證答案**的問題。\\
> NP problems are problems whose solution can be **verified in polynomial time**.

總之，NP 問題就是可以**快速被驗證答案正不正確的問題**。

#### **2-2-1. NP 問題的例子**

給定 n 個物品和預算 c，每個物品有不同的標價。給定一個購買方式使得，購買的所有物品總價剛好為 c。

給定一解（一組物品），我們可以很快速的將這組物品的總價算出，驗證是否等於 c。但由於我們必須試每個可能的購買組合，因此無法快速解這個問題。

{:.highlight-blockquote}

> - <span class="highlight">NP 問題是容易驗證答案的問題。</span>
> - <span class="highlight">NP 問題的解可以在多項式時間內被驗證。</span>

### **2-3. P 是 NP 的子集**

了解 P 問題和 NP 問題的定義後，我們很快地得出 P 問題也是 NP 問題的結論。原因是，可以用多項式時間找到解，意味著這個解在多項式時間被驗證。

當已知一邊的包含性 (P ⊆ NP)，我們好奇是不是可以證明另一邊的包含性 (NP ⊆ P)，以證明 P = NP。在嘗試證明 NP ⊆ P 之前，必須先了解什麼是 NP-Hard 問題和 NP-Complete 問題。

### **2-4. 運用歸約證明 P 和 NP 間的關係**

在前面「歸約」的小節中提到，如果問題 A 可以歸約成問題 B，則問題 B 的難度至少和問題 A 的難度一樣。在理解 P 問題和 NP 問題的定義後，明顯可以推得：

> 若 B 是一個 P 問題，則 A 也是一個 P 問題 $${\Leftrightarrow}$$
>
> 若 A 不是一個 P 問題，則 B 也不是一個 P 問題\\
> If B $$\in$$ P, then A $$\in$$ P $${\Leftrightarrow}$$ If A $$\notin$$ P, then B $$\notin$$ P

> 若 B 是一個 NP 問題，則 A 也是一個 NP 問題 $${\Leftrightarrow}$$
>
> 若 A 不是一個 NP 問題，則 B 也不是一個 NP 問題\\
> If B $$\in$$ NP, then A $$\in$$ NP $${\Leftrightarrow}$$ If A $$\notin$$ NP, then B $$\notin$$ NP

這個概念我們等等會用到。

---

## **3. NP-Hard 問題**

直覺上看到 NP-Hard 會以為它代表 NP 中困難的問題，但其實不然。我們先看定義：

> 若一個問題 Q 是 NP-Hard 問題，則**每個 NP 問題都可以歸約成 Q**。\\
> If problem Q is a NP-Hard problem, then **every NP problem can reduce to Q**.

直覺上出錯的地方在於，NP-Hard 問題不見得是 NP 問題。它之所以叫做 NP-Hard 不是因為它是 NP 問題，而是因為所有 NP 問題都可以歸約成它。

![np-and-np-hard](/assets/images/2024-06-15-introduction-to-complexity/np-and-np-hard.png){: width="400" }
_Q 為 NP-Hard 問題，X1 ~ X6 為 NP 問題，因此 X1 ~ X6 皆可歸約成 Q。_

而 ”Hard” 確實暗指了它是困難的。根據歸約的特性，NP-Hard 問題至少和所有的 NP 問題一樣難。因此，NP 與 NP-Hard 的難度關係可以大概想成[^5]：

![np-and-np-hard-difficulty](/assets/images/2024-06-15-introduction-to-complexity/np-and-np-hard-difficulty.png){: width="400" }

### **3-1. 非 NP 的 NP-Hard 問題 - 以停機問題為例**

有什麼問題是 NP-Hard 但不是 NP 嗎？這樣的問題是存在的，著名的「停機問題 (halting problem)」就是非 NP 的 NP-Hard 問題。事實上，所有的「不可解問題 (undecidable problem) 」都是非 NP 的 NP-Hard 問題。由於篇幅關係，本文僅討論該如何證明停機問題非 NP，對於如何證明停機問題為 NP-Hard 目前不贅述。

#### **3-1-1. 什麼是不可解問題？和 NP 又有什麼關係？**

一個問題為不可解問題的意思是，不存在一個算法能夠解決問題。

而根據 NP 的定義，NP 問題為那些可以花多項式時間用非確定性算法解出來的決定性問題，也就指出 NP 問題是可以被特定算法「解決」的。由於不可解問題無法用算法解決，因此**不可解問題不是 NP 問題**。

#### **3-1-2. 停機問題**

停機問題為一種不可解問題 (undecidable problem)。這個問題的敘述是：

> 給定一個程式，判定這個程式是否會停止 (terminate)。

如果我們宣稱停機問題是不可解的，代表我們找不到一種方法，能夠判斷任何一個程式是否會停止。但這是真的嗎，難道我們永遠找不到這個方法嗎？要解答這個疑惑，我們可以先試著假設我們有這個方法，然後觀察會有什麼事情發生。如果一定會產生矛盾，就代表這個假設是錯的，所以我們就可以篤定停機問題真的是不可解。

假設我們有道具 A，可以判斷任何一個程式是否會停止。這時候聰明的你就會想到，那我們不是可以寫一個程式叫做 Reverser，根據這個道具的輸出，決定要怎麼執行這個程式嗎？意思是，假設這個道具說程式 Reverser 會停止，那我們就讓 Reverser 進入一個無窮迴圈；反之，如果這個道具說程式 Reverser 不會停止，那我們就讓 Reverser 立刻停止。基本上就是和這個道具唱反調。

![reverser](/assets/images/2024-06-15-introduction-to-complexity/reverser.png){: width="400" }
_左圖為演算法 A；右圖為程式 Reverser_

這時候我們就會發現，由於 A 對 Reverser 是否終止的判斷永遠會錯，因此我們得到矛盾。證明不存在一個演算法可以判定所有的程式是否終止。

{:.highlight-blockquote}

> - <span class="highlight">全部的 NP 問題都可以歸約成 NP-Hard 問題。</span>
> - <span class="highlight">NP-Hard 問題不一定是 NP。</span>

---

## **4. NP-Complete 問題**

當我們知道什麼是 NP 問題以及什麼是 NP-Hard 問題，就可以討論 NP-Complete 問題了：

> NP-Complete 問題為那些同時**是 NP 也是 NP-Hard 的問題**。\\
> NP-Complete problems are problems that are **NP and NP-Hard** at the same time.

![np-npc-nph](/assets/images/2024-06-15-introduction-to-complexity/np-npc-nph.png){: width="400" }

這句話代表的意義又是什麼呢？根據定義，NP-Complete 問題是 NP-Hard 問題，代表它至少和 NP 問題一樣難。又因為它同時也是 NP 問題，代表它「剛剛好」和所有的 NP 問題一樣難，不會更難也不會更簡單。我們也可以說 NP-Complete 問題是 NP 問題中最難的問題。

![np-npc-nph-difficulty](/assets/images/2024-06-15-introduction-to-complexity/np-npc-nph-difficulty.png){: width="400" }

根據 NP-Complete 的定義，證明一個問題 Q 是 NP-Complete 的方式很直觀，就是證明：

1. Q 是 [NP 問題](#2-2-np-問題)
1. Q 是 [NP-Hard 問題](#3-np-hard-問題)（也就是證明所有 NP 問題都可以歸約成 Q）

直覺上，證明「所有」是非常困難的。這時候，聰明的你就會想到可以用前面提到的[歸約](#1-3-準備---歸約-reduction)技巧來幫助我們證明 Q 是 NP-Hard：

> 欲證明 Q 是 NP-Hard，可證明存在已知的 NP-Complete 問題 Q'，使得 Q' 可以歸約成 Q

也就是說，我們有以下等價關係：

> Q 是 NP-Hard $$\Leftrightarrow$$
>
> 存在已知的一個 NP-Complete 問題 Q'，使得 Q' 可以歸約成 Q\\
> Q is NP-Hard $$\Leftrightarrow$$ $$\exist$$ a NP-Complete problem Q' such that Q' $$\le$$ Q

為什麼證明 Q' 存在就可以證明 Q 是 NP-Complete 呢？因為全部的 NP 問題皆可已歸約成 Q'，而 Q' 又可以歸約成 Q。根據歸約的遞移性，全部的 NP 問題皆可以歸約成 Q，也就是說，Q 是 NP-Hard。

因此，欲證明 Q 是 NP-Complete，我們可以證明：

1. Q 是 NP 問題
1. 存在已知的 NP-Complete 問題 Q'，使得 Q' 可以歸約成 Q

### **4-1. NP-Complete 問題中的重要例子 - SAT Problem**

NP-Complete 中有一個著名的問題，叫做**布林可滿足性問題** (Boolean satisfiability problem; SAT problem)。由於這個問題已知是 NP-Complete，它就像是上面的問題 Q'，常常被拿來當作證明其它問題是 NP-Hard 的工具。

SAT problem 是判斷一個布林方程式 (Boolean function) 是否可滿足 (satisfiable)[^6]，換句話說，是否存在一組變數賦值 (assignment)[^7]滿足給定的布林方程式。直接看例子會比較清楚：

- 例一
  - 布林方程式：$$F = (x_1 \lor x_2) \land (\neg x_3)$$
  - 解（賦值）：$$x_1 = \text{True}, x_2 = \text{False}, x_3 = \text{True}$$
  - 由於將解代入 $$F$$ 後，$$F$$ 的值為 $$\text{True}$$，因此我們說 $$F$$ 是可滿足的，且 $$(x_1, x_2, \neg x_3)$$ 為滿足 $$F$$ 的賦值。
- 例二
  - 布林方程式：$$F = (x_1) \land (\neg x_1)$$
  - 解（賦值）：不存在一組賦值使得 $$F$$ 為 $$\text{True}$$
  - 由於不存在一組賦值使得 $$F$$ 為 $$\text{True}$$，因此我們說 $$F$$ 是不可滿足的 (unsatisfiable)。

我們有個重要定理叫做 "Cook-Levin Theorem"，就是在證明 SAT 是 NP-Complete。這個定理的證明相當複雜，我們本篇不討論，但它的結果延伸出其他重要的結論，我們將於下一節討論。

{:.highlight-blockquote}

> - <span class="highlight">NP-Complete 問題是 NP 問題中最難的問題。</span>
> - <span class="highlight">SAT problem 是 NP-Complete 問題，可以被用來證明其他問題是 NP-Complete。</span>

---

## **5. P = NP 猜想**

了解 NP-Hard 和 NP-Complete 的定義後，就可以回來看剛剛好奇的問題，P 到底等不等於 NP 呢，也就是我們可以證明 NP $$\subseteq$$ P 嗎？

答案是，**目前沒有人可以證明 P = NP 或是 P ≠ NP**。

這個問題是計算理論中最著名、最困難的未解問題之一，甚至被列為 Clay 數學研究所的「七大千禧難題」之一。

雖然問題很困難，我們還是可以從理論角度來理解它的關鍵。根據 Cook-Levin 定理，有以下等價關係：

> P = NP $$\Leftrightarrow$$ SAT 是 P 問題

意思是我們只要證明 SAT problem ∈ P（即 SAT 是多項式時間可解的），就可以證明 P = NP。我們就直接開始吧：

- 證明：P = NP $$\Rightarrow$$ SAT 是 P 問題
  - 由於 SAT 是 NP-Complete 問題，它屬於 NP；又因假設 P = NP，則 SAT ∈ P。
- 證明：SAT 是 P 問題 $$\Rightarrow$$ P = NP
  - 由於 SAT 是 NP-Complete 問題，代表所有 NP 問題都能在多項式時間內歸約到 SAT；又假設 SAT ∈ P，代表我們可以在多項式時間內解 SAT。
  - 因此，所有 NP 問題都可以透過「多項式歸約 + 多項式解法」來解，也就是說每個 NP 問題都能在多項式時間內解決。
  - 所以 NP ⊆ P，亦即 P = NP。

從證明中，可以發現其實 SAT problem 可以替換成任一 NP-Complete 問題，也不影響證明。這也代表：

> P = NP $$\Leftrightarrow$$ 任一 NP-Complete 問題為 P 問題

所以只要能找到一個 NP-Complete 問題的多項式時間演算法，我們就能解開這個世界級的謎題。

但為什麼學者們那麼希望可以證明 P = NP 呢？因為如果成功證明了 P = NP，這代表每個 NP 問題都存在一個有效率的解法，只是我們還沒找到而已。反之，如果可以成功證明 P ≠ NP，也是一個很大的進展，這代表有些 NP 問題，一定找不到有效率的解法。

不過，因為證明 P = NP 被視為一個幾乎不可能的任務，所以現今對於 P 和 NP 間的關係，普遍認知是：

![p-np-npc-nph-relation](/assets/images/2024-06-15-introduction-to-complexity/p-np-npc-nph-relation.png){: width="400" }

從上圖還可以得到一個有趣的推論是：

> 若問題 Q 是 NP-Hard，則問題 Q 不是 P（除非 P = NP）

另外也可以參考[下圖](https://en.wikipedia.org/wiki/NP-hardness)。除了分別畫出 P = NP 和 P ≠ NP 的狀況之外，也清楚的標記各分類的難度關係。

![p-np-npc-nph-relation-detailed](/assets/images/2024-06-15-introduction-to-complexity/p-np-npc-nph-relation-detailed.png){: width="400" }

{:.highlight-blockquote}

> - <span class="highlight">目前普遍認為 P ≠ NP。</span>
> - <span class="highlight">P = NP 若且唯若任一 NP-Complete 問題為 P 問題。</span>

<hr data-content=" 參考資料 ">

- [李耕銘 - 【為什麼要區分演算法的 NP 問題】](https://lkm543.medium.com/%E6%BC%94%E7%AE%97%E6%B3%95%E4%B8%AD%E7%9A%84-np-%E5%95%8F%E9%A1%8C-db976e55b60b)
- [Khan Academy - Undecidable problems](https://www.khanacademy.org/computing/ap-computer-science-principles/algorithms-101/solving-hard-problems/a/undecidable-problems)
- [MIT OpenCourseWare - 16. Complexity: P, NP, NP-Completeness, Reductions](https://www.youtube.com/watch?v=eHZifpgyH_4&list=PLUl4u3cNGP6317WaSNfmCvGym2ucw3oGp&index=23)
- [StackExchange - What is the definition of P, NP, NP-Complete and NP-Hard?](https://cs.stackexchange.com/questions/9556/what-is-the-definition-of-p-np-np-complete-and-np-hard)

<hr data-content=" 註解 ">

[^1]: 現在無法用多項式時間解的某個問題，未來有可能找到某個多項式時間算法解出。
[^2]: 在這裡等價的意思是，問題 A 的輸入經過多項式時間的轉換後，得到的問題 B 的輸入在問題 B 中得到的結果與原來在問題 A 中得到的結果是一樣的。換句話說，問題 A 的「是」或「否」答案對應於問題 B 的「是」或「否」答案。這種轉換允許我們利用問題 B 的解決方法來解決問題 A。
[^3]: 這裡的 500 是個常數，可以被任何其他常數取代。
[^4]: 因為決定性問題的答案只有是或否。
[^5]: 用線性的畫法來描述難度其實不夠精確，但目前可以先以這樣的概念理解。在文末有更精確的難度標示圖。
[^6]: 可滿足意即布林方程式結果為真；不可滿足意即布林方程式結果為假。
[^7]: 變數賦值的意思是給變數值，如：$$x = \text{True}$$ 代表將變數 $$x$$ 之值設為 $$\text{True}$$、$$y = \text{False}$$ 代表將變數 $$y$$ 之值設為 $$\text{False}$$。
