# Projector Prototype

投影筒まわりを新しくするための試作

## Circuit

LED回路と回路

### Problem

- 全体的に基板が劣化していて不安定なので、新しくしたい。

- パワーLEDの電流制限にセメント抵抗を用いており効率が悪く、発熱がひどい。

- 32個の投影筒は個別に制御できない。まとめてON、OFFできるのみ。
  演出として、地平線より下の領域が消灯できるほうがいい。過去に個別にPWM制御できた時代があるらしいが、ノイズがひどかったため、廃止されたらしい。

- 1枚の基板に32個の点灯回路を配置していて、保守性が悪い。

### Idea

- スイッチング方式のLEDドライバを用いる。
  
  - 効率が良い。電流を減らせる。
  - セメント抵抗より小さくできる。
  - 調光できる。

- 点灯回路を投影筒に内臓する。

  - 基板分割で故障や仕様変更への保守性が上がる。
  - 構造球中央に大きな基板をおかなくて済む。
  - デバッグのために投影筒単体で動作させやすい。

- DMX512を用いた調光制御。
  
  - デイジーチェインで拡張性が高い。星座絵や一等星を同一バス上に配置したい。
  - DMA512はAddressibleで制御しやすい。
  - RS485の差動信号なので長距離伝送とノイズに強い。

- 投影筒単体で無線制御可能なサブシステムとする。

  - モータや昼光と分離してシンプルなサブシステムにする。
  - 使いやすいように汎用的な通信プロトコルにする。( WiFi + REST API ? )

- 12V 駆動化

  - 高電圧 + スイッチング駆動で電流を減らせる。-> 損失減、配線に有利。

- 部品は可能な限り秋月等で入手できるようにする。

### Concept

```mermaid
flowchart TB
  subgraph 構造球基板
    A(ルータ) --> |WiFi| B(マイコン)
    B --> |UART| C(RS485ドライバ)
  end

  subgraph 投影筒基板1
    D1(RS485ドライバ) --> |UART| E1(マイコン)
    E1 --> |PWM| F1(LEDドライバ)
    F1 --> G1(パワーLED)
  end

  subgraph 投影筒基板2
    D2(RS485ドライバ) --> |UART| E2(マイコン)
    E2 --> |PWM| F2(LEDドライバ)
    F2 --> G2(パワーLED)
  end

  構造球基板 --> |DMX| 投影筒基板1 --> |DMX| 投影筒基板2
```

- LEDドライバ：CL6808  
  https://akizukidenshi.com/catalog/g/gI-06278/
  
  Max 1.2A  

- 投影筒マイコン：ATTiny 202  
  https://akizukidenshi.com/catalog/g/gI-15731/

  低価格、Arduino IDE 使用可能  
  不足気味・・・  
  [これ](https://www.arduino.cc/reference/en/libraries/dmxserial/)をATTiny202用に改造してDMXを実装できないか。

- RS485ドライバ：MAX485ED  
  https://akizukidenshi.com/catalog/g/gI-16568/

- 構造球基盤マイコン：Seeed Studio XIAO ESP32C3  
  https://akizukidenshi.com/catalog/g/gM-17454/

  WiFiを使いたいのでESP32、Arduino IDE も使用可能  
  ESP32-WROOM-32に対して書き込み回路が不要なので楽かも  
  [外部アンテナは利得がいいっぽい](https://intellectualcuriosity.hatenablog.com/entry/2022/09/03/143619)  
  [鯖実装は多分楽。](https://github.com/me-no-dev/ESPAsyncWebServer)

- 4芯シールドのケーブル  
  https://akizukidenshi.com/catalog/g/gP-07457/

  ノイズ対策でシールド線がよさそう
