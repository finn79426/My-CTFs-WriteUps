---
title: My-CTFs-WriteUps
tags: CTFs,WriteUps
---
# Index
* [TKU-TDOH](#TKU_TDOH)

# TKU_TDOH
題目連結：[k1tten.me](http://k1tten.me:4000/challenges)

* [ping1 (command inject)](#ping1)
* [ping2 (command inject)](#ping2)
* [Meow Dir (LFI)](#Meow_Dir)
* [Meow Dir2 (LFI)](#Meow_Dir2)
* [Dig (Reverse DNS)](#Dig)
* [MCS (SSRF)](#MCS)
* [MCS_2 (SSRF)](#MCS_2)
* [MXS (XXE)](#MXS)
* [Meow Hashmap (Hash collision)](#Meow_Hashmap)

## ping1
解答：
<font color="white">
    ;chmod 777 flag.php   <!-- 我愛777 -->
    ;./flag.php
    解完後把flag.php還原成權限644喔
</font>

Write up：
看到題目寫"try 8.8.8.8"
好喔 那就直接輸入 8.8.8.8 進去好了
![](https://i.imgur.com/jVBMrO5.png)

輸出：
```shell=
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=60 time=0.567 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.567/0.567/0.567 ms
```
欸...這不是長的跟Linux ping的輸出一樣嘛...
直覺是大概要考你command inject吧？
因為讓我想到去年在andy打的wargame，跟cal有關的(我也忘了 = =)
偷看Hint確定是command inject

隨便下個pwd看看會怎樣
![](https://i.imgur.com/2279Puv.png)

輸出：
```shell=
ping: bad address 'pwd'
```
所以我猜後端應該是這樣子處理我的輸入
```shell=
$ ping -c 1 <input>
```
那當然會找不到叫作pwd的address了

加個特殊字元就可以了，例如 **ls -l**
 [鳥哥私房菜](http://linux.vbird.org/linux_basic/0320bash.php#redirect_com)
![](https://i.imgur.com/kJoFgsO.png)
輸出：
```shell=
total 8
-rw-r--r--    1 nginx    nginx           95 Sep 29 07:04 flag.php
-rwxrwxrwx    1 nginx    nginx         3787 Sep 29 07:06 index.php
```
這題關鍵就在特殊字元，剩下的就沒甚麼了
<font color="red">可是為甚麼沒辦法用cat啊</font>
不過就算網站沒有過濾特殊字元，考量到某些指令需要權限，所以還是要多嘗試幾遍喔

## ping2
解答：
<font color="white">
    |chmod 777 f???.php   <!-- 我愛777 -->
    |./f???.php
    解完後把flag.php還原成權限644喔
</font>

Write up：
嗯...看起來長的跟ping1一模一樣
一樣先試試一些有的沒的指令
但是你會發現如果像上一題輸入;開頭
會被系統擋在外面，<font color="red">You are hacker!</font>
這樣子就比較麻煩了，所以更要多嘗試看看哪些指令會放行

我大概測試了5分鐘左右，我發現兩個重點
1. 這只是過濾字元，不是過濾指令，太棒LA
2. 什麼！居然把 'flag' 擋住了！

詳細如下
:::danger
會擋住的：
&nbsp;;
&nbsp;*
&nbsp;&
&nbsp;flag
:::
:::success
安全的：
&nbsp;.
&nbsp;.php
&nbsp;/
&nbsp;?
&nbsp;|
&nbsp;#
&nbsp;$
&nbsp;-
&nbsp;數字
&nbsp;英文(除了"flag")
:::

不過很好玩的是，居然擋住flag，但是fla不會擋
那就又是特殊字元上場的時候了，解答翻上面反白

## Meow_Dir
解答：
<font color="white">
    path=ĮĮ/
</font>

Write up：
起初看到這題，有點霧煞煞
所以嘗試了一些方法，但是都被擋住
```
path=../                
path=./                 
path=..%2F        #URL encode
path=..%252F      #Double encode
```
突然靈光乍現，想到CSAW有解過類似這種題目
詳情請看[orange大神](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf)，因為我也不太會講 ：p

## Meow_Dir2
解答：
<font color="white">
    path=⸮⸮/flag.php
</font>

Write up：
嗯...試了超多方法，沒輒
去翻[orange大神](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf)試圖看懂，但好像是有看沒有懂...= =
**花了100pt看自己已經在看的資料**，GAN~~~~~~~~~

這題下一次要問清楚原理，因為我怎麼想都覺得不對勁
因為 . = N = \0x2e = Į
可是為甚麼，2e結尾的其他字元不行？例如： Ȯ
而且為甚麼 ⸮ = .. = \2e2e = \2e ？
輸入⸮⸮/flag.php不是會變成 ..../flag.php嘛？

這題蠻多問號的，下次要問阿貓

## Dig
解答：
<font color="white">
    $ dig TXT plurk.pw
</font>

Write up：
剛開始看到這題突然暗爽了一下
因為題目名稱是dig，那八成跟dig command有關係

題目給的是 plurk.pw 但被轉到 k1tten.me:8804
```shell=
# 先curl看看有沒有甚麼彩蛋
$ curl plurk.pw

# 嗯...沒有
```
嘗試看看DNS正反解，或許會告訴我不一樣的東西？
```shell=
$ dig plurk.pw
# ANSWER SECTION: got 217.70.184.38

$ dig -x 217.70.184.38
# ANSWER SECTION: got webredir.vip.gandi.net.
```
欸...好像走錯路了，從頭再來一遍
或許我對DNS還沒有很了解，搜尋爬個資料好了

我發現以這題來說，DNS紀錄最可疑的是CNAME跟TXT
CNAME是主機的別名，一樣先丟dig看看會有甚麼東西，但結果沒有
TXT是一種文字紀錄，說明主機的資訊，這邊我解讀為類似於註解的東西
用dig查詢TXT flag就出來了
這提醒了我們，不要在TXT上面放太多有關於主機的資訊。

## MCS
解答：
<font color="white">
    127.0.0.1/secret/flag.php
</font>

Write up：
這題折騰了我很久，在剛開始看到題目的時候
又開始以為是commad inject，但後來試了一些指令，發現不是。
而在打這題之前，我對SSRF不瞭解
起初有非常多異想天開的作法，這邊懶的慢慢講了。(笑)

直到阿貓放上Hint的時候：**I put a robot cat in my web!**

一開始以為是要用服務本身的curl，上傳一個webshell上去
然後再用netcat連上去，找flag (非常的異想天開阿有木有！！)
不過，一題100分的題目，應該是不用這麼麻煩才對 (?

然後又突然想到，可能關鍵會是在robots.txt吧
看到flag路徑，爽了一下，輸入URL，403 Forbidden
然後接下來又是一連串的天馬行空.....
當中試過最鳥的方法就是利用meow curl service http://k1tten.me:8805/secret/flag.php

最後Bandit用一句話點通我的思路：localhost = ?
RRRRRRR太崩潰啦，明明有想到，但是認為最不可能的居然....QQQQQ

>![](https://i.imgur.com/3BR8ipR.jpg)
>The closer you look,the less you see
>[name=出神入化]


## MCS_2
<font size="5">我必須說，我接下來的解題思路很機車ww 阿貓看到別揍我</font>

解答：
<font color="white">
    file:///etc/passwd
</font>

Write up：
好的，在解MCS系列的題目時，我一直不知道是利用哪種手法來解
直到解完MCS的時候：
![](https://i.imgur.com/ytDHxJS.png)

斯八拉西~真是太棒了，有了一個方向後，就可以開始往那個方向找資料了
不過我也不是單純爬個Cheat Sheet而已，不然這樣太無恥了，而且沒學到東西。

在阿貓直接表明是SSRF之後，我開始查詢SSRF的原理
大致上整理的原理是這樣：
```
某伺服器提供的某服務，這裡指的是MCS
如果使用者對服務器發出惡意的Request，這邊指的是Input URL然後送出
那就可以達到某些使用者不能做到、但伺服器可以做到的事情，這邊就是 /secret/flag.php Only Access On localhost
```

回到正題，MCS題型是一樣的，但是這次是考驗我們能不能看到所有使用者的內容
換言之，能不能訪問 /etc/passwd ?
突然想到玩LFI的時候，有個東西叫作 File URI Scheme
自己在筆電下curl file:///etc/passwd 發現效果跟直接訪問/etc/passwd一樣。

好...我詞窮了，就這樣

題外話：
從打wargame我發現到一件事，解題的時候沒想法，其實可以往「永遠都不要信任使用者所輸入的東西」，好像會舒暢許多 (?


## MXS
有點想睡 懶的打太多

解答：
<font color="white">
    <?xml version="1.0" encoding="ISO-8859-1"?>
 <!DOCTYPE yee [  
  <!ELEMENT yee ANY >
  <!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=../../../../../secret/long/user/meow/me0w/security/secret/meow.php" >]><yee>&xxe;</yee>
</font>

Write up：
能破這題，我覺得運氣成份比較高
而且起初的思考模式很瞎： <!-- XDD -->
> 好，阿貓有參加AIS3的課，
> 也有參加ASIS 2017的Write up分享，
> 如果沒有想法，或許可以從這些資訊來分析看看？
> 
> ....好，這是某種XML服務
> 最近好像有種關於XML的攻擊，在freebuf有看過相關文章
> 原來是叫XXE，趕緊研究看看
> 
> 嗯？AIS3 Final也有XXE？那阿貓應該是出XXE的關卡吧 <!-- XDDD -->
[color=#CAE]

有點累了，想睡覺。快速解決，這邊簡單帶過解題過程：
反正凡事先看看passwd，或許會有彩蛋或提示
看到passwd下面有個超長的username，一看就知道是放flag的地方齁

但是實際上會發現，並沒有留shell的位置，只有家目錄
所以可以判斷說 meow.php應該是在secret裡面
如果實際造訪meow.php，會發現甚麼都沒有
我的想法是：<font color="red">YOU CAN'T SEE ME!!!</font>
或許是php的關係，所以看不到吧

然後想到一個月前看AIS3 pre-Exem，web3是可以利用 php://filter 看到php的內容
所以輸入解答，會得到base64 code，拿去decode後，flag就出來了

好我要睡了zzz



## Meow_Hashmap
解答：
<font color="white">
{
    "Dbrk": 1,
    "DbsJ": 2,
	"DcQk": 3,
    "DcRJ": 4,
    "EArk": 5,
	"EAsJ": 6
}
</font>

Write Up：
這題卡了2個禮拜了，在昨天終於破關了
這題放在Web裡面，我以為是像[AIS3 PE Web2](http://w181496.github.io/2017/07/16/AIS3-pre-exam-writeup/)一樣
大概是PHP的漏洞吧，然後我看Source code也是黑人問號(不熟PHP)
然後在國慶日瞎忙了兩三天吧
不過在狂爬資料的同時，知道了一件事，我的解讀是這樣：
<font color=red>"如果伺服器一直在計算Hash，會導致DoS攻擊"</font>
我知道我解釋不是很好啦 = =
後來跑去問原作者Cyku，他表示，這是密碼學的題目

再回去看一下Source Code，的確是Hash collision
要碰撞超過5個就可以拿到FLAG了
只要稍為Google 一下 djbx33a collision，就可以找到答案了

不過後來原作者表示，只要Hash 超過2^16^次，也可以拿到FLAG的樣子
不過太懶了 要做這麼多筆資料 懶的試 :+1: 
