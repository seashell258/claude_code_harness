一開始請求 "ABC"  然後模型回傳

  { type: text: "了解，ABC"},
  { type: tool_use: bash command ls}

  本地承接這個回傳的.py，會去看有沒有 tool use 字樣來決定要不要執行 bash。

  等到bash的result出來 我就把 result append 到整個對話歷史裡面。 

  整個request變成
{"role": "user", "訊息": "創建ABC.py"},

  {role: AI, 訊息 :[ 
  { type: text: "了解，你說創建ABC.py"},
  { type: tool_use, name: bash , input: ls}]
  }

{"role": "user", "訊息":[tool_result : 目錄裡面現在有 readme.md  ] },

然後"整包對話歷史"丟給模型。因為模型是無狀態的，只有丟整串歷史模型才能正常運作。
所以後續課程會聊到如何透過處理這一大段歷史來降低 token 消耗。 

