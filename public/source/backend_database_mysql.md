# SQL

## 01. データベース

### MySQLの準備

#### ・CentOSにインストール

```shell
# mysqlコマンドのみをインストールしたい場合はこちら
$ dnf install -y mysql
```

```shell
# mysqlコマンド，データベースサーバ機能，をインストールしたい場合はこちら
$ dnf install -y mysql-server
```

<br>

### パラメータ

#### ・パラメータの表示

データベースに登録されているグローバルパラメータとセッションパラメータを表示する．

```sql
-- セッション／グローバルパラメータを表示
SHOW SESSION VARIABLES;
SHOW GLOBAL VARIABLES;

-- OSとDBのタイムゾーンに関するパラメータを表示
SHOW SESSION VARIABLES LIKE "%time_zone";
SHOW GLOBAL VARIABLES LIKE "%time_zone";
```

#### ・パラメータの設定

```sql
-- グローバルパラメータの場合
SET GLOBAL time_zone = "Asia/Tokyo";

-- セッションパラメータの場合
SET time_zone = "Asia/Tokyo";
```

<br>

## 02. テーブル

### ```CREATE TABLE```句

#### ・使い方

**＊実装例＊**

```sql
-- 注文テーブル作成
CREATE TABLE order_data (

    -- プライマリキー制約
    order_id INT(10) PRIMARY KEY COMMENT "注文ID",

    -- Not Null制約
    order_kbn INT(3) NOT NULL COMMENT "注文区分",
    system_create_date_time DATETIME NOT NULL COMMENT "システム登録日時",
    system_update_date_time DATETIME NOT NULL COMMENT "システム更新日時",
    delete_flg INT(1) DEFAULT 0 NOT NULL COMMENT "0：通常，1：削除済",
  
    -- 複合プライマリキー制約（これを指定する場合，上記のプライマリキー制約の記述は不要）
    PRIMARY KEY(order_id, order_kbn)
  
    -- 参照制約キー
    FOREIGN KEY order_kbn REFERENCES order_kbn_data
)
```

<br>

### ```CREATE VIEW```句

#### ・使い方

ビューとはある表の特定のカラムや指定した条件に合致するレコードなどを取り出した仮想の表．また，複数の表を結合したビューを作成できる．ビューを作成することによりユーザに必要最小限のカラムやレコードのみにアクセスさせる事ができ，また結合条件を指定しなくても既に結合された表にアクセスできる．
⇒よくわからん…

**＊実装例＊**

```sql
CREATE VIEW { テーブル名 } AS
SELECT
    *
FROM
    { テーブル名 };
```

<br>

### 制約

#### ・プライマリキー制約と複合プライマリキー制約

テーブルの中で，レコードを一意に特定できる値を『プライマリキー』の値と呼ぶ．

![主キー](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/主キー.jpg)

プライマリキーは複数設定することができ，複合プライマリキーの場合，片方のフィールドの値が異なれば，異なるプライマリキーとして見なされる．以下のように，ユーザIDと期間開始日付を複合プライマリキーとすると，一人のユーザが複数の期間をもつ場合に対応できる．

| *user_id* | *period_start_date* | period_end_date | fee_yen |
| --------- | ------------------- | --------------- | ------- |
| *1*       | *2019-04-03*        | 2019-05-03      | 200     |
| *1*       | *2019-10-07*        | 2019-11-07      | 400     |
| *2*       | *2019-10-11*        | 2019-11-11      | 200     |

#### ・Not Null制約

レコードに挿入される値のデータ型を指定しておくことによって，データ型不一致やNullのための例外処理を実装しなくてもよくなる．

#### ・Foreign key（外部キー）と参照制約

複数のテーブルを関連付けるために用いられるカラムのことをForeign key（外部キー）という．外部キーの参照先のテーブルには，外部キーの値と同じ値のカラムが存在していなければならない（参照制約）．参照制約を行うと，以下の2つが起こる．

