---
title: "在 python 上使用 matlab、octave"
tags: ["文章","python","matlab"]
date: 2020-07-29T12:58:00+08:00
draft: false
author: "永格天(武則天、wxex)"
featuredImagePreview: "http://dr.sudo.host/vnx78Y"
featuredImage: "http://dr.sudo.host/vnx78Y+"

---
<!-- 「」 -->
# 師：「那個誰，幫我把這幾隻 matlab 程式轉 python」
「......老師你是嫌 matlab 太貴要轉移陣地嗎?」
「不要管那麼多！」
「老師那改成能在 python 使用 matlab 的功能好不好?」
「...好，交給你了」


# 在 python 上使用 octave

octave 簡單來說就是 matlab 的簡化版，雖然語法上完全相容，也有自己的 packages，但沒有 matlab 的豐富。 octave 是自由軟體。

這裡推薦使用的 package 是 [oct2py](https://github.com/blink1073/oct2py)，已有7年多的歷史，且近期還有在維護，使用上也很簡單。
安裝方式 ```pip install oct2py```，他另外還有 ```conda install -c conda-forge oct2py```，如果有用 conda 的可以考慮。
此外，這個套件僅支援 octave 4.0 以上(含)的版本，經測試目前最新的 5.2.0 可以使用。

在你安裝好 oct2py 後還不能直接使用，要先確定 ```C:\Octave\Octave-5.2.0\mingw64\bin``` 中有沒有一個 ```octave-cli.exe```檔案，如果有再將```C:\Octave\Octave-5.2.0\mingw64\bin```加入系統環境變數中
![加入系統環境變數中](http://dr.sudo.host/v8LBvD+)

再來準備範例檔案(**需擺在同一目錄下**)    

1. t.m
```matlab
function d = t(x,y)

out1 = x+y
out2 = x*y
disp(out1)
disp(out2)
d=[out1,out2]

```

2. test_octave.py
```python
from oct2py import octave, Oct2PyError, Oct2Py

octave.ones(3) # 執行 octave 原生函式

try:
    t = octave.t(3, 9) # 執行 function
    print(t)
    print(t[0])
    print(t[0][0])
except Oct2PyError as e:
    print(str(e))
```

![code result](http://dr.sudo.host/pXsCbX+)

**剩下的進階功能請直接去 [官方文件infomation](https://oct2py.readthedocs.io/en/latest/source/info.html)、[官方文件examples](https://oct2py.readthedocs.io/en/latest/source/examples.html) 看吧！**


# 在 python 上使用 matlab

matlab engine **目前** 只能在 python 2.7, 3.6, 3.7([詳細支援名單看這](https://www.mathworks.com/help/matlab/matlab-engine-for-python.html))。    

當你安裝好 matlab 後，在 ```C:\Program Files\Polyspace\R2019a\extern\engines\python``` (R2019a 是看你安裝哪個版本)
中找到 setup.py，並在此資料夾以工作管理員打開 cmd 或 powershell    

打開 cmd 或 powershell 後，在裡面輸入 ```python setup.py install``` 來安裝。    

環境準備好了，接下來準備要用的檔案(**需擺在同一目錄下**)    

1. test.m
```matlab
function out1,out2 = t(x,y)

out1 = x+y
out2 = x*y
disp(x)
disp(y)
disp(out1)
disp(out2)
```

2. test_matlab.py
```python
import matlab
import matlab.engine
eng = matlab.engine.start_matlab()

out1,out2 = eng.test(2.0, 3.0,nargout=2)
print(out1)
print(out2)

eng.mod(5,3) # matlab 原生 function
```

test.py 重點有4個    

```python
out1,out2 = eng.test(2.0, 3.0,nargout=2)
```
1. test.m 中的函式名稱是 "t"，但 ```eng.text``` 是用檔名來判斷引用的來源，所以要用檔名當來源。    

2. ```nargout=2```，這個是代表你有幾個返回的回傳值，就要改成對應的數字，不然只會返回第一個。

3. ```eng.mod(5,3)```，matlab 內建函式(比如這裡的mod)可以直接用這樣的方式呼叫。

4. test.m 的 disp() 沒有效果

以上是最基礎的用法
後面還有共用 matlab session 的 [matlab.engine.find_matlab](https://www.mathworks.com/help/matlab/apiref/matlab.engine.find_matlab.html) 跟 [matlab.engine.connect_matlab](https://www.mathworks.com/help/matlab/apiref/matlab.engine.connect_matlab.html) 方法
以及啟動 async 的 [matlab.engine.start_matlab(async=True)](https://www.mathworks.com/help/matlab/apiref/matlab.engine.start_matlab.html#buj7y8n-async)
更多 method 請看[這裡](https://www.mathworks.com/help/matlab/referencelist.html?type=function&category=matlab-engine-for-python&s_tid=CRUX_gn_function_matlab-engine-for-python)，因為很少場合會用就不介紹了。






<!--  -->
