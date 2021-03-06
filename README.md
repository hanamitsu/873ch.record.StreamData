# 873ch.record.StreamData
list型の数列を時系列で管理するためのパッチ。動画化して保存する。動画化したデータをlist型に再展開する。

フレームレートも変換可能

<img src="https://user-images.githubusercontent.com/8988162/105933481-d41b1080-6091-11eb-9268-ad0b805fd710.gif" width="500">
<img src="https://user-images.githubusercontent.com/8988162/105932773-c7e28380-6090-11eb-99a7-00807a15b4bd.gif" width="500">

## hana_2021/01/21_2301_memo
[jit.vcr]では固定fpsでレコードできないことがわかった。
調べてみると、入った画像データを書き込むのではなく、入ってきたデータを書き込む？振る舞いの様子
なので見かけ場30fpsで記録しているように見えて、30fpsを担保していない。
結果、フレームが１つコケる毎に（例えば30fpsであれば）33.33333ms分保存されないことがわかった。
解決策としては[jit.record @realtime 1]を利用すること。これでフレームがコケなくなる。
[jit.record]はN x Nルールが必要ないのでリーズナブルである。
Maxで利用するlist型の配列が255までの数字で管理される場合、動画化すると時間管理ができる。
多分これでfix.

## hana_2021/01/21_memo
【[jit.vcr @fps XXX.]でエラーを吐く問題】
* 保存した動画を変換したlistのストリームデータを[jit.fill]に入れるとfpsが保てないのかエラーを吐く
* なんで？？？？

## hana_2021/01/20_memo
jit.vcrが一番優秀。[jit.record] or [jit.recotd @engine viddll]は使えなさそう？
jit.vcrで画像サイズを N x Nだとrecordしてくれる。
なぜか N x 1, N x 2...だとダメで、正方形だと動いてくれた。要検証。
ある程度大きな画像サイズで保存して変換時に必要な配列分だけ取り出せばいい。
（切り取り方は下にある[jit.spill @ listlength N], N=必要な配列の長さで切り取れます。）

コーデックは『prores4444』が一番品質が高く圧縮されない（...はず）
H264で保存したら圧縮されて？値が丸まっていたので注意！！！
jit.recordで『prores4444』や『prores422』でRecordすると
'writer input not ready for media data'と言われて怒られた。
安定してRecordするまでは[print]で監視していたほうがいいかも。

上のエラーだけでなく、飛んでくるStreamデータのfpsがバラつくと
Recordするときにエラーを吐きやすい感じだったので
[jit.fill XXXX]で格納して、別系統で[qmetro FPS]を使って
[jit.matrix XXXX]を叩いたほうが安定した。

[jit.vcr @fps 120.]で実行したらエラーを吐いたので何かあるかも？
80fpsでは安定していたので採用。
飛んでくるメッセージが40fpsだったので倍のフレームレートで記録。
