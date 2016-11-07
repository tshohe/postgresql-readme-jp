src/backend/utils/misc/README

GUC Implementation Notes
========================

GUC 実装上の注意
========================

The GUC (Grand Unified Configuration) module implements configuration
variables of multiple types (currently boolean, enum, int, real, and string).
Variable settings can come from various places, with a priority ordering
determining which setting is used.


GUC（Grand Unified Configuration）モジュールは、
複数のタイプ（currently boolean, enum, int, real, and string）の設定変数を実装しています。
変数設定は、設定を決定する優先順位付けと、様々な場所から決定されるものです。

Per-Variable Hooks
------------------

変数毎のフック
------------------

Each variable known to GUC can optionally have a check_hook, an
assign_hook, and/or a show_hook to provide customized behavior.

GUCに把握されている各変数は、必要に応じてcheck_hook、assign_hookを持つことができ、
および/またはshow_hookは、カスタマイズされた動作を提供します。


Check hooks are used to perform validity checking on variable values
(above and beyond what GUC can do), to compute derived settings when
nontrivial work is needed to do that, and optionally to "canonicalize"
user-supplied values.  

チェックフックは、設定を行うために重要な作業を行うために必要とされる派生の設定の計算、
および、必要に応じてのユーザ指定値の"正規化"を実施するため、
変数の値妥当性（GUC上およびは超えて何ができるか）のチェックを実行するために使用されています。

Assign hooks are used to update any derived state
that needs to change when a GUC variable is set.  Show hooks are used to
modify the default SHOW display for a variable.

割り当てフックはGUC変数が設定されているときに、
変更する必要がある任意の派生状態を更新するために使用されます。
表示フックは、変数のデフォルトのSHOWの表示を変更するために使用されています。

If a check_hook is provided, it points to a function of the signature
	bool check_hook(datatype *newvalue, void **extra, GucSource source)
The "newvalue" argument is of type bool *, int *, double *, or char **
for bool, int/enum, real, or string variables respectively.  The check
function should validate the proposed new value, and return true if it is
OK or false if not.  The function can optionally do a few other things:

check_hookが提供されている場合、それは下記の関数の定義をポイントしています。
	bool check_hook(datatype *newvalue, void **extra, GucSource source)
"newvalue引数はそれぞれbool *, int *, double *, もしくは bool型のchar **, 
int/enum, realもしくはそれぞれの文字列変数です。
チェック関数が提案された新しい値を検証し、
それがOKである場合はtrue、そうでない場合はfalseで返す必要があります。
この関数は、必要に応じて他のいくつかのことを行うことができます：

* When rejecting a bad proposed value, it may be useful to append some
additional information to the generic "invalid value for parameter FOO"
complaint that guc.c will emit.  To do that, call
	void GUC_check_errdetail(const char *format, ...)

* 悪い提案値を拒否する場合は、guc.cが出力する一般的な"パラメータFOOに無効な値"の訴えに、
  いくつかの追加情報を付与するために有用である場合があります。それを行うために、下記を呼びます。
	void GUC_check_errdetail(const char *format, ...)

where the format string and additional arguments follow the rules for
errdetail() arguments.  The resulting string will be emitted as the
DETAIL line of guc.c's error report, so it should follow the message style
guidelines for DETAIL messages.  There is also
	void GUC_check_errhint(const char *format, ...)
which can be used in the same way to append a HINT message.
書式文字列と追加の引数の箇所はerrdetail（）の引数の規則に従ってください。
結果文字列はguc.cのエラーレポートの詳細行として出力されますので、
DETAILメッセージのメッセージ形式ガイドラインに従ってください。
	void GUC_check_errhint(const char *format, ...)
同じ方法でHINTメッセージを追加することが可能です。

Occasionally it may even be appropriate to override guc.c's generic primary
message or error code, which can be done with
	void GUC_check_errcode(int sqlerrcode)
	void GUC_check_errmsg(const char *format, ...)
