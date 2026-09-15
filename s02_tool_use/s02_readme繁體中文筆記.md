## 模型如何調用工具 : 
1. 回傳字串包含一個函數名稱
2. 回傳字串包含那個函數要用的參數
3. 自己寫的程式碼把 1 跟 2 組合再一起，呼叫那個函數。最後把 tool_result append 進去對話歷史

比如模型回傳
{ type: tool_use, name: bash , input: ls}

自己的程式碼會執行 : 
```

let 要呼叫的函數名稱 =  block.name    
tool_result = 要呼叫的函數名稱(**block.input)


↓

tool_result = bash(ls)
```

## 如何安全的調用工具: 

如果直接把模型回傳的訊息的 name 區塊提取出來，當成函數執行
{ type: tool_use, name: bash , input: ls}

那模型就可以隨意的調用任意它想要的函數，只要在 name 區塊塞字串就行了。 這樣不好

所以實際上我們會去限定模型只能呼叫特定函數。 做法:
```
Map 白名單 = [block.name: 執行函數1 , block.name2號 : 執行函數2]
let 要呼叫的函數名稱 =  白名單[ block.name ]
tool_result = 要呼叫的函數名稱(**block.input)
```
第一行新增一個 map。
如此，模型回傳的 block.name 不會直接被當成函數名稱去呼叫。而是要先經過一次 map 查找，才能對應到真正要呼叫的函數名稱。

這樣模型就只能呼叫 map 裡面查找的到的函數，也就將可呼叫的函數被控制在一個白名單之內。


## 使用 tool，這種 harness ，還屬於訂閱制能實現的範圍

訂閱制跟 api 的回傳結果是一樣的
差別在於，api 可以自由組裝 loop 的邏輯。


claude code cli 不會因為你用訂閱制就吞掉回傳的形式 
還是看的到
{tool_use , bash, ls}  這樣子的訊息

但 claude code cli 不會給你在 loop 裡面組 request 的機會，你 tool 呼叫完之後，沒辦法自己把結果 append 回去。 cc cli 會自動化完成這個 loop 。 

但訂閱制度有提供端口讓我們註冊新的 tool。 就像我們用 python 腳本可以定義一個 tool 函數，然後把函數名稱塞進我們的白名單 map 那樣。

claude mcp add my-tools -- python /完整路徑/my_tools.py