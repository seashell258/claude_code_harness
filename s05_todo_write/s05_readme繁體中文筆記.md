## AI 工作的時候發現錯誤就去修。最後上下文都被帶歪，忘記最初目的，所以會需要有人管理一個任務的 todo

我們新增一個 todo write 的 tool，讓 AI 找時機呼叫這函數去更新待辦清單

#### 何時 AI 會啟動 todo list，將一項任務分成多步驟? 
AI 自己決定 : 我們用 prompt 去請 AI 做事前拆分步驟、並自己判斷怎麼拆。

所以有不穩定的空間 

有更穩定需求的話，也許可以自己拆好步驟，然後照順序一個一個送出。

#### 何時AI會去 retrive todo list 來管理好現在的工作進度?
主要是 AI 自己決定，所以有不穩定的空間 : 

AI 每輪都能看到之前的全部對話紀錄，所以他會看到之前呼叫 todo write，去把第一項改成 in progess。 那現在做完了，它就會有傾向去把第一項改成 done，然後繼續做第二項

但這裡有可以 harness，讓AI表現更穩定的方式 : AI如果三輪結束都沒有呼叫 todo write 這個 tool，就在請求尾端 append <reminder>Update your todos.</reminder>

#### 訂閱制也能幹這樣的自訂 harness
每輪AI行動的結尾呼叫函數，如果沒看到對話歷史裡面有呼叫 todo write 工具，就把 counter +1。 如果加完 counter == 3，那就`"把提醒訊息 append 進請求裡面"`

這怎麼可能呢? claude code cli 跟自訂 agent 的差別就是 : 我們不能介入 AI 的 loop，在AI整個 loop 跑完之前去更改每次 loop 裡丟給模型的訊息。

然而 claude code cli 的 hook 提供這樣的端口，讓我們能模擬自訂 agent 介入 loop的能力

Hook 輸出這段 JSON，裡面的文字就會變成模型看得到的 system message：

{
  "hookSpecificOutput": {
    "hookEventName": "PostToolBatch",  
    "additionalContext": "<reminder>Update your todos.</reminder>"
  }
}

備註 : postToolBatch 這個觸發時機是 : AI一次可能回傳五個 tool_use 命令，我們等到一整批做完才觸發一次 hook 。
________

另外，每輪 hook 去觸發的 py 函數都是獨立的，所以counter沒辦法保留，要把counter的記數寫進獨立的檔案裏面，這樣才能保留。