親テーブルに存在しない値は，子テーブルに登録できない．

親テーブルで参照される値は，子テーブルからは削除できない．

![外部キー](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/外部キー.png)

<br>

### stored procedure

#### ・stored procedureとは

あらかじめ一連のSQL文をデータベースに格納しておき，Call文で呼び出す方式．

![p325](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/p325.gif)

#### ・使い方

**＊実装例＊**

```SELECT```文のstored procedureを作成するとする．

```sql
-- PROCEDUREを作成し，データベースへ格納しておく．
CREATE PROCEDURE SelectContact AS
SELECT
    { カラム名 }
FROM
    { テーブル名 }
```

```sql
-- PROCEDUREを実行
EXEC SelectContact
```

<br>

### エクスポート，インポート

#### ・テーブルのエクスポート

DBからテーブルをエクスポートする．エクスポートしたいテーブルの数だけ，テーブル名を連ねる

```shell
$ mysqldump --force -u "{ アカウント }" -p -h "{ DBのホスト }" "{ DB名 }" "{ テーブル名1 }" "{ テーブル名2 }" > table.sql
```

#### ・テーブルのインポート

DBにテーブルをインポートする．forceオプションで，エラーが出ても強制的にインポート．

```shell
 $ mysql --force -u "{ アカウント }" -p -h "{ DBのホスト }" "{ DB名 }" < table.sql
```

<br>

### データ型

#### ・数値型

整数値がどのくらい増えるかによって，3つを使い分ければ良い．

| データ型 | 値                                              | 負の値を許可しない時（UNSIGNED） |
| -------- | ----------------------------------------------- | -------------------------------- |
| TINYINT  | -128<br>~ +127                                 | 0<br>~ +255                     |
| INT      | 2147483648<br>~ +2147483647                    | 0<br>~ +4294967295              |
| BIGINT   | -9223372036854775808<br>~ +9223372036854775807 | 0<br>~ +18446744073709551615     |

 #### ・文字列型

文字数がどのくらい増えるかによって，3つを使い分ければ良い．

| データ型   | 最大バイト数 |
| ---------- | ------------ |
| VARCHAR(M) | 255          |
| TEXT       | 65535        |
| MEDIUMTEXT | 16777215     |

<br>

## 03. ユーザの管理

### CREATE

#### ・ユーザ作成

```sql
CREATE USER "{ ユーザ名 }" IDENTIFIED BY "{ パスワード }";
```

#### ・ユーザ一覧

ここで表示される特権と．ALL特権は異なる．

```sql
SELECT
    *
FROM
    mysql.user;
```

<br>

### DROP

#### ・ユーザ削除

```sql
-- ユーザ別のホスト名の確認
SELECT * FROM mysql.user;

-- ホストが「%」だった場合
DROP USER { ユーザ名 }@`%`;
```

<br>

### GRANT

#### ・全ての操作権限を付与


データベース名は，シングルクオーテーションで囲う必要が無い．全権限を付与する場合，```PRIVILEGES```は省略できるが，厳密には省略しないようほうがよい．


```sql
-- 全てのデータベースに関する権限を付与
GRANT ALL PRIVILEGES ON *.* TO "{ ユーザ名 }";

-- Amazon AuroraまたはRDSの場合はこちら
GRANT ALL PRIVILEGES ON `%`.* TO "{ ユーザー名 }";
```

```sql
-- Amazon Auroraも同じく
-- 特定のデータベースに関する全権限を付与
GRANT ALL PRIVILEGES ON {DB名}.* TO "{ ユーザ名 }";
```

#### ・一部の操作権限を付与

特定のデータベースに関する読み出し権限のみ付与する．

```sql
GRANT SELECT ON {DB名}.* TO "{ ユーザ名 }";
```

#### ・権限の振り方

