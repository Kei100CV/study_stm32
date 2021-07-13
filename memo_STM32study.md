### 参考プログラム
#### 開発環境
https://qiita.com/yosihisa/items/136bcc09c466227303a2
https://qiita.com/usashirou/items/65be086c28f7a6feac7d
https://qiita.com/kztriioa/items/b886ac442d3de22a3ff9

「MCU/MPU selector」ではなく「Board selector」を選択
#### デバッガ書込み
build(ハンマー) => debug(虫眼鏡) => switchを押す
エディタの行をダブルクリックでブレークを貼ることができる。
デバッグで使う機能：
  resume, step over, step in, 

#### Lチカ
https://depfields.com/gpio-led-toggle-apl/
https://qiita.com/yosihisa/items/136bcc09c466227303a2
仕様：
入力：PC13　フローティング入力 押しボタンスイッチ
出力：PA5　プッシュプル出力+500Ω+LED
```C
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);   //LEDを点灯
HAL_Delay(500); //500ms待つ
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET); //LEDを消灯
HAL_Delay(500); //500ms待つ
```


#### 外部入力信号による割り込みアプリケーション
GPIOCの値はデフォルトがGPIO_PIN_SETのようだ
同じSTM32でも型番によってドライバー関数名やインターフェースは大きく異なるようだ

```C
HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13); // default set
if (time_led == 500) {
    if (state_blue_sw == GPIO_PIN_RESET) {
        // flag_speed_led = 1;
        time_led = 250;
        HAL_Delay(2000); // time_led ms待つ
    } else {
        // time_led = 500;
    }
} else if (time_led == 250) {
    if (state_blue_sw == GPIO_PIN_RESET) {
        time_led = 500;
        HAL_Delay(2000); // time_led ms待つ
    } else {

    }
}
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);   //LEDを点灯
HAL_Delay(time_led); // time_led ms待つ
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET); //LEDを消灯
HAL_Delay(time_led); // time_led ms待つ
```
#### STM32へのFreeRTOS自前移植
https://zenn.dev/peterleif/articles/908c9dd433248a
https://depfields.com/freertos/

1. FreeRTOSをダウンロード
2. Sourceファイルをプロジェクト内にコピーしてパスを通す
    * 基本的には全部コピーして使えるがGCC内のファイルは使用するMPUに関わるもののみコピーしてよい
      * portable/Common, GCC, MemMan 
      * portable/GCC/ARM_CM4F
    * プロジェクトのフォルダを右クリックしてProperties -> c/c++ General -> Path and Symbolsを選びInclude pathを追加
    * sourceタブからソースのフォルダを追加する
4. STM32のデフォルトの割り込みライブラリはFreeRTOSの関数名と競合するので削除する 
5. heap*.cは1つのみ残し他は削除する必要がある
6. FreeRTOSConfig.hの作成
   * IDLE_HOOK, TICK_HOOK, stack overflow 検知の関数はデフォルトでないのでオフにする
   ```C
   #define configUSE_IDLE_HOOK				0 // default 1
   #define configUSE_TICK_HOOK				0 // default 1
   #define configCHECK_FOR_STACK_OVERFLOW	0 // default 2
   ```

* SYS
* NVIC


### STM32 関数説明

#### HAL_GPIO_WritePin関数
電流のON/OFFをする関数。
* HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_SET);
  * 第１引数はGPIOのグループ
  * 第２引数はグループ内のピン番号
  * 第３引数はON/OFF: ONにするときは「GPIO_PIN_SET」、OFFにするときは「GPIO_PIN_RESET」
* 使用例
  * PA１ピンを使用するとき：
    * HAL_GPIO_WritePin(GPIOA, GPIO_PIN_1, GPIO_PIN_SET);
  * PB10の電流をON/OFFしたい時:
    *  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_10, GPIO_PIN_SET);
  * PC6の電流をON/OFFするとき: 
    * HAL_GPIO_WritePin(GPIOC, GPIO_PIN_6, GPIO_PIN_SET);

#### HAL_Delay関数
引数に入れた値の分だけ待つ関数。単位はms
処理の流れ
1. 指定したピンの電流をONにする
2. その状態のまま一秒待つ
3. 指定したピンの電流をOFFにする
4. その状態のまま一秒待つ
5. 1に戻る
