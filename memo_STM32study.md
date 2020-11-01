### STM32 study memo
#### 参考リンク
* STM32開発環境
https://qiita.com/kztriioa/items/b886ac442d3de22a3ff9



#### main.c構成


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

* 