---
title: ADS 错题本
sync: /course/ads/correction.md
---

## 算法分析

### 群友求助


## 1. 平衡树

### 1.1. HW1 2-5

Consider the following buffer management problem. Initially the buffer size (the number of blocks) is one. Each block can accommodate exactly one item. As soon as a new item arrives, check if there is an available block. If yes, put the item into the block, induced a cost of one. Otherwise, the buffer size is doubled, and then the item is able to put into. Moreover, the old items have to  be moved into the new buffer so it costs $k+1$ to make this insertion, where $k$ is the number of old items. Clearly, if there are $N$ items, the worst-case cost for one insertion can be $\Omega (N)$.  To show that the average cost is $O(1)$, let us turn to the amortized analysis. To simplify the problem, assume that the buffer is full after all the $N$ items are placed. Which of the following potential functions works?

- A. The number of items currently in the buffer
- B. The opposite number of items currently in the buffer
- C. The number of available blocks currently in the buffer
- D. The opposite number of available blocks in the buffer