| ユーザの種類             | 権限                                                       |
| ------------------------ | ---------------------------------------------------------- |
| admin                    |                                                            |
| アプリケーション         | ```GRANT ALL PRIVILEGES ON {DB名}.* TO '{ ユーザー名 }'``` |
| 読み出し／書き込みユーザ | ```GRANT ALL PRIVILEGES ON {DB名}.* TO '{ ユーザー名 }'``` |
| 読み出しユーザ           | ```GRANT SELECT ON {DB名}.* TO '{ ユーザ名 }';```          |

<br>

#### ・ユーザ権限一覧

ユーザに付与されている権限を表示する．

```sql
SHOW GRANTS FOR "{ ユーザ名 }";
```

作成しただけで権限を何も付与してないユーザの場合，「データベースサーバ内の全データベースに関して，全権限なし」を表す```USAGE```として表示される．

```sql
GRANT USAGE ON *.* TO "{ ユーザー名 }";
```

特定のデータベースの操作権限を与えると，上記に加えて，付与したGRANT権限も表示されるようになる．

<br>

### REVOKE

#### ・全権限削除

全権限を削除し，GRANT権限をUSAGEに戻す．

```sql
-- Amazon AuroraまたはRDSの場合
REVOKE ALL PRIVILEGES ON `%`.*
FROM
    "{ ユーザ名 }";

REVOKE ALL PRIVILEGES ON { DB名 }.*
FROM
    "{ ユーザ名 }";
```

#### ・ユーザ名変更

```sql
RENAME USER "{ 古いユーザ名 }" TO "{ 新しいユーザ名 }";
```

<br>

## 04. レコードの読み出し：READ


### はじめに

#### ・句の処理の順番

```
FROM ---> JOIN ---> WHERE ---> GROUP BY ---> HAVING ---> SELECT ---> ORDER BY
```

<br>

### ```SELECT```句

#### ・```SUM()```

指定したカラムで，『フィールド』の合計を取得

```sql
SELECT
    SUM({ カラム名 })
FROM
    { テーブル名 };
```

#### ・```AVG()```

指定したカラムで，『フィールド』の平均値を取得

```sql
SELECT
    AVG({ カラム名 })
FROM
    { テーブル名 };
```

#### ・```MIN()```

指定したカラムで，『フィールド』の最小値を取得

```sql
SELECT
    MIN({ カラム名 })
FROM
    { テーブル名 };
```

#### ・```MAX()```

指定したカラムで，『フィールド』の最大値を取得

```sql
SELECT
    MAX({ カラム名 })
FROM
    { テーブル名 };
```

#### ・```COUNT()```

指定したカラムで，『フィールド』の個数を取得

```sql
SELECT
    { カラム名 } COUNT(*)
FROM
    { テーブル名 };
```

**※消去法の小技：集合関数を入れ子状にはできない**

**＊実装例＊**

集合関数を集合関数の中に入れ子状にすることはできない．

```sql
-- 
SELECT
    AVG(SUM({ カラム名 }))
FROM
    { テーブル名 };
```

指定したカラムで，値無しも含む『フィールド』を取得

```sql
SELECT
    { カラム名 } COUNT(*)
FROM
    { テーブル名 };
```
指定したカラムで，値無しを除いた『フィールド』を取得
```sql
SELECT
    { カラム名 } COUNT(*);
```

#### ・```LAST_INSERT_ID()```

最後に挿入されたIDを読み出す．

```sql
SELECT LAST_INSERT_ID();
```

#### ・```MD5()```

文字列をハッシュ化

```sql
SELECT MD5("xxxxx");
```

<br>

### ```CASE```句

 カラム1が```true```だったら，カラム2を取得する．```false```であったら，カラム3を取得する．

```sql
SELECT
    CASE
        WHEN { エイリアス }.{ カラム名1 } = 1 THEN { エイリアス }.{ カラム名2 }
        ELSE { エイリアス }.{ カラム名3 }
    END AS name
FROM
    { テーブル名 } AS { エイリアス };
```

