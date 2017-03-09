---
layout: post
title: "Arduino / I01 / 按壓開關"
date: 2017-03-09 00:00:05
description: "按壓開關 (Push button) 是一個常開按鈕，不按壓時為開路不連通(OFF)，按壓時形成短路連通(ON)。"
main-class: 'arduino'
color: '#637a91'
tags:
- "Arduino"
- "Input"
twitter_text: "Arduino / I01 / 按壓開關"
introduction: "按壓開關 (Push button) 是一個常開按鈕，不按壓時為開路不連通(OFF)，按壓時形成短路連通(ON)。"
---

我們利用按壓開關按下時回傳給 Arduino 板子的訊號，做為觸發輸出端改變的判斷，以下的電路示意與程式碼，示範情境為：按下按壓開關亮 LED，放開按壓開關則暗 LED

## 電路示意圖

![PushButton Circuit](https://www.arduino.cc/en/uploads/Tutorial/inputPullupButton.png)

## 範例程式

Basta adicionar a seguinte `meta tag` no `head`:

{% highlight c %}
<link rel="shortcut icon" href="/img/icons/favicon.ico" type="image/x-icon">
{% endhighlight %}

Algumas pessoas apoiam utilizar um formato mais adaptável que é o `png` usando as novas meta tags com size, como mostrado no código abaixo:

{% highlight html %}
<link rel="icon" type="image/png" href="/favicon-16x16.png" sizes="16x16">
<link rel="icon" type="image/png" href="/favicon-32x32.png" sizes="32x32">
<link rel="icon" type="image/png" href="/favicon-96x96.png" sizes="96x96">
{% endhighlight %}

## 思考練習

## 參考連結
* [Input Pullup Serial](https://www.arduino.cc/en/Tutorial/InputPullupSerial)
* [Arduino Internal Pull-Up Resistor Tutorial](https://www.baldengineer.com/arduino-internal-pull-up-resistor-tutorial.html)