guc.cの一般的なプライマリーメッセージやエラーコードを、
下記の関数で上書きすることが適切であることもあります。
	void GUC_check_errcode(int sqlerrcode)
	void GUC_check_errmsg(const char *format, ...)

In general, check_hooks should avoid throwing errors directly if possible,
though this may be impractical to avoid for some corner cases such as
out-of-memory.
一般的に、check_hooksは可能であれば、直接エラーを投げないようにする必要があります。
out-of-memoryのような、滅多に発生しない厄介なケースでは回避することが困難かもしれません。

* Since the newvalue is pass-by-reference, the function can modify it.
This might be used for example to canonicalize the spelling of a string
value, round off a buffer size to the nearest supported value, or replace
a special value such as "-1" with a computed default value.  
* newvalueは参照渡しであるので、この関数はそれを変更することができます。
これは、次のような特別な値を、文字列値のスペルを正規化する、
最も近いサポート値にバッファサイズを四捨五入する、もしくは
特別な値である「-1」を計算されたデフォルト値に置き換えるために利用されるでしょう。

If the function wishes to replace a string value, it must malloc (not palloc)
the replacement value, and be sure to free() the previous value.
関数は、文字列値を変更したい場合は、置換値をmalloc(pallocではない)し、
以前の値をfree()で解放するようにして下さい。

* Derived information, such as the role OID represented by a user name,
can be stored for use by the assign hook.  
* ユーザ名によって表されるロールOIDのような、引き渡された情報については
assing hookによる使用のために保存することができます。

To do this, malloc (not palloc)
storage space for the information, and return its address at *extra.
これを行うには、情報の格納スペースmalloc (pallocでない)し、
その*extraのアドレスを返却します。

guc.c will automatically free() this space when the associated GUC setting
is no longer of interest.  *extra is initialized to NULL before call, so
it can be ignored if not needed.
guc.cは、関連するGUC設定がもはや影響がない場合に、
自動的にこのスペースを解放(free())します。 *extraは呼び出し前にNULLに初期化されるため、
必要でない場合は、それを無視することが可能です。

The "source" argument indicates the source of the proposed new value,
If it is >= PGC_S_INTERACTIVE, then we are performing an interactive
assignment (e.g., a SET command). 
"ソース"引数は、新しい提案値のソースを示しており、
それが >=PGC_S_INTERACTIVEの場合、我々はインタラクティブな割り当てを実行しています。(例えば、SETコマンド)

But when source < PGC_S_INTERACTIVE,
we are reading a non-interactive option source, such as postgresql.conf.
しかし、ソースが< PGC_S_INTERACTIVEの場合、
我々は、postgresql.confのように、非対話型オプションソースを読んでいます。

This is sometimes needed to determine whether a setting should be
allowed.  The check_hook might also look at the current actual value of
the variable to determine what is allowed.
これは時折に設定を許可するかどうかを決定するために必要とされます。
check_hookも許可されているかを判断するために、
現在の変数の実際の値を確認することがあります。

Note that check hooks are sometimes called just to validate a value,
without any intention of actually changing the setting.  Therefore the
check hook must *not* take any action based on the assumption that an
assignment will occur.
実際の設定変更の意図がないような場合にも、単に値のバリデートのため、
check hooksが呼ばれることがあることに気を付けて下さい。従って
check hooksは、割り当てが発生するという仮定に基づいて任意の動作を実行しては*なりません*。

If an assign_hook is provided, it points to a function of the signature
	void assign_hook(datatype newvalue, void *extra)
where the type of "newvalue" matches the kind of variable, and "extra"
is the derived-information pointer returned by the check_hook (always
NULL if there is no check_hook).  This function is called immediately
before actually setting the variable's value (so it can look at the actual
variable to determine the old value, for example to avoid doing work when
the value isn't really changing).

assign_hookが提供される場合、それは下記関数の定義をポイントします。
	void assign_hook(datatype newvalue, void *extra)