<br>

### ```FROM```句

#### ・```JOIN```句の種類

![内部結合のベン図](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/内部結合のベン図.jpg)

#### ・```LEFT JOIN```（左外部結合）

『users』テーブルと『items』テーブルの商品IDが一致しているデータと，元となる『users』テーブルにしか存在しないデータが，セットで取得される．

![LEFT_JOIN](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/LEFT_JOIN.png)

<br>

#### ・```INNER JOIN```（内部結合）

基本情報技術者試験では，内部結合（A∩B）しか出題されない．

#### ・内部結合に```WHERE```を用いる場合

2つの```WHERE```文が，```AND```で結びつけられている時，まず一つ目の```WHERE```を満たすレコードを取得した後，取得したレコードの中から，二つ目の```WHERE```を満たすレコードを取得する．

**＊実装例＊**

```sql
-- 『カラム』だけでなく，どの『表』なの物なのかも指定
SELECT
    { テーブル名1 }.{ カラム名1 },
    -- 複数の表を指定
FROM
    { テーブル名1 },
    { テーブル名2 },
    -- まず，1つ目のフィールドと2つ目のフィールドが同じレコードを取得．
WHERE
    -- 次に，上記で取得したレコードのうち，次の条件も満たすレコードのみを取得．
    { レコード名1 } = { レコード名2 }
    AND { レコード名2 } = { レコード名3 }
```

#### ・内部結合に```INNER JOIN ON```を用いる場合

**＊実装例＊**

```sql
-- 『カラム』だけでなく，どの『表』なの物なのかも指定
SELECT
    { テーブル名1 }.{ カラム名1 },
    -- 複数の表を指定
FROM
    { テーブル名1 }
    -- 2つ目の表の『レコード』と照合
    INNER JOIN { テーブル名2 }
    ON { テーブル名1 }.{ カラム名1 } = { テーブル名2 }.{ カラム名2 }
    -- 3つ目の表の『レコード』と照合
    INNER JOIN { テーブル名3 }
    ON { テーブル名1 }.{ カラム名1 } = { テーブル名3 }.{ カラム名3 }
```

<br>

### ```ORDER BY```句

#### ・使い方

**＊実装例＊**


```php
<?php
$joinedIdList = implode(",", $idList);

// 並び替え条件を設定
$expression = call_user_func(function () use ($orders, $joinedIdList) {
    if ($orders) {
        foreach ($orders as $key => $order) {
            switch ($key) {
                case "id":
                    return sprintf("ss.id %s", $order);
            }
        }
    }
    
    // IN句順の場合
    return sprintf("FIELD(ss.id, %s)", $idList);
});

$sql = <<<SQL
            SELECT 
                name
            FROM
                table
            ORDER BY {$expression}
        SQL;
```

<br>

### ```IN```句，```ANY```句の違い

#### ・```IN```句の使い方

  指定した値と同じ『フィールド』を取得

**＊実装例＊**


指定したカラムで，指定した値の『フィールド』を取得

```sql
SELECT
    *
FROM
    { テーブル名 }
WHERE
    { カラム名 } in (xxx, xxx,...);
```
指定したカラムで，指定した値以外の『フィールド』を取得
```sql
SELECT
    *
FROM
    { テーブル名 } 
WHERE
    { カラム名 } not in ({ レコード名1 }, { レコード名2 },...);
```

指定したカラムで，```SELECT```で読み出した値以外の『フィールド』を取得

```sql
SELECT
    *
FROM
    { テーブル名 }
WHERE
    { カラム名 } not in (
        -- 
        SELECT
            { カラム名 }
        FROM
            { テーブル名 }
        WHERE
            { レコード名 } >= 160
    );
```

**【IN句を使用しなかった場合】**

```sql
SELECT
    *
FROM
    fruit
WHERE
    name = "みかん"
    OR name = "りんご";
```

