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
image: "/freakhq/assets/img/posts/pushbutton_bouce_count.png"
twitter_text: "Arduino / I01 / 按壓開關"
introduction: "按壓開關 (Push button) 是一個常開按鈕，不按壓時為開路不連通(OFF)，按壓時形成短路連通(ON)。"
---

我們利用按壓開關按下時回傳給 Arduino 板子的訊號，做為觸發輸出端改變的判斷，以下的電路示意與第一段程式碼，示範情境為：按下按壓開關則點亮主板內建 LED，放開按壓開關則關閉主板內建 LED

## 電路示意圖

![PushButton Circuit](https://www.arduino.cc/en/uploads/Tutorial/inputPullupButton.png)

## 範例程式

{% highlight c %}
const byte swPin = 2;    // 開關腳位
const byte ledPin = 13;  // 主板內建 LED 腳位
boolean ledState = LOW;  // initial state of LED

void setup() {
  pinMode(ledPin, OUTPUT);
  pinMode(swPin, INPUT_PULLUP);   // enable pull-up resistor
  digitalWrite(ledPin, ledState); // set LED OFF
}

void loop() {
  boolean swState = digitalRead(swPin);
  if (swState == LOW) {               // switch pressed
     digitalWrite(ledPin, HIGH);      // light up LED
  } else {
     digitalWrite(ledPin, LOW);
  }
}
{% endhighlight %}

## 思考練習

### 按一下開關亮，再按一下開關滅

#### 測試一

程式思路是，當按鈕按下時，反轉目前 LED 腳位 (13) 的狀態

{% highlight c %}
const byte swPin = 2;
const byte ledPin = 13;
boolean ledState = LOW;

void setup() {
  pinMode(ledPin, OUTPUT);
  pinMode(swPin, INPUT_PULLUP);
  digitalWrite(ledPin, ledState);
}

void loop() {
  boolean swState = digitalRead(swPin);
  if (swState == LOW) {
    digitalWrite(ledPin, !ledState);
  }
}
{% endhighlight %}

你發現問題了嗎？事實上 LED 的明滅似乎不是我們所想的對嗎？這種不正常現象來自按鈕開關的機械彈跳 (Bounce) 特性，當按下按鈕時，D2 並不是一次從上拉的 HIGH 直接跳到 LOW，而是在 HIGH 與 LOW 之間來回切換好幾次才成為穩定的 LOW 狀態，這種彈跳的暫態現象通常可能持續 10 毫秒到 50 毫秒左右。

由於 loop() 速度極快，每次按下的瞬間，事實上 digitalRead() 已經讀了數十次以上的開關 HIGH(奇數次) LOW(偶數次) 切換的數值了，所以如果上述的開關機械彈跳特性在達到穩定前，digitalRead() 讀進來的是偶數次的開關狀態，那麼下一個狀態仍然是回到奇數次 (HIGH)，然而，我們的程式邏輯只判斷 swState == LOW 時改變 LED 狀態，因為不符合條件，故 LED 並不會切換狀態。

我們先在原來的程式碼中加寫一段監控計數的程式碼來觀察這個現象。

#### 測試二

{% highlight c %}
const byte swPin = 2;
const byte ledPin = 13;
boolean ledState = LOW;
int counter = 0;

void setup() {
  Serial.begin(9600);
  pinMode(ledPin, OUTPUT);
  pinMode(swPin, INPUT_PULLUP);
  digitalWrite(ledPin, ledState);
}

void loop() {
  boolean swState = digitalRead(swPin);
  if (swState == LOW) {
    digitalWrite(ledPin, !ledState);

    Serial.print("Click count:");
    Serial.println(counter);
    counter++;
  }
}
{% endhighlight %}

看到了嗎？

![Click Count](/freakhq/assets/img/posts/pushbutton_bouce_count.png)

## 參考連結
* [Input Pullup Serial](https://www.arduino.cc/en/Tutorial/InputPullupSerial)
* [Arduino Internal Pull-Up Resistor Tutorial](https://www.baldengineer.com/arduino-internal-pull-up-resistor-tutorial.html)
