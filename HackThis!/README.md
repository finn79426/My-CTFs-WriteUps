---
title: HackThis! Writeups
tags: CTFs,WriteUps
---
<font size=36>
    一題一個 #
</font>

# Basic+ Level1
給你一個壞掉的b1.txt  
先用**file指令**觀察它的檔案頭  
他告訴你：b1.txt: PC bitmap, Windows 3.x format, 213 x 108 x 24  
將檔案副檔名改成.bmp即可看到flag。(bitmap = bmp)  

# Basic+ Level7
他告訴你網站上運行某向服務  
用nmap去掃這個網站，看他開了甚麼服務  
```
nmap -sV www.hackthis.co.uk -p 1-65535
```
把它的所有port都掃過一次要蠻久的  
nmap告訴你6776 port 有開一個未知服務  
接下來用nc連上去就可以看到flag  
```
nc www.hackthis.co.uk 6776
```