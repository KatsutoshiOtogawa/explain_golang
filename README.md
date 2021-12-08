# md描くようの説明

mdからpowerpointの作成。

```shell
pandoc explain.md -o explain.pptx
```


CREATE INDEX datetime_ix  on spz_11(datetime)


sqlite> drop index datetime_ix
   ...> ;
Run Time: real 21.716 user 1.918825 sys 4.394432
sqlite> explain query plan select count(ROWID) from spz_11;
QUERY PLAN
`--SCAN TABLE spz_11
Run Time: real 0.000 user 0.000081 sys 0.000043
sqlite> select count(*) from spz_11;

まず、sql自体はDB側で解析されて
帰ってくる結果は件数だけなので、
ネットワークのIOの差は環境は関係ないと思います。
(あるとするならDBのディスクIOの方。)

と言うことで
インデックスある方
sqlite> explain query plan select count(*) from spz_11;
QUERY PLAN
`--SCAN TABLE spz_11 USING COVERING INDEX datetime_ix
Run Time: real 0.000 user 0.000044 sys 0.000009

インデックス無い方

ふむふむ、全件検索なのでdatetimeにしか指定されていないindexが使われていますね？
このせいで遅くなっているのでは？


一応確認のためとりあえず実行してみると
インデックスある方
sqlite> select count(*) from spz_11;
91491840
Run Time: real 195.668 user 2.300456 sys 37.383349

インデックス無い方
sqlite> select count(*) from spz_11;
91491840
Run Time: real 17.317 user 1.014329 sys 3.629020

私の方も10倍ほど差がでるので、問題はアルゴリズムとか計算量の方っぽいですね。

-- group by で指定
sqlite> explain query plan select count(*) from spz_11 group by datetime, spz;
QUERY PLAN
`--SCAN TABLE spz_11 USING COVERING INDEX spz_11_ix
Run Time: real 0.001 user 0.000091 sys 0.000022


これは否定