**【IN句を使用した場合】**

```sql
SELECT
    *
FROM
    fruit
WHERE
    name IN("みかん", "りんご");
```

#### ・```ANY```句の使い方

  書き方が異なるだけで，```in```と同じ出力

```sql
SELECT
    *
FROM
    { テーブル名 }
WHERE
    { カラム名 } = ANY(xxx, xxx, xxx);
```

<br>

### ```GROUP BY```句

#### ・使い方

カラムをグループ化し，集合関数を使用して，フィールドの値を計算する．

**＊実装例＊**

指定したカラムをグループ化し，フィールドの値の平均値を算出する．

```sql
SELECT
    { カラム名1 },
    AVG({ カラム名2 })
FROM
    { テーブル名 }
GROUP BY
    { カラム名1 };
```

<br>

### ```HAVING```句

#### ・使い方

各句の処理の順番から考慮して，```GROUP BY```でグループ化した結果から，```HAVING```で『フィールド』を取得．```SELECT```における集計関数が，```HAVING```における集計関数の結果を指定していることに注意せよ．

**＊実装例＊**

```sql
-- HAVINGによる集計結果を指定して出力．
SELECT
    { カラム名1 },
    COUNT({ カラム名2 })
FROM
    { テーブル名 }
GROUP BY
-- グループ化した結果を集計し，２個以上の『フィールド』を取得
    { カラム名1 }
HAVING
    COUNT(*) >= 2;
```

※以下の場合，```GROUP BY + HAVING```を使っても，```WHERE```を使っても，同じ出力結果になる．

```sql
SELECT
    { カラム名 }
FROM
    { テーブル名 }
GROUP BY
    { カラム名 }
HAVING
    { レコード名 };
```

```sql
SELECT
    { カラム名 }
FROM
    { テーブル名 }
WHERE
    { レコード名 }
GROUP BY
    { カラム名 };
```

<br>

### ```WILDCARD```句

#### ・使い方

**＊実装例＊**

```sql
SELECT
    *
FROM
    { テーブル名 }
WHERE
    { カラム名 } LIKE "%営業";
```

```sql
SELECT
    *
FROM
    { テーブル名 }
WHERE
    { カラム名 } LIKE "_営業";
```

<br>

### ```BETWEEN```句

#### ・使い方

**＊実装例＊**

指定したカラムで，1以上10以下の『フィールド』を取得

```sql
SELECT
    *
FROM
    { テーブル名 }
    BETWEEN 1
    AND 10;
```

<br>

### ```SET```句

#### ・使い方

**＊実装例＊**

```sql
SET
    @A = { パラメータ値 };

SET
    @B = { パラメータ値 };

UPDATE
    { テーブル名 }
SET
    { カラム名 } = @A,
WHERE
    { カラム名 } = @B;
```

<br>

### サブクエリ

#### ・使い方

掛け算と同様に，括弧内から先に処理を行う．

**＊実装例＊**

```sql
-- Main-query
SELECT
    *
FROM
    { テーブル名 }
WHERE
    { カラム名 } != (
        -- Sub-query
        SELECT
            max({ カラム名 })
        FROM
            { テーブル名 }
    );
```

<br>

### インデックス

#### ・インデックスとは

テーブルから特定のカラムだけを抜き出し，検索しやすいように並び替え，名前を付けて保存しておいたもの．インデックスとして保存されたカラムから特定のレコードを直接取得できるため，SQLの実行時間がカラム数に依存しなくなる．インデックスを使用しない場合，SQLの実行時に全てカラムを取得するため，実行時間がテーブルのカラム数に依存してしまう．

#### ・クラスタインデックス

プライマリキーあるいはユニークキーのカラムを抜き出して並び替えたインデックスのこと．

#### ・セカンダリインデックス

プライマリキーあるいはユニークキーではないカラムを抜き出して並び替えたインデックスのこと．

