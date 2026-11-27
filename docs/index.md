# 「蔵」

「蔵」, the database layer of Suru. SQLite, PostgreSQL and MySQL can be swapped by changing only the driver.
Add it with `kumu 追加 github.com/suru-lang/db` and write `「蔵」 を 読み込んで 使う` in your file. Every name starts with `DB`.
A driver connects to the database. Drivers are in other libraries (for SQLite, `SQLite駆動` in suru-lang/sqlite). Change only the driver in `先 を 駆動 で DB開封する`, and the rest of the code stays the same.

A database value is the type `DB値`, which has these forms:

- `DB無`: NULL.
- `42 の DB数`: a number (integer or decimal).
- `「a」 の DB文字列`: a string.
- `b の DBバイト列`: a byte sequence.
- `真 の DB真偽`: a truth value (1 and 0 in databases without one, such as SQLite).

Every function that touches the database is an action. When it fails, it is a runtime error with the driver's reason, which `試行する` can catch.

For driver authors: `DB駆動` is a record of the name and the open function. The open function is an action that takes 先 and gives a `DB接続`, which holds the functions of the database (execute, query, last id, close) and its `DB方言` (see "Writing a driver" in the README).

```{toctree}
:maxdepth: 2
:caption: API
:glob:

api/*
```
