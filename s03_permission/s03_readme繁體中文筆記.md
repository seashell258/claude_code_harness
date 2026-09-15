## 在 tool_use 的函數執行前加個 safe check 函數

rm -rf 直接 deny ，回傳給模型 permission denied (得回傳，這樣模型才能繼續 loop)

如果是其他不確定的，就問使用者，看它回 accept 還 deny 

概念很簡單，而且 claude code cli 都已經做完了。它會幫你 deny 和問你許可。 

值得思考的會是要對哪些命令做過濾，可以看下面例子 :

1. write_file("cleanup.sh", "rm -rf ~")：檔案在 workspace 內，放行
2. bash("sh cleanup.sh")：指令裡沒有 rm，放行

每一步單獨看都合法，合起來就是破壞性操作。

由於同一種操作就有無限種表達方式，只要允許 bash，就很難靠著黑名單去合理的擋下 rm rf。
所以可能 safe check 可能會使用白名單、或是把整行命令 split 開，對每個部份各自做字串比對

但無論如何，很難把 safe check 想的完備，AI 意外的 bash 了糟糕的指令的可能性一直都在。

所以使用沙盒隔離整個環境，才能算安全。

## 這樣的 harness 也在訂閱制可以實現的範圍內
因為訂閱制提供 hook : 讓你在特定時機執行自定義的函數 

所以我們可以在每次 tool 執行前，跑自己定義的 safe check 函數。 當然必要性可能不高，因為 claude code cli 已經內建了它們家的 safe check函數。