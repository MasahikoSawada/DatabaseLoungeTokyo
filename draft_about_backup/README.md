# タイムテーブル案

|時間  |やること| 
|------|--------|
|19:15|Lighting Talks #1|
|19:25|Lighting Talks #2|
|19:35|Lighting Talks #3|
|19:45|休憩|
|20:00|渡部さんによるバックアップ・リストアの話(仮)|
|20:30|パネルディスカッション|
|21:30|ビアバッシュ|
|23:00|閉場|

# MySQLからOracle, PostgreSQLに聞きたい

- Oracle, PostgreSQLのバックアップの知識は [バックアップと障害復旧から考えるOracle Database, MySQL, PostgreSQLの違い](http://www.slideshare.net/ryotawatabe/2016715-jpoug-rdbms-backup-architecture) を読んだくらいしかないです

## 共通

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

- ストレージスナップショットって使うの？
  - PostgreSQLだと
    - ハードウェアの機能と連動したバックアップツールは無い気がする。なのでだいたい手作り。
    - pg_rmanはAPI的なものを提供しているけど作りこみ必要だよね・・
- パラレルリカバリって早い？
  - PostgreSQLはできません。MySQLはできる？
  - oracleはできるそうですが、やっぱり早い？ケースによる？
- バックアップの暗号化
  - PostgreSQLだと
    - たまにニーズがある。
    - PostgreSQLではバックアップファイルを暗号化するものは無い・・気がする。商用だとあるかも。
  - そもそもDBとかファイルシステム(ブロックデバイス)の暗号化とかに絡んでくるし、ハードウェアでやるってのもあるのでバックアップ話から逸れるけど・・
- オンラインバックアップのファイルコピー
  - start_backup ～ stop_backup間で行うファイルコピーってrsync？がメジャー？
    - PostgreSQLではrsyncがメジャー
- PITRのリカバリポイント
  - オペミスとかした時にどうやってリカバリ位置を決めてますか？
  - PostgreSQLだとXIDか時間(10.0からはLSNも可)ですが、明確に記録する仕組みは作りこむか手順で定める必要がある。
    -  DDL操作時間の履歴が無いので・・
  -  この辺、他のDBMSでどうやってやってるのか気になります。

## for Oracle 

- RMANってどこまで追加費用なしで使えるの？
  - や、そもそもどんな機能があるのかよくわかってないですが

## for PostgreSQL

- `SELECT pg_start_backup()` ってした時に中では何が起こってるの？

- `pg_start_backup`と`pg_stop_backup`の間にクラッシュした場合はどうなるの？
