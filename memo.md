- Excel
	- 1つのシートにデータとロジックの両方が載っている
	- 1つのシートやテーブルにいろいろな種類のデータを入れる
	- 他の箇所で使用中のデータを消せてしまう
	- データが複数の箇所に散らばることがある
	- 同じ箇所を同時に変更したときの競合は考慮しない
	- グラフ作成機能がある
	- フォーム作成機能がある
	- 罫線を使って帳票を作れる

- RDB
	- **データとロジックが分離している**
		- テーブル(データ)には生徒の点数だけが載っており、集計ロジック(SQL)を実行する
		- **ロジックを変えれば、違う集計ができる(様々な集計や検索が切り替えやすい)**
	- 種類ごとにテーブルを分けて、必要に応じて結合することで望みの結果を得る
		- 柔軟な操作や検索ができる
	- 他で使用中のデータを削除できないように制限できる
		- データの整合性を保つことができる
	- データは1箇所のみに保持され、必要時応じてテーブルを連結する
		- 1箇所だけ修正すればよいので、データの修正漏れを防げる
	- 大量のデータを扱える
	- 同時変更による競合を考慮している
		- 変更の競合を検出したり、整合性が取れなくなったら変更をキャンセルする(トランザクション機能)
	- フォーム入力を受け付けたい場合は、Webアプリケーションを作ってそこで入力されたデータをDBに保存する

- 行を識別するのは主キーの値、列を識別するのは列名
	- RDB では縦に伸びる(行が増える)テーブルを扱うのは得意
		- データに応じて自動的に伸びる
	- 横に伸びる(列を増やす)には、テーブルやSQLを書き換える必要がある

- **RDB では行の順番は保持されない(行の集合であって、リストではないから)**
	- **リストは順番を保持するが、集合は順番を保持しない**
		- `A -> B -> C` も `B -> C -> A` も集合では `A,B,C`
	- **リストは要素の重複を許すが、集合は許さない**
		- リストでは `A -> B -> A` で重複は許されるが、集合では `A,B,A` は重複のない `A,B` と同じとみなされる
	- 行の順番は保持しないかわりに、検索の度に並び替える

---
- 値の出力

```sql
select 'Hello' as greeting, 123 as number;

 greeting | number 
----------+--------
 Hello    |    123

select 'hello ' || 'SQL' AS greeting;

+---------+
|greeting |
+---------+
|hello SQL|
+---------+
```

- 順番を安定させたい場合は、 order by で複数条件を指定する
	- **同じ値がない列(ID,コード,注文番号など)と組み合わせる**
	- desc(descending),asc(ascending)は列ごとに指定できる

```sql
-- height を高い順に並べ、身長が同じなら id の小さい順に並べる
select * from members order by height desc, id;

-- 名前順で同じ名前の場合は id の大きい順
select * from members order by name, id desc;
```

- 範囲の選択

```sql
-- 男子のうち背の低い順で2番目と3番目のメンバー
select id, name, height
from members
where gender = 'M'
order by height
limit 2 offset 1;
```

- SQLの実行順序のイメージ
	- from members で検索対象テーブルが決まる
	- where gender = 'M' で条件にあった行のみが選ばれる
	- order by height
	- offset 1, limit 2;
	- select id, name, height
	- 実際の実行順序は order by の前に select が実行されるがイメージのしやすさを優先する
		- select で取り出していない列を使って order by できるため
		```sql
		select id, name from members order by height;
		```

- SQLは「集合への操作」と考えるより「データの加工や変形データの加工」と捉えても良い

```sql
-- 身長の高い順に3名の男子を選ぶ
select height, name
from members
where gender = 'M'
order by height desc, id
limit 3;

-- 1番目に身長が低い
select  id, name, height, gender
from members
order by height, id
limit 1;

-- 2番目に身長が低い
select  id, name, height, gender
from members
order by height, id
limit 1 offset 1;

-- 男女を分けて id 順に並べる
-- gender を M,F の順になるように desc
-- 左から順に評価されるため、 id による並び替えを先に行わない
select *
from members
order by gender desc, id;
```

---

- group by でテーブルの内容をグループ化して集計できる
    - 集約関数
	- **行をグループ化**する

```sql
-- 男女の身長の平均値
select gender, avg(height)
from members
group by gender;

-- to_char() で書式指定ができる
select gender, to_char(avg(height), '999.9') from members group by gender;
 gender | to_char 
--------+---------
 M      |  166.5
 F      |  169.0
(2 rows)
```

- **group by のある SQL では select, order by の対象がグループになる**

