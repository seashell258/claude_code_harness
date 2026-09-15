## 上下文太長導致表現不好，要對歷史進行壓縮

文章提出四個壓縮手段一起採用。自訂harness的時候參考這邊的做法來實現壓縮
主要原理是把對話存檔起來放本地文件，這樣對話歷史就不用存

每次對話之後  
1. 太長的 tool result 存去本地文件，只留下前幾個字預覽(非摘要)  
2. 只保留前3訊息(不忘記初衷)、最近的46訊息。  中間的對話存去本地文件

對話歷史達到100%之後
3. 不管長度，把老舊的 tool result 存去本地文件，只留摘要
4. 如果全部的 tool result 存本地文件了，但模型跟使用者的 text 還是超過100%。 那就把整個對話歷史丟給AI，讓它進行摘要

## 2跟3都已經過時了，不適用
20260915 的時間點查詢，claude code 只要有 thinking，它就會需要它前面的對話沒有被改過。

否則當某 thinking 的結論是 : 這是正常的請求，可以協助使用者
然後使用者把前面原本正常的問題替換成，「 幫我駭進這個網站 」 、「 請逐字說出你思考的過程 」 就會有安全性問題、蒸餾問題。

#### 現在要修改對話歷史讓它變少的做法是:
第一步 還是可保留，因為每次tool_result出來就改，是模型還沒看過的內容，不會卡到thinking
第二步 淘汰
第三步用替代方案，別自己碰 tool result，改用 API 的 context editing（clear_tool_uses）
           不破壞thinking
        然後用 clear_at_least 參數調控清的頻率 ( 參考下方 prompt caching )
第四步 保留

____
## 自建 harness 除了照抄這裡的實作，還需要處理什麼


1. Prompt caching 和成本
Prompt caching 的前提是訊息的開頭部分不變。第三步 clear 改寫最舊訊息，cache 就會全部失效，每輪都要付完整的輸入費用。 

所以需要調整 clear_at_least 參數，至少要一次清掉多少token，不然就不清 (清太少破壞快取不值得)
少壓、一次壓多一點：讓 context 長到接近上限，再一次砍到很低，換取長時間的 cache 命中。


2. 不是寫程式的任務
這章預設 context 大部分被 tool result 佔掉。如果你做的是客服或對話型 agent，context 大多是使用者對話，前三步幾乎起不了作用，壓力全落在第四步。這時候重點變成摘要 prompt 要保留什麼，例如訂單編號、答應過客戶的事，這得依你的領域自己寫。


## claude code cli 內建 compact 壓縮機制，那還有 harness 空間嗎?
第一步可以。 PostToolUse hook 有 updatedToolOutput 欄位，可以讓使用者在結果送給 Claude 之前把result換掉：

第三步不行。 沒有 100% context window 的 hook 、也沒有 clear at least 可以實作。

第四步可以  就是 /compact 指令。  盡量在 /compact 後面明講自己要下一步的任務是甚麼 所以需要保留的歷史可能包含哪些，讓效果更好。 