ここでは「newvalue」の型は、変数の種別に一致し、派生情報ポインタ"extra"は
check_hookによって返されます(check_hookがない場合は常にNULL)。この関数は、
実際には変数の値を設定する直前に呼び出されます。(古い値を決定するために、
実際の変数を見ることができます。例えば値が実際に変更されていない場合に、
作業を行うことを避けることができます。)

Note that there is no provision for a failure result code. assign_hooks
should never fail except under the most dire circumstances, since a failure
may for example result in GUC settings not being rolled back properly during
transaction abort.  
失敗結果コードに関する規定が存在しないことに注意してください。
assign_hooksは、障害が例えばGUC設定をもたらす可能性があるため、
トランザクションアボート中に適切にロールバックされないため、
最も悲惨な状況を除いて失敗することはありません。

In general, try to do anything that could conceivably
fail in a check_hook instead, and pass along the results in an "extra"
struct, so that the assign hook has little to do beyond copying the data to
someplace.  
一般的に、check_hookにおける失敗の代わりに考えられる限りの何をしようと、
"extra"構造体内に結果を伝えます。そのため、assign hookはどこかへのデータコピーを超えて、
行うことはほとんどありません。

This applies particularly to catalog lookups: any required
lookups must be done in the check_hook, since the assign_hook may be
executed during transaction rollback when lookups will be unsafe.

これは、カタログの検索に特に適用されます:カタログ検索が安全ではない時、
assign_hookはトランザクションのロールバック中に実行することができるので、
必要な検索は、check_hookで行わなければなりません。

Note that check_hooks are sometimes called outside any transaction, too.
This happens when processing the wired-in "bootstrap" value, values coming
from the postmaster command line or environment, or values coming from
postgresql.conf.  Therefore, any catalog lookups done in a check_hook
should be guarded with an IsTransactionState() test, and there must be a
fallback path to allow derived values to be computed during the first
subsequent use of the GUC setting within a transaction.  A typical
arrangement is for the catalog values computed by the check_hook and
installed by the assign_hook to be used only for the remainder of the
transaction in which the new setting is made.  Each subsequent transaction
looks up the values afresh on first use.  This arrangement is useful to
prevent use of stale catalog values, independently of the problem of
needing to check GUC values outside a transaction.

check_hooksも時々、トランザクションの外部で呼ばれることに注意してください
「bootstrap」の値を処理する場合に上記問題が発生し、postmasterのコマンドライン、
環境依存、またpostgresql.confから来た値を処理する場合に発生します。
従って、check_hookで実施される任意のカタログ検索は、
IsTransactionState（）テストでガードされるべきであり、
得られた値は、トランザクション内のGUC設定の最初の後続の利用中に、
計算できるようにするためのフォールバックパスが存在しなければなりません。
典型的な構成はcheck_hookによって計算されたカタログ値のためのものであり、
assign_hookによるインストールは新しい設定がなされている、
トランザクションの残りの部分にのみ使用されます。後続の各トランザクションは、
最初の使用時に新たに値を検索します。この構成は、古くなったカタログ値の使用を防止および
トランザクション外のGUC値のチェックが必要である問題とは関係なく有用です。

If a show_hook is provided, it points to a function of the signature
	const char *show_hook(void)
This hook allows variable-specific computation of the value displayed
by SHOW (and other SQL features for showing GUC variable values).
The return value can point to a static buffer, since show functions are
not used re-entrantly.

show_hookが提供されている場合、それは下記の関数の定義をポイントしています
	const char *show_hook(void)
このフックは、SHOW（およびGUC変数の値を表示するための、その他のSQL機能）
で表示される値の可変固有の計算を可能にします。
show関数は、リエントラントに使用されないので、
戻り値は、静的バッファを指すことができます。

Saving/Restoring GUC Variable Values
------------------------------------

Prior values of configuration variables must be remembered in order to deal
with several special cases: RESET (a/k/a SET TO DEFAULT), rollback of SET
on transaction abort, rollback of SET LOCAL at transaction end (either
commit or abort), and save/restore around a function that has a SET option.
RESET is defined as selecting the value that would be effective had there
never been any SET commands in the current session.