```sql
-- グループ化のキーでも集約関数でもない場合はエラーになる
select name, gender
from members
group by gender;

[42803] ERROR: column "members.name" must appear in the GROUP BY clause or be used in an aggregate function Position: 8

-- グループ化のキーに指定すれば問題なし
select name, gender
from members
group by gender, name
order by gender;

-- group by がある場合は order by の指定もグループ化のキーか集約関数の必要がある
select gender
from members
group by gender
order by name;

[42803] ERROR: column "members.name" must appear in the GROUP BY clause or be used in an aggregate function Position: 8
```

- グループ化のキーが主キーの場合は、グループ化のキー以外を select に指定できる
	- 主キーは null かつ 重複していないため、行を特定でき、列の値も特定できるため

```sql
-- 結合時に役に立つテクニックらしい
select gender, name
from members
group by id
order by gender;
```

- avg() のように**グループから複数の値を受け取って計算する関数を集約関数(Aggregate function)という**

```sql
-- count は * で行そのものを指定し、行の数を数えるという意味(null を含む)
-- count(height) は null でない height の数を数える
select gender,
       max(height),
       min(height),
       sum(height),
       count(*),
       to_char(avg(height), '999.99')
from members
group by gender;

-- height が登録されていない件数を数える
select gender, count(*) - count(height) AS height_is_null
from members
group by gender;

-- group by なしで使える集約関数
-- 全部の行が1つのグループにグループ化されて集約関数が実行される
select
       max(height),
       min(height),
       sum(height),
       count(*),
       to_char(avg(height), '999.99') AS avg_height
from members;

+---+---+----+-----+----------+
|max|min|sum |count|avg_height|
+---+---+----+-----+----------+
|175|158|1004|6    | 167.33   |
+---+---+----+-----+----------+
```

---
- having 句で group by によってグループ化したものに対して選択条件を指定できる	
	- **where はテーブルの行を対象とし、条件にあった行だけが選択される**
	- **having はグループを対象とし、条件にあったグループだけが選択される**
		- having は group by の後に実行される

```sql
select gender, avg(height)
from members
group by gender;
+------+-----+
|gender|avg  |
+------+-----+
|M     |166.5|
|F     |169  |
+------+-----+

select gender, avg(height)
from members
group by gender
having avg(height) >= 167;
+------+---+
|gender|avg|
+------+---+
|F     |169|
+------+---+

select gender, count(*)
from members
group by gender;
+------+-----+
|gender|count|
+------+-----+
|M     |4    |
|F     |2    |
+------+-----+

select gender, count(*)
from members
group by gender
having count(*) >= 3;
+------+-----+
|gender|count|
+------+-----+
|M     |4    |
+------+-----+
```

- グループ化のキーに式を使える
	- **列名は式の一種で、 group by には列名か式が指定できる、ではなく(列名を含めた)式が指定できると理解する**

```sql
select length(name), count(*)
from members
group by length(name);

+------+-----+
|length|count|
+------+-----+
|3     |5    |
|4     |1    |
+------+-----+
```

```sql
-- 男女別の人数と平均身長
select gender, count(*), avg(height)
from members
group by gender;

-- 男女別の最高身長、最低身長とその差
-- 1行目を男子、2行目を女子
select gender,max(height), min(height), (max(height) - min(height)) AS max_gap
from members
group by gender
order by gender desc;

-- 男子の人数
-- 男女両方を集計してから女子の集計を捨てている
select count(*) AS man_count
from members
group by gender
having gender = 'M';

-- 女子の人数を集計する必要がないのでより高速
select count(*) AS man_count
from members
where gender = 'M';
```

---
- SQL では基本的に大文字小文字を区別しない仕様

- 式に対して別名を付けられる
	- 列名は式の一部なので、別名が付けられる
		- 参照用の別名とヘッダの別名を変えるには、導出テーブルを使う

```sql
select t.id as "ID", t.n as 名前, t.h as 身長
from (
         select id, name as n, height as h
         from members
         order by h desc, n
     ) as t;
```

- テーブルに対して別名を付けられる
	- それを列名の接頭辞として使用する(別名でなくても接頭辞の使用は可能)
		- SQLのキーワード(予約語)と同じでないことを明示するため
		- 1つのSQLで複数のテーブルを使用する時、どのテーブルの列名か明示するため

```sql
select m.id, m.name, m.height
from members m
where m.gender = 'F';

-- order by は select の後に実行されるので別名が使える(あまりこういう使い方しないが)
select m.id AS 名前, m.name, m.height
from members m
where m.gender = 'F'
order by 名前;
```