#### ・複合インデックス

複数のカラムをひとまとめに抜き出して並び替えたインデックスのこと．対象としたカラムごとに異なる値のレコード数が計測され，この数が少ない（一意の値の多い）カラムが検出される．そして，カラムのレコードの昇順で並び替えられ，インデックスとして保存される．

**＊例＊**

以下のようなテーブルがあり，```name```カラムと```address```カラムをインデックスとして抜き出すとする．

| id   | name      | address | old  |
| ---- | --------- | ------- | ---- |
| 1    | Suzuki    | Tokyo   | 24   |
| 2    | Yamada    | Osaka   | 18   |
| 3    | Takahashi | Nagoya  | 18   |
| 4    | Honda     | Tokyo   | 16   |
| 5    | Endou     | Tokyo   | 24   |

抜き出されたカラムごとに異なる値のレコード数が計測され，```name```カラムは```address```カラムよりも一意のレコードが多いため，```name```カラムの昇順（アルファベット順）に並び替えられ，インデックスとして保存される．

| name      | address |
| --------- | ------- |
| Endou     | Tokyo   |
| Honda     | Tokyo   |
| Suzuki    | Tokyo   |
| Takahashi | Nagoya  |
| Yamada    | Osaka   |

<br>

### ```EXPLAIN```句

#### ・使い方

設定した```SELECT```句が仮に実行された場合に，いずれのテーブルのいずれのカラムを取得することになるかを表示する．また，想定実行時間も検出できるため，スロークエリの検出に役立つ．

参考：https://dev.mysql.com/doc/refman/5.7/en/explain-output.html

```sql
EXPLAIN
SELECT
    *
FROM
    t1,
    t2
WHERE
    t1.c1 = 1
    AND t1.c2 = t2.c3 
```

```shell
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
         type: ref
possible_keys: index_t1_on_c1_and_c2
          key: index_t1_on_c1_and_c2
      key_len: 5
          ref: const
         rows: 10
        Extra: Using where; Using index
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: t2
         type: ref
possible_keys: index_t2_on_c3
          key: index_t2_on_c3
      key_len: 5
          ref: sample.t1.c2
         rows: 1
        Extra: Using index   
```

#### ・```select_type```

SQLの種類が表示される．サブクエリを含まないSQLは```SIMPLE```となり，サブクエリを含むと，サブクエリの種類に応じて，```PRIMARY```，```SUBQUERY```，```DEPENDENT SUBQUERY```，```UNCACHEABLE SUBQUERY```，```DERIVED```，のいずれかが表示される．

#### ・```table```

設定した```SELECT```句がアクセスするテーブル名が表示される．

#### ・```type```

設定した```SELECT```句がテーブルにアクセスする時に，どの程度の数のカラムを検索するのかが表示される．検索するカラムが多いSQLほど，想定実行時間が長くなる．

| 種類   | 条件                                                         | 検索するカラム数           | 補足                                                         |
| ------ | ------------------------------------------------------------ | -------------------------- | ------------------------------------------------------------ |
| ALL    | ・インデックスを使用していない．                             | 全てのカラム               | 全てのカラムを検索するため，実行時間が最も長く，改善する必要がある． |
| index  | ・インデックスを使用していない．                             | 全てのインデックスのカラム |                                                              |
| range  | ・セカンダリインデックスを使用している．<br>・```WHERE```句に重複したレコード値，```IN```句，```BETWEEN```句を使用している． | 特定の複数カラム           |                                                              |
| ref    | ・セカンダリインデックスを使用している．<br>・```WHERE```句に重複しないレコード値 | 特定の複数カラム           |                                                              |
| eq_ref | ・クラスタインデックスを使用している．                       | 一つのカラム               | 一つのカラムしか```fetch```しないため，```JOIN```句を使用したアクセスの中で，実行時間が最も短い． |
| const  | ・クラスタインデックスを使用している．<br>・```JOIN```句を使用していない． | 一つのカラム               | 一つのカラムしか```fetch```しないため，実行時間が最も短い．  |

