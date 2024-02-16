---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Shortest Path

### Floyd-Warshall

\
Floyd-Warshall 演算法是一種計算圖中所有頂點對之間最短路徑的動態規劃算法。它能處理含有正或負權重邊的有向圖，但不適用於有負權重循環的圖。這個算法透過三重循環，逐步更新圖中每對頂點間的最短路徑距離。核心思想是考慮是否通過中繼頂點（第三層循環的頂點）來更新任意兩點間的最短路徑。時間複雜度是 $$O(V^3)$$，其中 $$V$$ 是圖中的頂點數。

### Dijkstra

請見 Priority Queue 章節。

### Bellman-Ford

這個演算法能夠處理帶有負權重邊的圖，這是它與Dijkstra演算法的主要區別。但是它不能處理包含負權重循環的圖，因為這會導致無限減少的路徑長度。

Bellman-Ford演算法的原理是基於這個事實：在一個沒有負權重循環的圖中，從源點到任何其他頂點的最短路徑最多包含 $$V−1$$ 條邊，其中 $$V$$ 是圖中的頂點數量。因此，演算法反覆地放鬆（relax）圖中的所有邊，放鬆操作是指如果通過某條邊可以獲得更短的路徑，則更新目標頂點的距離估計值。這個過程重複 $$V−1$$ 次。

演算法還包括一個檢測負權重循環的步驟。在執行了 $$V−1$$ 次放鬆後，如果再次放鬆任何邊會導致距離變小，則表示圖中存在負權重循環。

時間複雜度上，Bellman-Ford演算法是 $$O(VE)$$，其中 $$E$$ 是圖中的邊數。