---
- パターンマッチ演算子

```sql
select 'Taro' like '%a'; -- false
select 'Taro' like '%o'; -- true
select 'Taro' like 't%'; -- false
select 'Taro' like 'T%'; -- true
select 'Taro' like 'T___'; -- true

select 'Taro' not like '%x'; -- true

-- in 演算子
select 10 not in (2,3,4,5); -- true
```

---
- null

```sql
-- null は等号や比較が使えない
select 0 = 0, 0 > 1, null = null, null > null;

+--------+--------+--------+--------+
|?column?|?column?|?column?|?column?|
+--------+--------+--------+--------+
|true    |false   |NULL    |NULL    |
+--------+--------+--------+--------+

-- null は計算できない
-- NPE は発生しないがすべて null になる
select null + 1, null * 1;

+--------+--------+
|?column?|?column?|
+--------+--------+
|NULL    |NULL    |
+--------+--------+

-- null であることは等号では検索できない
select * from characters where movie_id = null;

 id | movie_id | name | gender 
----+----------+------+--------
(0 rows)

-- is null, is not null を使う
select * from characters where movie_id is null;

+---+--------+----+------+
|id |movie_id|name|gender|
+---+--------+----+------+
|407|NULL    |クラリス|F     |
+---+--------+----+------+

-- null のときに別の値に変換する場合は coalesce() を使用
select coalesce(null, 0); -- 0
select coalesce('a', 'b', 'c'); -- a
select coalesce(null, 'b', 'c'); -- b
select coalesce(null, null, 'c'); -- c
select coalesce(null, false); -- false
```

---
- 複合値(Composite value)
	- 数値や文字列や真偽値など複数を組み合わせてまとめたもの
	- python の tuple のようなもの

```sql
-- 複合値同士で比較される
select (1,2,3) = (2,3,4); -- false
select (1,2,3) < (2,3,4); -- true
select (1,2,3) < (1,3,4); -- true
select (1,2,3) < (1,2,2); -- false
```

```sql
-- 出力結果を加工
select m.id, m.name || 'さん' as name, m.height || 'cm' as height
from members m
where gender = 'F'
order by m.id;

-- 範囲検索
select * from members m
-- where height >= 170 and height <= 180;
where height between 170 and 180;

-- パターンマッチ
select *
from members m
where m.name like '%ン'
order by name;

select * from members m
-- where m.name like '____';
where length(name) = 4;

-- null を未登録に変換
-- 整数型を文字列型に変換してから、未登録に変換
select c.id, coalesce(c.movie_id || '', '未登録')
from characters c
order by id;
```

---
- サブクエリ(Subquery)

```sql
-- 単一列単一行を返すクエリは、単一値(Scalar value)の代わりとして使用可能
-- 国語で最高点を取った生徒の ID と点数を検索
select t.student_id, t.subject, t.score
from test_scores t
where t.subject = '国語'
  and t.score = (
    select max(score)
    from test_scores
    where subject = '国語'
)
order by t.student_id;

-- 複数列単一行を返すクエリは複合値の代わりサブクエリとして使用可能
-- subject = '国語' and score = 80 の代わりに
-- (subject, score) = ('国語', 80) を使用
-- 複合値の代わり以外にも、1行だけのテーブルの代わりに使われることがほとんど
select *
from test_scores t
where (t.subject, t.score) = (
    select subject, max(score)
    from test_scores
    where subject = '国語'
    group by subject
);

-- 単一列複数行を返すクエリは in 演算子で指定できる
-- どれかの教科で100点をとった生徒のIDをすべて表示
-- サブクエリに該当のレコードがなかったら結果も0件(エラーにはならない)
select s.*
from students s
where s.id in (
    select student_id
    from test_scores
    where score = 100
)
order by s.id;

-- 複数列複数行を返すクエリはテーブルの代わりにサブクエリとして使用可能

-- 複数列複数行を返すクエリ
select t.subject, avg(t.score) as avg_score
from test_scores t
group by subject;
+-------+---------+
|subject|avg_score|
+-------+---------+
|理科     |65       |
|国語     |70       |
|社会     |80       |
|算数     |70       |
+-------+---------+

-- 平均点が70点に満たない教科とその平均点
-- サブクエリには alias (x)が必要
-- テーブルのように扱うサブクエリは導出テーブル(Derived table)という
-- テーブルとしての実体はないが、サブクエリによって導出されたテーブルという意味
select *
from (
         select t.subject, avg(t.score) as avg_score
         from test_scores t
         group by subject
     ) x
where x.avg_score < 70;

+-------+---------+
|subject|avg_score|
+-------+---------+
|理科     |65       |
+-------+---------+

-- 今回のケースは having 句でも問題ない
select t.subject, avg(score) as avg_score
from test_scores t
group by t.subject
having avg(t.score) < 70;

-- 複数のサブクエリのあわせ技、国語で最高点を取った生徒をすべて検索
select s.*
from students s
where id in (
    select t.student_id
    from test_scores t
    where t.subject = '国語'
      and score = (
        select max(score)
        from test_scores t
        where subject = '国語'
    )
);
```

