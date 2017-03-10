---
layout: post
title: "Arduino / I05 / 光敏電阻"
date: 2017-03-09 00:00:05
description: "光敏電阻是利用光電導效應的一種特殊的電阻，當外界光線強時，光敏電阻會讀到較大的值；外界光線弱時，會讀到較小的值。"
main-class: 'arduino'
color: '#B31917'
tags:
- "Arduino"
- "Input"
twitter_text: "Arduino / I05 / 光敏電阻"
introduction: "光敏電阻是利用光電導效應的一種特殊的電阻，當外界光線強時，光敏電阻會讀到較大的值；外界光線弱時，會讀到較小的值。"
---


以　analogRead()　讀取光敏電阻的值，會回傳　0~1023　之間的值。以下的電路示意與範例程式碼，示範情境為：遮住光敏電阻（環境光不足）時，點亮　LED。

光敏電阻需要一支 10KΩ 的電阻接到地端。

## 電路示意圖

![photoresistance Circuit](/freakhq/assets/img/posts/I05-1.png)


## 範例程式

{% highlight c %}
const int ledPin = 13;
const int sensorPin = A0;

int level;
const int threshold = 150;          // 判斷亮暗邊界的數值

void setup() {
  pinMode (ledPin, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  level = analogRead(sensorPin);    // 0-1023
  Serial.println(level);
  if (level < threshold) {          // 若取得的數值小於 150
    digitalWrite(ledPin, HIGH);     // 亮燈
  } else {
    digitalWrite(ledPin, LOW);
  }
}
{% endhighlight %}