#### ・```possible_keys```

インデックスとして設定されたカラムのうちで，実際に利用可能なものの一覧が表示される．

<br>

### Tips

#### ・各データベース容量の確認

```sql
SELECT
    table_schema,
    sum(data_length) / 1024 / 1024 AS mb
FROM
    information_schema.tables
GROUP BY
    table_schema
ORDER BY
    sum(data_length + index_length) DESC;
```

#### ・カラムの検索

```sql
SELECT
    table_name,
    column_name
FROM
    information_schema.columns
WHERE
    column_name = { 検索したいカラム名 }
    AND table_schema = { 検索対象のデータベース名 }
```

#### ・最適なインデックスの検出

<br>

## 04-02. 読み出されたレコードの取得

###  ```FETCH```

#### ・```FETCH```とは

読み出したレコードをに一度に全て取得してしまうと，サーバ側のメモリを圧迫してしまう．そこで，少しずつ取得する．

#### ・```FETCH```のメソッド名に関する注意点

注意点として，```FETCH()```系のメソッドは，ベンダーによって名前が異なっていることがある．そのため，同じ名前でも同じ分だけレコードを取得するとは限らない．

<br>

### PDOにおける```FETCH```

#### ・```fetch(): array```

読み出された全てのレコードのうち，最初のレコードの全てのカラムを取得し，一次元の連想配列で返却する．

#### ・```fetchAll(): array```

読み出された全てのレコードの，全てのカラムを取得し，二次元の連想配列で返却する．

**＊実装例＊**

```php
<?php
// SELECT文を定義して実行．
$sql = "SELECT * FROM doraemon_characters";
$stmt = $dbh->prepare($sql);
$stmt->execute();


// 全てのレコードを取得
$data = $stmt->fetchAll();

// 出力
print_r($data);

// カラム名と値の連想配列として取得できる．
// Array
// (
//     [0] => Array
//     (
//         [id] => 1
//         [name] => のび太
//         [gender] => man
//         [type] => human
//     )
//     [1] => Array
//     (
//         [id] => 2
//         [name] => ドラえもん
//         [gender] => man
//         [type] => robot
//     )
// )
```

#### ・```fetchColumn(): mixed```

読み出された全てのレコードのうち，最初のレコードの一番左のカラムのみを取得し，混合型で返却する．主に，```COUNT()```の場合に用いる

```php
<?php
// SELECT文を定義して実行．
$sql = "SELECT { カラム名 }OUNT(*) FROM doraemon_characters";
$stmt = $dbh->prepare($sql);
$stmt->execute();

// レコードを取得
$data = $stmt->fetchColumn();

// 出力
print_r($data); 

// 10 (件)
```

<br>

### Javaの場合

PHPとは異なり，変数定義に『$』は用いないことに注意．

**＊実装例＊**

```java
// SELECT文を定義して実行．
String sql = "SELECT * FROM doraemon_characters";
ResultSet result statement.executeQuery();


// 全てのレコードを取得
while(result.next()){
    System.out.println(result.getInt("id"));
    System.out.println(result.getString("name"));
    System.out.println(result.getString("gender"));
    System.out.println(result.getString("typeL"));
}

// カラム名と値の連想配列として取得できる．
// ここに出力結果コードを書く．

```

<br>

## 05. レコードの書き込み ：CREATE，UPDATE，DELETE

### ```INSERT```

#### ・PDOの場合