---
- exists 演算子は条件に合う行があるかを判定
	- 条件に合う行があるかを調べるケースはかなりある
		- ex.) この顧客は過去に商品を購入したことがあるか、一ヶ月以内にログインしたか など
	- true/false となる演算子や関数は述語(Predicate)と呼ばれる

```sql
-- exsits だと1行でも見つかれば、そこで検索を終了して true を返すため、すべての行を調べる必要がないため高速
-- exsits はサブクエリが返す行の中身を見ていないため、
-- サブクエリの select に中身を使っていないことの慣習として 1 を入れることも
select exists(select 1 from test_scores where subject = '社会' and score = 100); -- true
```

- all 演算子は、サブクエリが返した値が全て条件に合っているかを確認

```sql
select 40 <= all (select score from test_scores where subject = '算数');
```

- any 演算子は条件に合うものがサブクエリ内にあるか調べる

```sql
-- 1人もいなければ false
select 100 = any (select score from test_scores where subject = '算数');

-- exsists を使って書く場合
select exists(select 1 from test_scores where subject = '算数' and score = 100);
```

- **with 句(Common Table Expression,CTE) を使うと、サブクエリに別名を付けられる**

```sql
with avg as (
    select subject, avg(score) as avg_score
    from test_scores
    group by subject
)

select avg.subject, avg.avg_score
from avg
where avg.avg_score < 70;

-- 列名にも別名をつけることで、with 句の先頭行を見るだけで列名がわかる
with avg(subject, avg_score) as (
    select subject, avg(score)
    from test_scores
    group by subject
)

select avg.subject, avg.avg_score
from avg
where avg.avg_score < 70;

-- 国語で最高点を取った生徒のIDと点数を検索
with x(max_score) as (
    select max(score)
    from test_scores
    where subject = '国語'
)

select t.student_id, t.score
from test_scores t
where t.subject = '国語'
  -- with はサブクエリに別名をつける機能なので、 t.score = x.max_score のように代入は不可
  -- サブクエリの結果を変数のように扱えるわけではない
  and t.score = (select x.max_score from x)
order by t.student_id;
```

---
- with 句を使ったリファクタリング
	- サブクエリに適切な名前がついていて理解しやすい
	- SQL が入れ子になっていないので理解しやすい

```sql
-- , で区切って複数のサブクエリに alias をつけられる
-- 前のサブクエリの結果を、後ろのサブクエリで利用できる
with max_score(value) as (
    select max(score)
    from test_scores
    where subject = '国語'
)
   , targets(student_id) as (
    select student_id
    from test_scores
    where subject = '国語'
      and score = (select value from max_score)
)

select *
from students
where id in (select student_id from targets)
order by id;
```

---
- サブクエリを一時テーブルとみなす
	- **with 句はサブクエリの結果を一時テーブルに格納する**
	- これを**実体化(Materialize)**という

```sql
-- サブクエリを一時テーブル(temporary table)とみなす
-- 1つの SQL 内で同じサブクエリを2回以上使うときに便利

-- with を使わない場合は同じサブクエリを2回書くことに
-- select subject, avg_score
-- from (
--          select subject, avg(t.score) as avg_score
--          from test_scores t
--          group by t.subject
--      ) avg_scores
-- where avg_score = (
--     select min(x.avg_score)
--     from (
--              select subject, avg(score) as avg_score
--              from test_scores
--              group by subject
--          ) x
-- );

with avg_scores(subject, avg_score) as (
    select subject, avg(score)
    from test_scores
    group by subject
)
select subject, avg_score
from avg_scores
where avg_score = (
    select min(x.avg_score)
    from avg_scores x
)
```

- with 句のサブクエリはサブクエリ外にある条件式に影響を受けない
	- with 句のサブクエリが最適化されることはない
	- それにより実行速度が遅くなる場合もある
		- **有り無しで試して速度に差が出る場合は無しにする**

