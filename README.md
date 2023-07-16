# EMUZ80-6502LED
MEZ6502LED基板によってEMUZ80に増設した7セグメントLEDの制御を追加したファームウェアです。  
CMOS 6502専用です。NMOS6502ではクロック信号のHighレベルを維持できる時間が最大40uSなのでLED制御に必要な時間が得られません。  

LED制御レジスタを0x9000 - 0x901Fに配置します。

![MEZ6502LED](https://github.com/satoshiokue/EMUZ80-6502LED/blob/main/MEZ6502LED.jpeg)

MEZ6502LED  
https://github.com/satoshiokue/MEZ6502LED

PICは6502のクロック信号がHighレベルを維持している期間にLEDを制御します。  

ソースコードは電脳伝説さんのEMUZ80用main.cを元に改変してGPLライセンスに基づいて公開するものです。

## ファームウェア
emu6502_led.cをEMUZ80で配布されているフォルダemuz80.X下のmain.cと置き換えて使用してください。  
ターゲットのPICを適切に変更してビルドしてください。  


## アドレスマップ
```
RAM   0x0000 - 0x0FFF 4Kbytes (0x1FFF 8Kbytes:PIC18F47Q83,PIC18F47Q84)
ROM   0xC000 - 0xFFFF 16Kbyte

UART  0x8018   Control REGISTER
      0x8019   Data REGISTER

LED   0x9000 - 0x901F
```

## PICプログラムの書き込み
EMUZ80技術資料8ページにしたがってPICに適合するemu6502LED_Qxx.hexファイルを書き込んでください。  

またはArduino UNOを用いてPICを書き込みます。  
https://github.com/satoshiokue/Arduino-PIC-Programmer

BASIC  
PIC18F47Q43 emu6502LED_Q43.hex  
PIC18F47Q83 emu6502LED_Q8x.hex  
PIC18F47Q84 emu6502LED_Q8x.hex  

DEMO LEDトレースモードでBASICを起動します  
PIC18F47Q43 emu6502LED_DEMO_Q43.hex  
PIC18F47Q83 emu6502LED_DEMO_Q8x.hex  
PIC18F47Q84 emu6502LED_DEMO_Q8x.hex  

```
6502 EhBASIC [C]old/[W]arm ?

Memory size ?

Enhanced BASIC 2.22p5a
3327 Bytes free

Ready
```
起動メッセージが出たら「c」キーを押します。Memory sizeでリターンキーを押すと利用可能な最大メモリでBASICの起動が完了します。


Enhanced 6502 BASIC by Lee Davison  
https://philpem.me.uk/leeedavison/6502/ehbasic/

## 6502プログラムの改編
バイナリデータをテキストデータ化してファームウェアの配列rom[]に格納すると6502で実行できます。

テキスト変換例
```
xxd -i -c16 foo.bin > foo.txt
```

## トレースモード
0x9006 Trace modeで表示方法を選択、0x9004 Display modeを0x04にするとトレースモードに入ります。  
6502が取得するバスの状態を表示します。6502が実行中の命令ではありません。  
LED表示は左から4桁がアドレス、2桁がリードで消灯、ライトでドットが点灯、右2桁がデータです。  

LED表示のウェイトは150ミリ秒です。  
ウェイトはソースコードの217行目で指定しています。  
```
#define WAIT_LED_TRACE 150	// 150 msec
```

UARTを選択するとバスの内容が「すべて」取得できます。  
# Display control resistors
レジスタ初期値は0

## 0x9000-0x9003 HEX data

|Address|LED Position|
| --- | --- |
|0x9000|88 -- -- --|
|0x9001|-- 88 -- --|
|0x9002|-- -- 88 --|
|0x9003|-- -- -- 88|

書き込みで即時表示更新

## 0x9004 Display mode

|Code|Description|
| --- | --- |
|0x00| off  
|0x01| Select bank1  
|0x02| Select bank2  
|0x03| hex dump at 0x9000 - 0x9003  
|0x04| Trace mode  
|0xFF| "8.8.8.8. 8.8.8.8."  

## 0x9005 Display brightness
```
0x00(min) - 0x0f(max)  
```

## 0x9006 Trace mode

|Code|Description|
| --- | --- |
|0x00| off  
|0x01| LED Display  
|0x02| UART  
|0x03| LED & UART  

## 0x9010 - 0x9017 No-Decode bank1
1になったデータビットに対応したセグメントが点灯  
0xF004 Display modeに0x01(=Select bank1)を書き込むと表示更新
|Bit|7|6|5|4|3|2|1|0|
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Segment Line|DP|A|B|C|D|E|F|G|
## 0x9018 - 0x901F No-Decode bank2
1になったデータビットに対応したセグメントが点灯  
0xF004 Display modeに0x02(=Select bank2)を書き込むと表示更新
|Bit|7|6|5|4|3|2|1|0|
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Segment Line|DP|A|B|C|D|E|F|G|

## BASICによる制御例

LED全点灯、消灯
```
POKE $9000,255
POKE $9000,0
```

0x9000-0x9003の値をLEDに表示
```
POKE $9004,3
POKE $9000,$12
POKE $9001,$34
POKE $9002,$AB
POKE $9003,$CD
```

## 参考）EMUZ80
EUMZ80はZ80CPUとPIC18F47Q43のDIP40ピンIC2つで構成されるシンプルなコンピュータです。

![EMUZ80](https://github.com/satoshiokue/EMUZ80-6502/blob/main/imgs/IMG_Z80.jpeg)

電脳伝説 - EMUZ80が完成  
https://vintagechips.wordpress.com/2022/03/05/emuz80_reference  

EMUZ80専用プリント基板 - オレンジピコショップ  
https://store.shopping.yahoo.co.jp/orangepicoshop/pico-a-051.html
