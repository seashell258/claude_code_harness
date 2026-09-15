## 全部對程式的要求都塞在 system prompt，context 空間就沒了，所以只應在需要的時候載入特定要求

那些被拆出去的 prompt 就是 skill。 一個 skill 會有 name、description、content

新 tool : 
skillLoader.catalog 
skillLoader.load 

.catalog() 會回傳所有 skill 的名字跟描述，塞進 prompt 裡面，讓 AI 知道自己有哪些 skill 可以調用、靠描述判斷調用時機。

在 AI 覺得該調用 skill 的時候，它就會 type:tool_use, name: skillLoader.load 
來 load 整個 skill 的完整內文。

## claude code cli 已經是寫好的二進位制 harness，所以沒有自己定義 catalog 、 load 的彈性。 但還是可以把 prompt 放進 skills 資料夾，然後格式包含 name、description，有一樣的基本效果

模型會靠著 description 來自己判斷何時要調用 skill。

然後 skill 也可以有自己的 skill，也就是再把 prompt 拆分出去。
模型只有在看 skill.md ，判斷自己需要更多資訊時，才會去讀 skill 的 skill。
更進一步讓 context 乾淨。