- with を使用しないサブクエリは最適化が行われる
	- DBエンジンが最適化(Optimization)を行う

```sql
-- 2. サブクエリの結果をそのまま出力していることになるので、サブクエリにしないよう最適化される
-- select t.student_id, t.max_score from (
    slect student_id, max(score) as max_score
    from test_scores
    where student_id = 201;
    group by student_id
-- ) t;
-- 1. サブクエリ外の条件式がサブクエリの最適化に使われる
-- where student_id = 201;
```

---
- 結合

```sql
-- from 句に複数のテーブルを指定すると、すべての行の組み合わせが得られる
-- movie table, characters table の行数を掛け算した組み合わせの件数が出力される
select movies.*, characters.* from movies, characters;

-- すべての組み合わせから movie_id が一致する組み合わせだけを選ぶと、映画とキャラクターの正しい組み合わせが得られる
-- これが SQL での結合
-- 複数のテーブルを結合する際は、テーブル名に短い別名をつけると読み書きしやすい
select m.*, c.*
from movies m,
     characters c
where m.movie_id = c.movie_id;
```

- SQL での結合は二つの操作に分けられる
    - 1. すべての組み合わせを生成
    - 2. 条件に合った組み合わせだけを選ぶ

```sql
-- 映画テーブルとキャラクターテーブルを結合し、女性キャラクターのみ選ぶ
-- where 句にテーブルの結合条件と行の選択条件が入っている(内部の仕組みとしてはどちらも行の選択)
select m.*, c.*
from movies m,
     characters c
where m.movie_id = c.movie_id
  and c.gender = 'F';

-- where 句の複数の条件の中からテーブルの結合条件を探すのは面倒
-- テーブルの結合条件を where 句から分離できる join 演算子がある
-- from と where に散りばめられていた、テーブル結合の指定が join 演算子にまとめられた
-- join は演算子なので(句ではない、 +, and の仲間)、 from A join B は A join B の結果を from 句に指定している
select *
from movies m
         join characters c -- 結合するテーブル
              on m.movie_id = c.movie_id -- テーブルの結合条件を指定
where c.gender = 'F';
```

- 主役となるテーブルを from 句、補助となるテーブルを join 演算子に指定する
    - どちらを from 句に指定しても結果は変わらないが、テーブルの役割をわかりやすくするため

```sql
-- 映画の一覧を、キャラクター名付きで表示する場合
-- 映画の一覧が主役で、キャラクター名はその補助
select m.*, c.name, c.gender
from movies m -- 主役となるテーブルを指定
         join characters c on m.movie_id = c.movie_id
order by m.movie_id, c.id;

-- キャラクターの一覧
select c.*, m.*
from characters c
         join movies m on c.movie_id = m.movie_id
where m.title = 'となりのトトロ' -- 結合することで別のテーブル(movies)を検索条件として利用できる
order by c.id;
```

- サブクエリでも検索時に別テーブルを利用できる、書き換えはできるようにしておく

```sql
select c.*
from characters c
where c.movie_id in (
    select m.movie_id
    from movies m
    where m.title = 'となりのトトロ'
)
```

- using を使用すると join の on を書き換え可能
    - 同じ名前の列同士でしか結合できない
    - 複数の列名を使って結合する際は using を使用するほうが簡潔に書けるが、複数列名で結合するかは複合主キーを使うかによる

```sql
select c.*, m.*
from characters c
         join movies m
              -- on c.movie_id = m.movie_id
              using (movie_id)
order by c.id;
```
---

- inner join だと、テーブルには存在するが、結合すると表示されない行がある
    - すべての組み合わせを生成し、条件に合った組み合わせだけを選ぶのが inner join による結合のため
        - 結合する双方のテーブルにレコードがないと条件に一致しない

```sql
select m.*, c.*
from movies m
         join characters c using (movie_id)
order by m.movie_id, c.id;
```

- left join により対応する行に加えて、対応しない行を左側のテーブルから取り出す
    - キャラクターが登録されていない映画でも一覧に表示したい

```sql
select m.*, c.*
from movies m
         left join characters c using (movie_id)
order by m.movie_id, c.id;

-- 登録されていないキャラクターの列は null
/*
 movie_id |      title       | id  | movie_id |   name   | gender
----------+------------------+-----+----------+----------+--------
       93 | 風の谷のナウシカ | 401 |       93 | ナウシカ | F
       94 | 天空の城ラピュタ | 402 |       94 | パズー   | M
       94 | 天空の城ラピュタ | 403 |       94 | シータ   | F
       94 | 天空の城ラピュタ | 404 |       94 | ムスカ   | M
       95 | となりのトトロ   | 405 |       95 | さつき   | F
       95 | となりのトトロ   | 406 |       95 | メイ     | F
       96 | 崖の上のポニョ   |     |          |          |
(7 rows)
*/
```

