## 有些邏輯太長，寫在 loop 裡面會稀釋業務邏輯，所以抽出來寫成 hook
AI 工作中的特定時機，執行一個自訂的函數就是 hook
和 git hook 可以在你 commit 或 push 之前執行一些你自訂的邏輯是一樣的。

時機有
before tool use 、 ai 工作完一輪 等等