セーブ/リストア GUC変数の値
------------------------------------

設定変数の前の値は、いくつかの特別な場合に対処するために
忘れてはなりません: RESET (SET TO DEFAULTとして知られる)、 
SETにおけるトランザクションアボートのロールバック、
トランザクションの最後(commitもしくはabort)における、
SET LOCALのロールバック、そしてSETオプションが持っているセーブ/リストア周辺の機能。
RESETは、現在のセッション内の任意のSETコマンドが実行されなかった場合に、
効果的であろう値を選択するように定義されています。

To handle these cases we must keep track of many distinct values for each
variable.  The primary values are:
このような場合を扱うために私たちは、それぞれの変数のために多くの異なる値を追跡する必要があります。
主要な値は次のとおりです:

* actual variable contents	always the current effective value

* reset_val			the value to use for RESET

* actual variable contents	現在の有効値

* reset_val			値はRESETにて使用

(Each GUC entry also has a boot_val which is the wired-in default value.
This is assigned to the reset_val and the actual variable during
InitializeGUCOptions().  The boot_val is also consulted to restore the
correct reset_val if SIGHUP processing discovers that a variable formerly
specified in postgresql.conf is no longer set there.)

(各GUCエントリもワイヤードでのデフォルト値であるboot_valも持っています。
これは、InitializeGUCOptions()にてreset_valと実際の変数に代入されています。
boot_valもSIGHUP処理が以前のpostgresql.conf内で指定された変数が、
もはや設定されていることが判明した場合に、正しいreset_valを復元するためにも参照されます。)

In addition to the primary values, there is a stack of former effective
values that might need to be restored in future.  Stacking and unstacking
is controlled by the GUC "nest level", which is zero when outside any
transaction, one at top transaction level, and incremented for each
open subtransaction or function call with a SET option.  A stack entry
is made whenever a GUC variable is first modified at a given nesting level.
(Note: the reset_val need not be stacked because it is only changed by
non-transactional operations.)

プライマリ値に加えて、将来的にリストアのために
必要になる場合がある前の有効値のスタックが存在します。
スタッキングおよびアンスタッキングはGUC "nest level"により制御され、
0の場合、すべてのトランザクションが外部、トップトランザクション・レベルが1、かつ
SETオプションを指定したオープンサブトランザクションまたは関数呼び出しです。
GUC変数が最初に与えられたネスト・レベルで変更される度にスタックエントリが作成されます。
(注意: reset_valが非トランザクション操作によってのみ変更されるため、
reset_valをスタックする必要はありません。)

A stack entry has a state, a prior value of the GUC variable, a remembered
source of that prior value, and depending on the state may also have a
"masked" value.  The masked value is needed when SET followed by SET LOCAL
occur at the same nest level: the SET's value is masked but must be
remembered to restore after transaction commit.

スタックエントリは状態、GUC変数の前の値、前の値の覚えたソースを保持し、
そして、状態に応じて、「マスクされた」値を保持ができます。マスクされた値は、
SET LOCALが続くSETが同じネスト・レベルで発生したときに必要とされます:
SET値はマスクされていますが、トランザクションのコミット後にリストアすることを忘れてはなりません。

During initialization we set the actual value and reset_val based on
whichever non-interactive source has the highest priority.  They will
have the same value.
初期化中に私たちは実際の値とreset_val、最も優先度の高い非対話ソースに基づいて設定します。

The possible transactional operations on a GUC value are:
GUC値において可能なトランザクション操作は、次のとおりです:

Entry to a function with a SET option:
SETオプションを持つ関数へのエントリ：

	Push a stack entry with the prior variable value and state SAVE,
	then set the variable.
	前の変数の値とセーブ状態とのスタックエントリをプッシュし、
	その変数を設定します。