- right join により対応する行に加えて、対応しない行を右側のテーブルから取り出す
    - キャラクターの一覧を映画付きで表示(キャラクターが登録されていない映画も表示)

```sql
select c.*, m.title
from characters c
         right join movies m using (movie_id)
order by c.id;

/*
 id  | movie_id |   name   | gender |      title
-----+----------+----------+--------+------------------
 401 |       93 | ナウシカ | F      | 風の谷のナウシカ
 402 |       94 | パズー   | M      | 天空の城ラピュタ
 403 |       94 | シータ   | F      | 天空の城ラピュタ
 404 |       94 | ムスカ   | M      | 天空の城ラピュタ
 405 |       95 | さつき   | F      | となりのトトロ
 406 |       95 | メイ     | F      | となりのトトロ
     |          |          |        | 崖の上のポニョ
(7 rows)
*/
```

- 内部結合を使ったフィルター
    - キャラクターが登録されている映画だけ選ぶ、映画が登録されているキャラクターを選ぶ、など

```sql
-- character table と結合するが、キャラクターは表示していない(キャラクターのある映画のみを表示)
select m.*
from movies m
         join characters c using (movie_id)
group by m.movie_id, m.title
order by m.movie_id;

/*
 movie_id |      title
----------+------------------
       93 | 風の谷のナウシカ
       94 | 天空の城ラピュタ
       95 | となりのトトロ
(3 rows)
*/
```

- 外部結合によるフィルター
    - 対応するものがない行だけを選べる = 対応するものがある行を取り除ける
        - 出演キャラクターのない映画だけ選ぶ

```sql
select m.*
from movies m
         left join characters c using (movie_id)
where c.id is null
order by m.movie_id, c.id;

/*
 movie_id |     title
----------+----------------
       96 | 崖の上のポニョ
(1 row)
*/
```

```sql
-- 出演キャラクターのいない映画の一覧
select m.movie_id, m.title
from movies m
         left join characters c on m.movie_id = c.movie_id
where c.id is null
order by m.movie_id, c.id;

-- 出演作が登録されていないキャラクター
-- movie の情報を出さないので結合は不要
-- select c.id, c.name, c.gender
-- from characters c
--          left join movies m on m.movie_id = c.movie_id
-- where c.movie_id is null
-- order by c.id, m.movie_id;

select c.id, c.name, c.gender
from characters c
where c.movie_id is null
order by c.id;

-- 映画ごとの出演キャラクター数
select m.movie_id, m.title, count(c.id)
from movies m
         left join characters c on m.movie_id = c.movie_id
group by m.movie_id -- 主キーでグループ化しているため、グループ化のキーで無い列名を select で指定可能(m.title)
order by m.movie_id;
```

---
- 主キーとは、あるテーブルに置いて、行を特定するための列のこと
    - null ではなく、重複せず、値が変わらないことが特徴

- **Foregin key(外部キー) は他テーブルの主キーを参照している列のこと**
    - 参照先のテーブルにない値を外部キーに入れようとした時、すでに外から参照されている主キーを変更しようとした時エラーになる(外部キー制約)

```sql
/*
create table characters (
  id          integer    primary key
, movie_id    integer    references movies(movie_id)
, name        text       not null
, gender      char(1)    not null
);
*/

-- テーブル定義の確認
/*
test=# \d characters
                Table "public.characters"
  Column  |     Type     | Collation | Nullable | Default
----------+--------------+-----------+----------+---------
 id       | integer      |           | not null |
 movie_id | integer      |           |          |
 name     | text         |           | not null |
 gender   | character(1) |           | not null |
Indexes:
    "characters_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "characters_movie_id_fkey" FOREIGN KEY (movie_id) REFERENCES movies(movie_id)
*/
```
---

- 1:N の関係では、 「N」の側のテーブルから「1」の側のテーブルを参照する
    - **「N」側に外部キーを作成し、「1」側のテーブルの主キーを参照する**
    - characters テーブルの外部キーで movies テーブルの主キーである movie_id を指定
    - もし参照方向が逆だったら、movie テーブルの外部キーが character_id を複数保持することになり、1つのフィールドに複数の値を埋め込むことになる
        - 第1次正規化に反する??

