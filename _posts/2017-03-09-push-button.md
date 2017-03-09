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

如何做到：按一下開關亮，再按一下開關滅？

#### 程式碼一

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
    ledState = !ledState; // 反轉目前 LED 腳位 (13) 的狀態
    digitalWrite(ledPin, ledState);
  }
}
{% endhighlight %}

你發現問題了嗎？事實上 LED 的明滅似乎不是我們所想的對嗎？這種不正常現象來自按鈕開關的機械彈跳 (Bounce) 特性，當按下按鈕時，D2 並不是一次從上拉的 HIGH 直接跳到 LOW，而是在 HIGH 與 LOW 之間來回切換好幾次才成為穩定的 LOW 狀態，這種彈跳的暫態現象通常可能持續 10 毫秒到 50 毫秒左右。

由於 loop() 速度極快，每次按下的瞬間，事實上 digitalRead() 已經讀了數十次以上的開關 HIGH(奇數次) LOW(偶數次) 切換的數值了，所以如果上述的開關機械彈跳特性在達到穩定前，digitalRead() 讀進來的是偶數次 (LOW) 的開關狀態，那麼下一個狀態仍然是回到奇數次 (HIGH)，然而，我們的程式邏輯只判斷 swState == LOW 時改變 LED 狀態，因為不符合條件，故 LED 並不會切換狀態。

我們先在原來的程式碼中加寫一段監控計數的程式碼來觀察這個現象。

#### 程式碼二

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
    ledState = !ledState;
    digitalWrite(ledPin, ledState);

    Serial.print("LOW count:");
    Serial.println(counter);
    counter++;
  }
}
{% endhighlight %}

看到了嗎？當你只是按一下的瞬間，程式進入到判斷 swState 為 LOW 的狀態已經跑了十幾次了。

![Click Count](/freakhq/assets/img/posts/pushbutton_bouce_count.png)

解決這個問題的辦法最直接的是時間延遲法，我們多準備一個延遲的時間，來避免按壓按鈕的機械彈跳特性，意即當偵測到按鈕被按下，D2 位準變 LOW 後，多延遲一段時間再判斷一次，若狀態還是 LOW，那表示彈跳情況已消失，按鈕已穩定在被按下的狀態，此時才去反轉 LED 明滅狀態。

#### 程式碼三
{% highlight c %}
const byte swPin = 2;
const byte ledPin = 13;
boolean ledState = LOW;
int counter = 0;
int debounceDelay = 200; // debounce delay (ms)
unsigned long lastMillis; // record last millis

void setup() {
  Serial.begin(9600);
  pinMode(ledPin, OUTPUT);
  pinMode(swPin, INPUT_PULLUP);
  digitalWrite(ledPin, ledState);
}

void loop() {
  boolean swState = digitalRead(swPin);
  if (swState == LOW) {
    if (debounced()) { //debounced: reverse LED state
      ledState = !ledState;
      digitalWrite(ledPin, ledState);
      Serial.print("LED Change State count:");
      Serial.println(counter);
      counter++;
    }
  }
}

boolean debounced() { // check if debounced
  unsigned long currentMillis = millis(); // get current elapsed time
  if ((currentMillis - lastMillis) > debounceDelay && digitalRead(swPin) == LOW) {
    lastMillis = currentMillis; // update lastMillis with currentMillis
    return true; // debounced
  }
  else {return false;} // not debounced
}
{% endhighlight %}

可是老師，如果我按鈕一直按著，LED 會一直明滅耶。這也是由於 loop() 速度極快，每次按下的瞬間，事實上 digitalRead() 已經讀了數十次以上的開關 HIGH(奇數次) LOW(偶數次) 切換的數值了，所以你一直按著，loop() 還是一直在跑，每次 digitalRead() 讀進來的如果是偶數次 (LOW) 的開關狀態，那麼下一個狀態仍然是回到奇數次 (HIGH)，如果 digitalRead() 的是奇數次 (HIGH)，我們的程式邏輯判斷 swState == LOW 時，就會改變 LED 狀態。

我們這裡再多加一行程式碼，用一個空迴圈的寫法，讓 swState 是 LOW 狀態的時候，進入一個什麼動作都不會作的迴圈，然後一直等到 swState 狀態改變為 HIGH（也就是你再次放開的時候），才讓程式跳脫空迴圈，從頭再來過。

#### 程式碼四
{% highlight c %}
const byte swPin = 2;
const byte ledPin = 13;
boolean ledState = LOW;
int counter = 0;
int debounceDelay = 200; // debounce delay (ms)
unsigned long lastMillis; // record last millis

void setup() {
  Serial.begin(9600);
  pinMode(ledPin, OUTPUT);
  pinMode(swPin, INPUT_PULLUP);
  digitalWrite(ledPin, ledState);
}

void loop() {
  boolean swState = digitalRead(swPin);
  if (swState == LOW) {
    if (debounced() && digitalRead(swPin) == LOW) { //debounced: reverse LED state
      ledState = !ledState;
      digitalWrite(ledPin, ledState);

      // 等待指令 while(statement);
      // 當按鍵一直被按著的時候，給一個空迴圈，表示什麼都不做，
      // 直到按鍵不再被按著的時候才離開這個迴圈繼續其他動作。
      while(digitalRead(swPin) == LOW);

      Serial.print("LED Change State count:");
      Serial.println(counter);
      counter++;
    }
  }
}

boolean debounced() { // check if debounced
  unsigned long currentMillis = millis(); // get current elapsed time
  if ((currentMillis - lastMillis) > debounceDelay) {
    lastMillis = currentMillis; // update lastMillis with currentMillis
    return true; // debounced
  }
  else {return false;} // not debounced
}
{% endhighlight %}

## 參考連結
* [Input Pullup Serial](https://www.arduino.cc/en/Tutorial/InputPullupSerial)
* [Arduino Internal Pull-Up Resistor Tutorial](https://www.baldengineer.com/arduino-internal-pull-up-resistor-tutorial.html)