Plain SET command:
プレーンSETコマンド：

	If no stack entry of current level:
		Push new stack entry w/prior value and state SET
	else if stack entry's state is SAVE, SET, or LOCAL:
		change stack state to SET, don't change saved value
		(here we are forgetting effects of prior set action)
	else (entry must have state SET+LOCAL):
		discard its masked value, change state to SET
		(here we are forgetting effects of prior SET and SET LOCAL)
	Now set new value.
	
	現在のレベルのスタックエントリがない場合：
		新しいスタックエントリに前の値とSET状態をプッシュする
	スタックエントリの状態がSAVE、SET、またはLOCALである場合：
		SETするためスタックの状態を変更、保存された値を変更しない
	そうでない場合（エントリは、SET+ LOCAL状態を持つ必要がある）:
		そのマスクされた値を破棄し、状態をSETに変更します
	そして新しい値を設定します。	
	
SET LOCAL command:
SET LOCALコマンド;

	If no stack entry of current level:
		Push new stack entry w/prior value and state LOCAL
	else if stack entry's state is SAVE or LOCAL or SET+LOCAL:
		no change to stack entry
		(in SAVE case, SET LOCAL will be forgotten at func exit)
	else (entry must have state SET):
		put current active into its masked slot, set state SET+LOCAL
	Now set new value.
	
	現在のレベルのスタックエントリがない場合：
		新しいスタックエントリに前の値とSET状態をプッシュする
	スタックエントリの状態がSAVE、LOCAL、またはSET+LOCALである場合：
		スタックエントリを変更しない
		(SAVEの場合においては、SET LOCALは関数終了時に忘れられる)
	そうでない場合（エントリは、SET状態を持つ必要がある）:
		マスクされたスロットに、現在のアクティブを入れ、状態をSET+LOCALに設定します
	そして新しい値を設定します。
	

Transaction or subtransaction abort:
トランザクションまたはサブトランザクションがアボート:

	Pop stack entries, restoring prior value, until top < subxact depth
	スタックエントリをポップし、top < subxact depthまで前の値をリストアします

Transaction or subtransaction commit (incl. successful function exit):
トランザクションまたはサブトランザクションがコミット (関数の正常終了も含む)

	While stack entry level >= subxact depth
		if entry's state is SAVE:
			pop, restoring prior value
		else if level is 1 and entry's state is SET+LOCAL:
			pop, restoring *masked* value
		else if level is 1 and entry's state is SET:
			pop, discarding old value
		else if level is 1 and entry's state is LOCAL:
			pop, restoring prior value
		else if there is no entry of exactly level N-1:
			decrement entry's level, no other state changeprior
		else
			merge entries of level N-1 and N as specified below
	
	stack entry level >= subxact depthの間
		エントリの状態がSAVEの場合：
			ポップし、前の値をリストアします
		レベル1かつエントリ状態がSET+LOCALの場合:
			ポップし、*マスクされた*値をリストします
		レベル1かつエントリ状態がSETの場合:
			ポップし、古い値を廃棄します
		レベル1かつエントリ状態がLOCALの場合:
			ポップし、前の値をリストアします
		エントリがなく、レベルがN-1の場合
			エントリレベルをデクリメントし、他の状態変化はありまえん
		その他の場合
			エントリレベルのN-1,N,Nをマージし、以下に指定されます。

The merged entry will have level N-1 and prior = older prior, so easiest
to keep older entry and free newer.  There are 12 possibilities since
we already handled level N state = SAVE:
マージされたエントリはレベル N-1かつ前の値 = 古い前の値を持っています。そのため、
古いエントリを保持し、新しい値を解放することが最も簡単です。
我々は既にlevel N state = SAVEの状態扱うため、12の可能性があります。

N-1		N

SAVE		SET		discard top prior, set state SET
SAVE		LOCAL		discard top prior, no change to stack entry
SAVE		SET+LOCAL	discard top prior, copy masked, state S+L

SET		SET		discard top prior, no change to stack entry
SET		LOCAL		copy top prior to masked, state S+L
SET		SET+LOCAL	discard top prior, copy masked, state S+L