```sql
select e.id,
       e.name,
       c.id   AS comp_id,
       c.name AS company,
       d.id   AS dept_id,
       d.name AS department
from employees e
         join departments d on e.dept_id = d.id
         join companies c on d.company_id = c.id
order by e.id, c.id;

+-----+----+-------+-------+-------+----------+
|id   |name|comp_id|company|dept_id|department|
+-----+----+-------+-------+-------+----------+
|10001|社員1 |301    |○○会社   |2001   |開発部       |
|10002|社員2 |301    |○○会社   |2001   |開発部       |
|10003|社員3 |301    |○○会社   |2001   |開発部       |
+-----+----+-------+-------+-------+----------+
```

- 交差テーブル(Intersection table)は多対多の関連をそのまま保存した中間テーブル

```sql
select * from writings;

+-------+---------+----+
|book_id|author_id|role|
+-------+---------+----+
|31     |71       |原作  |
|31     |72       |漫画  |
|32     |71       |原作  |
|33     |73       |原作  |
+-------+---------+----+

select b.id AS book_id,
       b.title,
       w.author_id,
       a.name,
       w.role
from books b
         join writings w on b.id = w.book_id
         join authors a on a.id = w.author_id
order by b.id;
```
---

- インデックスを言い換えると、検索に必要な情報だけをあらかじめテーブルから抜き出したデータのこと

- インデックスがない場合、データベースエンジンはテーブルを先頭行から順に読み取って調べる(シーケンシャルスキャン)

- 検索対象にインデックスがある場合、走査対象がテーブルではなくインデックスになる
    - インデックス内で見つかれば、そこからテーブルの該当の行だけを読み込む
    - 見つからなければ、テーブルの行は一切読み込まない
    - **テーブルの中から条件に合う、ごく少数の行を取り出す場合に、効果的な走査方法**
    - 逆に、テーブルの全部や大部分を使って集計する場合には向いていない

```sql
\d members

/*
                 Table "public.members"
 Column |     Type     | Collation | Nullable | Default 
--------+--------------+-----------+----------+---------
 id     | integer      |           | not null | 
 name   | text         |           | not null | 
 height | integer      |           | not null | 
 gender | character(1) |           | not null | 
Indexes:
    "members_pkey" PRIMARY KEY, btree (id)
*/

# \d members

create index members_name_idx on members(name);
/*
Indexes:
    "members_pkey" PRIMARY KEY, btree (id)
    "members_name_idx" btree (name)
*/

drop index members_name_idx;
```

- テストデータを100万行用意して、 index の効果を確認

```sql
create table testmembers
(
    id     serial primary key,
    name   text    not null,
    height integer not null,
    gender char(1) not null
);

-- member#1,member#2,member#3 となるように || で文字列を結合
-- select * from generate_series(1, 3);
/*
+---------------+
|generate_series|
+---------------+
|1              |
|2              |
|3              |
+---------------+
*/
-- select n from generate_series(1, 3) x(n);
insert into testmembers(name, height, gender)
select 'member#' || x.n,
       160 + floor(random() * 20)::integer,
       case when x.n % 2 = 0 then 'M' else 'F' end
from generate_series(1, 1000000) x(n);

-- 実行時間を表示
# \timing on
-- Timing is on.

select * from testmembers where name = 'member#1000000';
-- Time: 176.01ms

-- インデックススキャンをすると高速化する
create index testmembers_name_idx on testmembers(name);
select * from testmembers where name = 'member#1000000';
-- Time: 2.519 ms
```

- インデックスが効いているかの確認

```sql
explain select * from testmembers where name = 'member#1000000';
/*
                                       QUERY PLAN                                        
-----------------------------------------------------------------------------------------
 Index Scan using testmembers_name_idx on testmembers  (cost=0.42..8.44 rows=1 width=23)
*/

-- = では index を使うが、 <> では使わない
explain select * from testmembers where name <> 'member#1000000';
/*
                              QUERY PLAN                              
----------------------------------------------------------------------
 Seq Scan on testmembers  (cost=0.00..19844.12 rows=1000009 width=23)
*/
```

- **カーディナリティ(Cardinality) とは、ある列に入っている値の種類の多さのこと**
    - **id,name の列は値に重複がないため、行の数だけ値がありカーディナリティが高い**
    - **gender の列は値が2種類しかなくカーディナリティが低い**

- **index の効果が大きいのは、値によって行を大きく絞り込めるため、カーディナリティが高い方**
    - **gender に index を作成してもカーディナリティが低い(値が2つしか無い)から行を大きく絞り込めない**