```php
<?php
// $_POSTを用いて，送信されたpostメソッドのリクエストを受け取り，属性から各値を取得
$staff_name = $_POST["name"];
$staff_pass = $_POST["pass"];


// HTMLとして変数の内容を出力する際，「<」「>」などの特殊文字をエスケープ（無害化）
$staff_name = htmlspecialchars($staff_name, ENT_QUOTES, "UTF-8");
$staff_pass = htmlspecialchars($staff_pass, ENT_QUOTES, "UTF-8");


// データベースと接続（イコールの間にスペースを入れるとエラーになる）
$dsn = "mysql:dbname=kizukeba_pronami_php;
host=kizukebapronamiphp
charaset=UTF-8";
$user = "root";
$password = "";
$dbh = new PDO($dsn, $user, $password);
$dbh->setAttribute(PDO::ATTR_ERRMODE,PDO::ERRMODE_EXCEPTION);


// 列名と値を指定してINSERT
$sql="INSERT INTO mst_staff (name,password) VALUES (?,?)";
$stmt = $dbh->prepare($sql);


// 配列に値を格納（格納する値の順番と，SQLでの引数の順番は，合わせる必要がある）
$data[] = $staff_name;
$data[] = $staff_pass;


// SQLを実行
$stmt->execute($data);


// データベースとの接続を切断
$dbh = null;
```

#### ・マイグレーションの場合

マイグレーションファイルと呼ばれるスクリプトファイルを作成し，テーブルの新規作成やカラムの追加はこのスクリプトファイルに記述していく．

1. 誰かが以下のMigrationファイルをmaster別名にPush

2. Migrationファイルをローカル環境にPull

3. データベース更新バッチを実行し，ローカル環境のデータベーススキーマとレコードを更新

**＊実装例＊**

```php
<?php
namespace Migration;

class ItemQuery
{
    // 列名と値を指定してINSERT
    public static function insert()
    {
        return "INSERT INTO item_table VALUES(1, "商品A", 1000, "2019-07-24 07:07:07");";
    }
}
```

<br>

### ```UPDATE```

```sql

```

<br>

### ```DELETE```

```sql

```

<br>

## 06. その他

### レコードの突き合わせ処理アルゴリズム

#### ・突き合わせ処理とは

ビジネスの基盤となるマスタデータ（商品データ，取引先データなど）と，日々更新されるトランザクションデータ（販売履歴，入金履歴など）を突き合わせ，新しいデータを作成する処理のこと．

![マッチング処理_1](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/マッチング処理_1.PNG)

#### ・アルゴリズム

![マッチング処理_4](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/マッチング処理_4.png)

#### ・具体例

とある生命保険会社では，顧客の保険契約データを契約マスタテーブルで，またそれとは別に，保険契約データの変更点（異動事由）を異動トランザクションテーブルで，管理している．毎日，契約マスタテーブルと異動トランザクションテーブルにおける前日レコードを突き合わせ，各契約の異動事由に応じて，変更後契約データとして，新契約マスタテーブルに挿入する．

![マッチング処理_2](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/マッチング処理_2.PNG)

前処理として，契約マスタデータと異動トランザクションデータに共通する識別子が同じ順番で並んでいる必要がある．

1. 契約マスタデータの1行目と，異動トランザクションデータの1行目の識別子を突き合わせる．『契約マスタデータ = 異動トランザクションデータ』の時，異動トランザクションデータを基に契約マスタデータを更新し，それを新しいデータとして変更後契約マスタデータに挿入する．
2. 契約マスタデータの2行目と，異動トランザクションデータの2行目の識別子を突き合わせる．『マスタデータ < トランザクションデータ』の場合，マスタデータをそのまま変更後マスタテーブルに挿入する．
3. マスタデータの3行目と，固定したままのトランザクションデータの2行目の識別子を突き合わせる．『マスタデータ = トランザクションデータ』の時，トランザクションデータを基にマスタデータを更新し，それを変更後データとして変更後マスタテーブルに挿入する．
4. 『契約マスタデータ < 異動トランザクションデータ』になるまで，データを突き合わせる．
5. 最終的に，変更後マスタテーブルは以下の通りになる．

![マッチング処理_3](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/マッチング処理_3.png)