LOCAL		SET		discard top prior, set state SET
LOCAL		LOCAL		discard top prior, no change to stack entry
LOCAL		SET+LOCAL	discard top prior, copy masked, state S+L

SET+LOCAL	SET		discard top prior and second masked, state SET
SET+LOCAL	LOCAL		discard top prior, no change to stack entry
SET+LOCAL	SET+LOCAL	discard top prior, copy masked, state S+L


N-1		N

SAVE		SET		top priorを破棄し、状態をSETに設定する
SAVE		LOCAL		top priorを破棄し、スタックエントリへの変更を行わない
SAVE		SET+LOCAL	top priorを破棄し,コピーはマスクされ、状態はS+L

SET		SET		top priorを破棄し、スタックエントリへの変更を行わない
SET		LOCAL		top priorを破棄し、スタックエントリへの変更を行わない
SET		SET+LOCAL	top priorを破棄し,コピーはマスクされ、状態はS+L

LOCAL		SET		top priorを破棄し、状態をSETに設定する
LOCAL		LOCAL		top priorを破棄し、スタックエントリへの変更を行わない
LOCAL		SET+LOCAL	top priorを破棄し,コピーはマスクされ、状態はS+L

SET+LOCAL	SET		discard top prior and second masked, state SET
SET+LOCAL	LOCAL		top priorを破棄し、スタックエントリへの変更を行わない
SET+LOCAL	SET+LOCAL	top priorを破棄し,コピーはマスクされ、状態はS+L

RESET is executed like a SET, but using the reset_val as the desired new
value.  (We do not provide a RESET LOCAL command, but SET LOCAL TO DEFAULT
has the same behavior that RESET LOCAL would.)  The source associated with
the reset_val also becomes associated with the actual value.

If SIGHUP is received, the GUC code rereads the postgresql.conf
configuration file (this does not happen in the signal handler, but at
next return to main loop; note that it can be executed while within a
transaction).  New values from postgresql.conf are assigned to actual
variable, reset_val, and stacked actual values, but only if each of
these has a current source priority <= PGC_S_FILE.  (It is thus possible
for reset_val to track the config-file setting even if there is
currently a different interactive value of the actual variable.)

The check_hook, assign_hook and show_hook routines work only with the
actual variable, and are not directly aware of the additional values
maintained by GUC.


GUC Memory Handling
-------------------

String variable values are allocated with malloc/strdup, not with the
palloc/pstrdup mechanisms.  We would need to keep them in a permanent
context anyway, and malloc gives us more control over handling
out-of-memory failures.

We allow a string variable's actual value, reset_val, boot_val, and stacked
values to point at the same storage.  This makes it slightly harder to free
space (we must test whether a value to be freed isn't equal to any of the
other pointers in the GUC entry or associated stack items).  The main
advantage is that we never need to malloc during transaction commit/abort,
so cannot cause an out-of-memory failure there.

"Extra" structs returned by check_hook routines are managed in the same
way as string values.  Note that we support "extra" structs for all types
of GUC variables, although they are mainly useful with strings.


GUC and Null String Variables
-----------------------------

A GUC string variable can have a boot_val of NULL.  guc.c handles this
unsurprisingly, assigning the NULL to the underlying C variable.  Any code
using such a variable, as well as any hook functions for it, must then be
prepared to deal with a NULL value.

However, it is not possible to assign a NULL value to a GUC string
variable in any other way: values coming from SET, postgresql.conf, etc,
might be empty strings, but they'll never be NULL.  And SHOW displays
a NULL the same as an empty string.  It is therefore not appropriate to
treat a NULL value as a distinct user-visible setting.  A typical use
for a NULL boot_val is to denote that a value hasn't yet been set for
a variable that will receive a real value later in startup.

If it's undesirable for code using the underlying C variable to have to
worry about NULL values ever, the variable can be given a non-null static
initializer as well as a non-null boot_val.  guc.c will overwrite the
static initializer pointer with a copy of the boot_val during
InitializeGUCOptions, but the variable will never contain a NULL.
