## 子任務的過程有時候都是雜訊，我們只挑出子任務的結果進入到對話歷史，凝鍊上下文

新增一個 tool。tool 的功能是讓 AI 去開一個乾淨的 session，然後把參數的 prompt 放進去。

接著當新的一輪 session 跑到最後 ( 無任何工具呼叫 )，我們就判斷那些文字是任務完成報告。於是，就把那一則 json 的 type: text 取出來，return 回 tool 函數。 

這樣主 agent 透過呼叫 tool 函數就拿到了結果報告，結果以外的報告都沒有進入對話歷史

## 運作方式
透過看對話歷史的字串，當看到兩個 tool_use , name:task。 我們知道現在需要兩個 subagent 做子任務。

for subagent in subagentList:
    子任務報告 = run_task(subagent.task_prompt )

    對話歷史.append( 子任務報告 )

對 python 來說，每個 subagent 都是呼叫自己的 run_task 函數，所以 return 結果它也知道是來源於哪個 prompt 。`但對 ai 來說，我們只是把所有任務報告塞進對話歷史，它無法辨認哪個報告是在回應 subagent A 的 task_prompt`

所以 AI 回傳 tool_use 請求我們呼叫函數使用工具的時候，會先給我們一個 id。
我們需要把最後一行改成這樣 :  

對話歷史.append( subagent.tool_use_id ,子任務報告結果 )

從對話歷史就看得出 任務 -> 任務結果 :

 {"role": "assistant", "content": [
      {"type": "tool_use", "id": "toolu_A", "name": "task",
       "input": {"prompt": "找出這個專案用什麼測試框架"}},
      {"type": "tool_use", "id": "toolu_B", "name": "task",
       "input": {"prompt": "摘要 agents/ 下每個 .py 的用途"}}
  ]},
  {"role": "user", "content": [
      {"type": "tool_result", "tool_use_id": "toolu_A", "content": "用 pytest ..."},
      {"type": "tool_result", "tool_use_id": "toolu_B", "content": "s01.py: ...; s02.py: ..."}]}

___
備註 

code.py 使用的 for subagent in subagentList:
    子任務報告 = run_task(subagent.task_prompt )


這樣的寫法是照順序執行的，一個 subagent 跑完才跑下一個 subagent。 實務上我們會用併行的，就把所有任務都發派給一個 thread 去處理。

但概念是一樣的。 只是須小心同時修改同一檔案等問題

## 訂閱制也可以實現這樣的 harness ，已經完全內建了
判斷何時該用 subagent 開發來節省 context ，這個思考是唯一的門檻。

但基本款就是全部交給 AI 判斷，只要註冊好 subagent 開發的 tool (內建有)，AI 判斷合適的時候就會自己呼叫。