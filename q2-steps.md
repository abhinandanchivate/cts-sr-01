
---

## 🔍 Step 1: Understand what “tower of blocks” means

* For a tower of **L levels**, number of blocks needed is:

  [
  1^2 + 2^2 + 3^2 + \dots + L^2
  ]

* Instead of adding squares one by one, we use the formula:

  [
  \text{blocks}(L) = \frac{L \times (L + 1) \times (2L + 1)}{6}
  ]

This gives us **how many blocks are required to build a tower of height L**.

---

## 🧠 Step 2: What does the problem want?

You are given `N` blocks.

You must:

1. **Always build the tallest possible tower** you can with the current remaining blocks.
2. Subtract the blocks used for that tower from `N`.
3. Repeat the same process with the new remaining N.
4. Stop when you don't have enough blocks even for a **1-level** tower.

Finally, you must return **how many towers** you were able to build.

---

## 🧱 Step 3: Overall Strategy (Greedy Approach)

We use a **greedy approach**:

> At every step, use as many levels as possible (maximum L) such that `blocks(L) <= remainingBlocks`.

So the high-level logic is:

1. Read input `N` (total blocks).
2. Initialize `countTowers = 0`.
3. While `N >= 1`:

   * Find the **maximum L** such that `blocks(L) <= N`.
   * If no such L exists (i.e., even `blocks(1) > N`), break the loop.
   * Subtract `blocks(L)` from `N`.
   * Increment `countTowers` by 1.
4. Print `countTowers`.

---

## 🧮 Step 4: How to find the tallest possible L each time?

For the current `N`:

1. Start with `L = 1`.
2. Keep increasing L while `blocks(L) <= N`.
3. When `blocks(L)` becomes larger than `N`, the tallest valid tower height is `L - 1`.

In code you might:

* Either loop from `L = 1` upwards each time,
* Or precompute all `blocks(L)` values up to some max `L`, then use them.

But in **words**, it's simply:

> Keep increasing the height L as long as you have enough blocks to build that full tower.

---

## 🧪 Step 5: Walk through the sample (N = 40)

### First tower:

* Check L = 1 → blocks(1) = 1
* L = 2 → 1 + 4 = 5
* L = 3 → 1 + 4 + 9 = 14
* L = 4 → 1 + 4 + 9 + 16 = 30
* L = 5 → 1 + 4 + 9 + 16 + 25 = 55 (> 40, so not possible)

So tallest possible tower = **4 levels, needs 30 blocks**.

* Remaining blocks = 40 − 30 = **10**
* Towers built so far = 1

---

### Second tower (N = 10 now):

* L = 1 → 1
* L = 2 → 5
* L = 3 → 14 (> 10, stop)

Tallest possible tower now = **2 levels, needs 5 blocks**.

* Remaining blocks = 10 − 5 = **5**
* Towers built so far = 2

---

### Third tower (N = 5 now):

* L = 1 → 1
* L = 2 → 5
* L = 3 → 14 (> 5, stop)

Tallest possible tower now = **2 levels, needs 5 blocks**.

* Remaining blocks = 5 − 5 = **0**
* Towers built so far = 3

⛔ Now `N = 0`, so cannot build more towers.

But **note**: in the original statement I gave you earlier, I stopped after two towers (4-level and 2-level) and didn’t build the third one — that was a mistake in the sample logic. With the rules as stated (“repeat until you can’t even build a 1-level tower”), **40 actually allows 3 towers** (4-level, 2-level, 2-level).

If you want to strictly match the earlier sample output `2`, you would **change the rule** slightly (for example, “don’t reuse the same height again”) — but with the written rules, the mathematically consistent answer is 3.

So for your training/assignments, decide which rule you want:

* **Allow repeated tower heights → 3 towers for 40**
* **Disallow repeated heights → 2 towers for 40**

---

