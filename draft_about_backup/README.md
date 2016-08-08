# MySQLからOracle, PostgreSQLに聞きたい

- Oracle, PostgreSQLのバックアップの知識は [バックアップと障害復旧から考えるOracle Database, MySQL, PostgreSQLの違い](http://www.slideshare.net/ryotawatabe/2016715-jpoug-rdbms-backup-architecture) を読んだくらいしかないです

## 両方

- 論理バックアップと物理バックアップ、どっちが主流だと思います？ （偏見でいいです）
  - MySQLだと（偏見）
    - データベースが十分小さい間 .. 稼働系で `mysqldump`
    - mysqldumpでリストアが追いつかなくなると .. バックアップ専用スレーブ作ってそれを止めて（or XtraBackup）物理バックアップ

- 「1つのテーブルだけもとに戻したい」時にどうする？
  - MySQLは別の場所にフルリストアしてから、そのテーブルだけ `FLUSH TABLES FOR EXPORT` か `mysqldump`

- 基本になる物理バックアップファイルの中に、WALもセットにしてくれる？ クラッシュリカバリーまではコマンド一発とかでできる？
  - もし興味があれば、XtraBackupの動作原理を説明します（たぶん、OracleやPostgreSQLの物理バックアップに似てます）

- バックアップって稼働系から取るのが常識？
  - MySQLはバックアップ専用スレーブ（Not 稼働系）を作ることが少なくない

- バックアップに関して（他のDBMSでできるかどうかは置いておいて）一番お気に入りの機能を教えてください :)
  - MySQLだったら、レプリケーションが楽なのでバックアップ専用スレーブという選択肢がフツーにあること、かなあ。。

- フル、差分、増分バックアップ、どうやってやる？ どれくらいのスパンにする？ （雑に）
  - MySQLに（原則）差分バックアップはないです。ウチの環境だと、デイリーでフルバックアップ + ストリームで増分バックアップ（バイナリーログバックアップ、 `log_slave_updates` の場合を含む）

- 論理バックアップのパラレルリストアの仕組みってあります？
  - MySQLだと今のところMyDumper一択
  - PostgreSQLだとpg_restoreでパラレルリストア可能

## for Oracle 

- RMANってどこまで追加費用なしで使えるの？
  - や、そもそもどんな機能があるのかよくわかってないですが

## for PostgreSQL

- `SELECT pg_start_backup()` ってした時に中では何が起こってるの？

- `pg_start_backup`と`pg_stop_backup`の間にクラッシュした場合はどうなるの？