- 主キーと一意制約が設定された列は重複がなく、自動的に index が作成される
    - 厳密には主キー制約、一意制約は index により実現されている

- **主キーや一意制約付きの列を使って行を絞り込むと index が効いて高速になる**

- テーブルの行数が少ない場合は、インデックスは使われない(全て読み込んだほうが早いため)

```shell
# 先程の 100万件のデータのテーブルの容量は 58M
$ ls -lh /var/lib/postgresql/data/base/16384/16549
-rw------- 1 postgres postgres 58M Aug 22 23:52 /var/lib/postgresql/data/base/16384/16549
```

- 確かめ方、DB, table の実体の確認([参考](https://gist.github.com/Epictetus/3386858))

```sql
select datname, pg_size_pretty(pg_database_size(datname))
from pg_database
where datname = 'test';

/*
+--------+--------------+
|datname |pg_size_pretty|
+--------+--------------+
|test|117 MB        |
+--------+--------------+
*/

select datid, datname
from pg_stat_database
where datname = 'test';
/*
+-----+--------+
|datid|datname |
+-----+--------+
|16384|test|
+-----+--------+
*/

select relid, relname from pg_stat_all_tables
where relname = 'testmembers';
/*
+-----+-----------+
|relid|relname    |
+-----+-----------+
|16549|testmembers|
+-----+-----------+
*/
```

```shell
# 実体としての DB ごとの容量
$ du -sh /var/lib/postgresql/data/base/* | sort -r
7.8M    /var/lib/postgresql/data/base/1
7.6M    /var/lib/postgresql/data/base/13067
7.4M    /var/lib/postgresql/data/base/13066
120M    /var/lib/postgresql/data/base/16384
0       /var/lib/postgresql/data/base/pgsql_tmp

# 実体としての table ごとの容量
$ du -sm /var/lib/postgresql/data/base/16384/* | sort -r | head -n 5
59      /var/lib/postgresql/data/base/16384/16549
32      /var/lib/postgresql/data/base/16384/16558
23      /var/lib/postgresql/data/base/16384/16556
1       /var/lib/postgresql/data/base/16384/PG_VERSION
1       /var/lib/postgresql/data/base/16384/pg_internal.init
```

- 実体ファイルから逆引きで他に容量の大きなものの確認
    - table の pkey, index の情報だった

```sql
select relname, relfilenode
from pg_class
where relfilenode in ('16558', '16556');

/*
+--------------------+-----------+
|relname             |relfilenode|
+--------------------+-----------+
|testmembers_pkey    |16556      |
|testmembers_name_idx|16558      |
+--------------------+-----------+
*/
```

- 比較式に関数や演算子を使っていると index は使われない
    - `where lower(name) = 'testname'` のような場合
    - **index 作成時に `lower(name)` という式を指定していれば上記のケースでも index が使える = 式インデックス(Expression index)**

- データ型が一致していないと index は使われない
    - integer 型で index が作られている場合、 `where id = round()` では index が使われない
    - データ型を調べる

```sql
select round(3.14); -- 3
select pg_typeof(round(3.14));
/*
+---------+
|pg_typeof|
+---------+
|numeric  |
+---------+
*/
```

- 複合インデックス(composite index)は複数の列から構成される index
    - 複合主キーのようなイメージ
    - 最初の列をつかわないと動作しない

- index は select が高速になる代わりに、 insert, update, delete 時にテーブルだけでなく、 index の更新も必要となりそれらが遅くなる
    - 必要でない index はつけない
    - システム移行時などの、大量データ挿入時は index を削除してから insert し、終わったら改めて index を作り直す
    - create index はテーブルをロックする
        - `create index concurrently` でバックグラウンドで作成できるためロックを回避できる

- 部分インデックス(Partial index)
    - 列によっては偏りがでる場合、少ない方の値に限り有効な index
    - 退会したユーザを表す列でほとんどのユーザは null になっている場合

```sql
create index app_user_deleted_at_idx on app_users(deleted_at) where deleted_at in not null;

-- partial index が使われる
select count(*) from app_users where deleted_at is not null;
```

- index only scan
    - テーブルから行を読み込むことなく index の読み込みだけで完結することを指す

```sql
explain select * from testmembers where name = 'member#1000000';
--  Index Scan using testmembers_name_idx on testmembers  (cost=0.42..8.44 rows=1 width=23)

explain select name from testmembers where name = 'member#1000000';
--  Index Only Scan using testmembers_name_idx on testmembers  (cost=0.42..8.44 rows=1 width=13)
```
