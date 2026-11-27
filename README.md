English | [日本語](README_jp.md)

# 蔵 (a database layer for Suru)

A thin database layer for [Suru (「する」)](https://github.com/suru-lang/suru), a programming language in Japanese. It decides only connecting, executing, fetching rows and transactions, so the same program can use SQLite, PostgreSQL or MySQL by changing the name of one driver. It is written in Suru only and needs no C library; the drivers live in their own repositories. The O/R mapper 「台帳」 in [suru-lang/orm](https://github.com/suru-lang/orm) is built on top of it (plan: suru-lang/suru#164).

## Installing

In a [kumu](https://github.com/suru-lang/suru#building-projects-kumu) project, add this layer and the driver for your database:

```sh
kumu 追加 github.com/suru-lang/db
kumu 追加 github.com/suru-lang/sqlite
```

| Database | Driver | Repository |
|---|---|---|
| SQLite | `SQLite駆動` | [suru-lang/sqlite](https://github.com/suru-lang/sqlite) v0.2.0 or later |
| PostgreSQL | `PostgreSQL駆動` | [suru-lang/postgresql](https://github.com/suru-lang/postgresql) v0.1.0 or later |
| MySQL | `MySQL駆動` | [suru-lang/mysql](https://github.com/suru-lang/mysql) v0.1.0 or later |

## Usage

```
「蔵」 を 読み込んで 使う
「SQLite」 を 読み込んで 使う

DB行 を 名前取得する は 文字列
行 を 名前取得する とは 行 から 「名前」 を DB列取得して DB文字列取出する

主処理する とは 入出力手順
  db とは 「店.db」 を SQLite駆動 で DB開封した 結果
  db に 「CREATE TABLE IF NOT EXISTS 品 (名前 TEXT, 値段 REAL)」 を DB実行する
  db に 「INSERT INTO 品 VALUES (?, ?)」 を （（「りんご」 の DB文字列） と （120 の DB数）） で DB差込実行する
  行群 とは db に 「SELECT 名前, 値段 FROM 品 WHERE 値段 < ?」 を （（200 の DB数） を 空 に 挿入する） で DB行問合した 結果
  （行群 の それぞれ を 名前取得して 改行 で 結合する） を 表示する
  db を DB閉鎖する
  「以上です」 を 表示する

主処理する
```

To use PostgreSQL instead, write `「PostgreSQL」 を 読み込んで 使う` and `… を PostgreSQL駆動 で DB開封した 結果`; the rest stays the same. Every name starts with `DB`. With `「蔵」 を 読み込む`, write them with the prefix, as in `蔵内のDB開封する`.

## Decisions

1. **Connecting through a driver.** `先 を 駆動 で DB開封する` connects and gives a connection (`DB接続`). What `先` means depends on the driver (a file path for SQLite). `db を DB閉鎖する` closes it. All functions that touch the database are actions.
2. **Values have the type with alternatives `DB値`.** `DB無` (NULL), `42 の DB数` (integers and decimals), `「a」 の DB文字列`, `b の DBバイト列` and `真 の DB真偽`. Values read back have this type too. `v を DB数取出する` (also `DB文字列取出する`, `DBバイト列取出する`, `DB真偽取出する`) takes the value out, and is a runtime error for another alternative. `DB真偽取出する` also accepts the numbers 1 and 0, because SQLite and MySQL store truth values as numbers. `v を DB値文字列にする` is a string for display (「NULL」 for `DB無`).
3. **Executing and querying.** `db に 文 を DB実行する` runs SQL without result rows and gives the number of changed rows. `db に 文 を DB問合する` gives the result rows, each a list of `DB値` in column order. `db を DB最終番号取得する` gives the auto-numbered id of the last INSERT.
4. **Values go into 「?」.** `DB差込実行する` and `DB差込問合する` take a list of `DB値` with 「で」 and put them into the statement's 「?」 in order. Placeholders are always written 「?」, whatever the database; the PostgreSQL driver changes them to `$1, $2, …` with `文 を DB番号差込変換する`, which leaves 「?」 inside strings and quoted names alone.
5. **Rows by column name.** `db に 文 を 値群 で DB行問合する` gives rows of type `DB行`, and `行 から 「名前」 を DB列取得する` gives the value of that column (a runtime error if there is no such column).
6. **Transactions.** `db で 処理 を DB取引する` runs the action `処理` between BEGIN and COMMIT and gives its result. If `処理` fails with a runtime error, it runs ROLLBACK and fails with the same message (catchable with `試行する`).
7. **Dialects.** A connection carries a `DB方言` (`db の DB方言値`) with the database's differences: the character that quotes names, the column types for integers, decimals, strings, byte strings, truth values and auto-numbered primary keys, and whether `INSERT … RETURNING` can be used. `DBSQLite方言`, `DBPostgreSQL方言` and `DBMySQL方言` are provided. `名前 を 方言 で DB名囲みする` quotes a table or column name (`"名前"`, or `` `名前` `` for MySQL). The O/R mapper uses these to build SQL.
8. **Failures are runtime errors.** Drivers fail with the database's reason, catchable with `試行する`.

## Writing a driver

A driver is `名前 と 開く関数 の DB駆動`. The opening function takes `先` and gives a `DB接続` as an action. A `DB接続` is a record of five fields, in this order:

1. `DB実行口`: a function `文 を 値群 で …` giving the number of changed rows (an action).
2. `DB問合口`: a function `文 を 値群 で …` giving the column names and rows, `列名群 と 行群 の DB問合結果` (an action).
3. `DB最終番号口`: an action giving the last auto-numbered id.
4. `DB閉鎖口`: an action that closes the connection.
5. `DB方言値`: the `DB方言`.

Make the functions by partially applying the driver's own functions to its handle, as in `（h に 低実行すること）`. [試験/層.suru](試験/層.suru) has a fake driver written this way.

## API documentation

The API documentation is made from the comments in `蔵.suru`: the head comment of the file and the comment right above each public definition, written in English.

```sh
kumu 文書                                                            # writes docs/api/蔵.md
pip install -r docs/requirements.txt && sphinx-build docs docs/_build   # HTML in docs/_build
```

`docs/` and `.readthedocs.yaml` were written by `kumu 文書 --雛形` (Suru 0.11.0 or later). On every build, Read the Docs downloads kumu 0.11.0, runs `kumu 文書 --出力 docs/api` and then Sphinx, so the generated pages are not committed. The editor hover shows the same descriptions.

## Tests

```sh
kumu 試験
kumu 点検
```

The tests use a fake driver that writes the SQL it receives to a file, so no database is needed. Each driver tests itself against its real database.

## Future work

- Reuse prepared statements, and read large results one row at a time
- Named placeholders 「:名前」
- Date and time values (`日時` of the standard library)
- Connection pools for threads

## License

[MIT](LICENSE)
