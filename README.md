RTOS 自習

# タスク

* タスクとは、複数の処理を並行処理するためのもの。
  * 並行に処理する必要がなければタスクを使う必要はない
  * OS がタスクを制御する
  * 優先度によってどのタスクを実行するか制御される
  * タスクとしての処理をおこなうものをタスクエントリーと呼ぶ。ようするに関数である。
  * OS の制御により、このタスクエントリーが呼ばれる。

* タスク生成
  * cre_tsk を使う
  * タスクエントリーに注意
    * OS がディスパッチするとタスクエントリーが呼ばれる。
    * READY→RUNNING と状態遷移する
    * タスクエントリー内では(長時間)CPUを占有するようなことをしてはいけない
    * for(;;){ レジスタ読んで確認 } のようなことをしてはいけない。
    * 他のタスクが CPU を使えなくなる。
    * 迅速に処理しとっとと終了すべし
  * 優先度に注意
    * ハードウェアに近いタスクほど高優先度
    * そうでないタスクが低優先度
    * カメラ全体を考慮して優先度が決定されるので、勝手に優先度を設定しないこと。
  * スタックサイズに注意
    * スタックサイズはタスクが利用できるメモリ
    * 自動変数や関数呼び出しの引数を記憶したりする
    * カメラではスタックサイズはデフォルトで4096バイト
    * 大きすぎる自動変数(配列とか)を作ったり、関数呼び出しが深すぎるとスタックオーバーフローする
    * スタックオーバーフローしてもカメラはASSERTしたりPanicなどしない。
    * 原因調査しづらい非常に厄介な現象となるので注意

* タスク終了
  * ExitTask を使う (タスクエントリー内で呼ぶ)
  * タスク終了するときは ExitTask すべし

* タスク情報
  * chg_pri 優先度変更
  * ref_tsk タスク状態参照
  * slp_tsk タスクを WAIT へ状態遷移させる
  *  wup_tsk タスクを READY へ状態遷移させる

* 状態遷移
  * タスクは複数の状態がある
  * READY <=> RUNNING は OS によって遷移される
  * RUNNING => WAITING => READY は タスクがみずから状態遷移させる
  * RUNNING => DORMANT も然り

# タスク間通信

* セマフォ
  * バイナリセマフォ
  * 計数セマフォ
    * 初期値 2 以上でセマフォを作成した場合 TakeSemaphore してもタスクは待たされない

* イベントフラグ

* ミューテックス
  * ミューテックスはようするに初期値 0 のバイナリセマフォと役割りは同じ
  * 優先度逆転は 1 つのセマフォを 2 つ以上のタスク(コンテキスト)で操作しようとするから発生する
  * 対策 セマフォ2つ用意する
    * セマフォの関数をニューテックスの関数に置き換えただけでは何も変わらない

* 同期オブジェクトの違い
  * セマフォ
  * ミューテックス
  * イベントフラグ

* 実装時の注意事項
  * プライオリティインバージョン
  * デッドロック
    * 複数のタスクが同じ順番でセマフォ確保すること
    * セマフォの扱いは入れ子とすること
    * 入れ子にしなくてもデッドロック回避できるが、入れ子のほうが分かりやすい

# 割り込み

* 割り込み制御
  * ディスパッチ禁止
  * 割り込み禁止
  * 割り込みマスク

# メモリ

* ヒープ
  * タスクごとに割り当てられる(スタックのサイズは CreateTask の引数に指定する)。
  * 関数ネストが深すぎたり、自動変数のサイズが大きすぎると「スタックオーバーフロー」が発生する。
  * スタックサイズはコンパイル時に決まる。
  * こういうところで使われる:
    * 関数の自動変数
    * 関数の引数
    * 関数呼び出し時の戻り先アドレス"

* スタック
  * 実行時に確保する。OSから割り当てられる。
  * こういうところで使われる:
    *  malloc/free
    *  AllocateMemory/FreeMemory

* 実施する上での注意事項
  * キャッシュ
  * スタック
  * ヒープ
  * フラグメンテーション
    * malloc/freeしているとヒープ領域が断片化する。様々なサイズの領域を確保するので虫食いのような状態となり、いずれヒープ領域から必要なサイズのメモリを確保できなくなる可能性がある。
    * 対策としては、メモリプールを確保しておき、プールの領域サイズを固定して使うようにする( 使用する最大値を見積もる必要がある )
