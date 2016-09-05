# PostgreSQL 9.5.x　翻訳

以下ファイルについて格納します。

|No.|ファイルパス|概要|行数|
| --- | --- | --- | --- |
|1|README|PostgreSQLの概要|27|
|2|README.git|gitリポジトリにINSTALLファイルがない説明|14|
|3|contrib/README|contribツールの概要(詳細はドキュメントに移管)|28|
|4|contrib/start-scripts/osx/README|OS X用の起動スクリプト|3|
|5|doc/src/sgml/README.links|SGML文書のリンク|46|
|6|src/backend/access/gin/README|GINインデックス|379|
|7|src/backend/access/gist/README|GiSTインデックス|419|
|8|src/backend/access/hash/README|HASHインデックス|467|
|9|src/backend/access/heap/README.HOT|ヒープのHOT更新|499|
|10|src/backend/access/heap/README.tuplock|ヒープのタプルロック|144|
|11|src/backend/access/nbtree/README|B-Treeインデックス|656|
|12|src/backend/access/spgist/README|SP-GiSTインデックス|373|
|13|src/backend/access/transam/README|トランザクション管理|850|
|14|src/backend/catalog/README|システムカタログ|111|
|15|src/backend/executor/README|エグゼキュータ|202|
|16|src/backend/libpq/README.SSL|libpqのSSL接続|60|
|17|src/backend/nodes/README|ノードシステム（PostgreSQL自前のOOP的機構）|80|
|18|src/backend/optimizer/README|オプティマイザ|854|
|19|src/backend/optimizer/plan/README|プランナ|158|
|20|src/backend/parser/README|パーサ|30|
|21|src/backend/port/darwin/README|Darwin依存のコード|36|
|22|src/backend/regex/README|正規表現|371|
|23|src/backend/replication/README|レプリケーション|99|
|24|src/backend/snowball/README|ワードステミング|49|
|25|src/backend/storage/buffer/README|共有バッファ管理|282|
|26|src/backend/storage/freespace/README|空き領域マップ|196|
|27|src/backend/storage/lmgr/README|内部レベルのロック機構（LWLockやスピンロックなど）|639|
|28|src/backend/storage/lmgr/README-SSI|SSI TX分離レベル|629|
|29|src/backend/storage/lmgr/README.barrier|メモリバリア|199|
|30|src/backend/storage/page/README|ページ管理|63|
|31|src/backend/storage/smgr/README|ストレージマネージャ|58|
|32|src/backend/utils/fmgr/README|SQL関数呼び出し機構|556|
|33|src/backend/utils/mb/README|マルチバイト文字関連ソースの説明|20|
|34|src/backend/utils/mb/conversion_procs/README.euc_jp|エンコード変換関数の追加方法|83|
|35|src/backend/utils/misc/README|GUCパラメータ実装|295|
|36|src/backend/utils/mmgr/README|メモリアロケータ|448|
|37|src/backend/utils/resowner/README|リソースオーナによるクリーンアップ機構|86|
|38|src/bin/pgevent/README|Windowsイベントログ用DLL|20|
|39|src/interfaces/ecpg/README.dynSQL|ECPGの動的SQL|11|
|40|src/interfaces/ecpg/preproc/README.parser|ECPG特有のパース処理|42|
|41|src/interfaces/ecpg/test/connect/README|接続テストの説明|9|
|42|src/interfaces/libpq/README|ディレクトリの内容|3|
|43|src/interfaces/libpq/test/README|libpq用テストスイート|7|
|44|src/pl/plperl/README|PL/Perl|10|
|45|src/pl/plpython/expected/README|PL/Pythonのバージョン別テスト予想結果ファイル|12|
|46|src/pl/tcl/modules/README|PL/Tcl|18|
|47|src/port/README|環境差吸収用ライブラリlibpgport|32|
|48|src/test/isolation/README|テストツール（TX分離性）|117|
|49|src/test/locale/README|テストツール（ロケール）|28|
|50|src/test/locale/de_DE.ISO8859-1/README|テストツール（de_DE.ISO8859-1ロケール）|4|
|51|src/test/locale/gr_GR.ISO8859-7/README|テストツール（gr_GR.ISO8859-7ロケール）|4|
|52|src/test/locale/koi8-to-win1251/README|テストツール（koi8-to-win1251ロケール）|6|
|53|src/test/mb/README|テストツール（マルチバイト文字）|10|
|54|src/test/regress/README|テストツール（スレッド）|3|
|55|src/test/thread/README|テストツール（ロケール）|54|
|56|src/timezone/README|タイムゾーン|42|
|57|src/timezone/tznames/README|タイムゾーン略名|34|
|58|src/tools/findoidjoins/README|開発用ツール（findoidjoins:OID列の結合調査）|208|
|59|src/tools/ifaddrs/README|開発用ツール（ifaddrs:IPv4/6アドレス調査）|12|
|60|src/tools/make_diff/README|開発用ツール（make_diff:diff一括取得）|39|
|61|src/tools/msvc/README|開発用ツール（MS VCビルド）|103|
|62|src/tools/pginclude/README|開発用ツール（pginclude:#includeクリーンアップ）|55|
|63|src/tools/pgindent/README|開発用ツール（pgindent:インデント整形）|113|
|64|src/tutorial/README|SQLチュートリアル|16|
