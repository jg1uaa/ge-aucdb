# SPDX-License-Identifier: 0BSD

# 社会保険診療報酬支払基金にある、
# 医薬品マスター→医薬品の全件マスター→全件分ファイルのURL
URL = https://www.ssk.or.jp/seikyushiharai/tensuhyo/kihonmasta/kihonmasta_04.files/y_ALL20231018.zip

# URLから操作対象となるファイル名を生成
ZIPFILE = $(lastword $(shell echo $(URL) | sed 's/\// /g'))
CSVFILE = $(ZIPFILE:.zip=.csv)

# 出力ファイル名
OUTFILE=template.csv

# 項目
FIELD = \# code subcode name AUC(hr) AUC-unit Cmax-unit
FIELDx = n AUC-mean AUC-SD Cmax-mean Cmax-SD tmax-mean tmax-SD t1/2-mean t1/2-SD
FIELD += $(FIELDx) $(addsuffix :std,$(FIELDx))
FIELD += remarks

all: $(OUTFILE)

clean:
	$(RM) $(ZIPFILE) $(CSVFILE) $(OUTFILE)

$(ZIPFILE):
	wget $(URL)

$(CSVFILE): $(ZIPFILE)
	unzip -x $(ZIPFILE) $(CSVFILE)

$(OUTFILE): $(CSVFILE)
	echo "$(FIELD)" | sed 's/ /,/g' > $(OUTFILE)
	nkf -w $(CSVFILE) | \
		sed 's/,/ /g' | sed 's/"//g' | \
		awk '{if ($$17 == 1 && $$28 == 1){print $$5, $$3}}' | \
		sort | \
		awk '{printf("#,%s,0,%s\n", $$2, $$1)}' >> $(OUTFILE)

# 出力ファイルの生成手順は以下の通り：
#   1)見出し（雛形）の出力（出力ファイルの初期化も兼ねる）
#   2)マスターはShift-JISを使っているため、UTF-8に変換
#   3)カンマ区切りをスペース区切りに変換し項目の前後にあるダブルクォートを除去
#   4-1)後発品($17)かつ内用薬($28)のみ出力対象とする
#   4-2)名称順にソートを行うため漢字名称($5)→医薬品コード($3)の順に並べ替え
#   5)ソート処理
#   6-1)デフォルトはコメントアウトとするための#とサブコードを付加
#   6-2)医薬品コード→漢字名称に出力順を戻し、カンマ区切りCSVの形に整えて出力
#
# ※医薬品マスターの仕様は
#   https://www.ssk.or.jp/seikyushiharai/tensuhyo/kihonmasta/index.html
#   にある本編とファイルレイアウトを参照

# 各項目の定義
#   #（最初の項目）
#	 0もしくは#。0はそのエントリが有効、#は無効（コメントアウト）。
#   code
#	医薬品マスターに記された医薬品コード。
#   subcode
#	同一の医薬品コードに複数のエントリが存在する場合におけるサブコード。
#	0から始まる正の整数とする。実際のユースケースとしてよく使われるものを
#	0側に据える（例：OD剤における水あり投与/水なし投与の場合、
#	水あり投与を0、水なし投与を1とする）。
#	subcodeを使用し、codeに対する複数のエントリを記述する場合は
#	remarks（後述）にエントリの詳細を記載すること。
#   name	
#	医薬品マスターに記された漢字名称。
#   AUC(hr)
#	AUCの時間範囲（時間）。例）AUC0-24(hr)の場合は24
#   AUC-unit
#	AUCの単位。例）ng*hr/mL  ※ng･hr/mLとはしない
#   Cmax-unit
#	Cmaxの単位。例）ng/mL
#   n, n:std
#	対象製剤(n)もしくは標準製剤(n:std)のサンプル数。
#	これ以降の項目において、:stdの付いているものは標準製剤を示す。
#   AUC-mean, AUC-mean:std
#	AUCの平均値
#   AUC-SD, AUC-SD:std
#	AUCの標準偏差
#   Cmax-mean, Cmax-mean:std
#	Cmaxの平均値
#   Cmax-SD, Cmax-SD:std
#	Cmaxの標準偏差
#   tmax-mean, tmax-mean:std
#	tmaxの平均値
#   tmax-SD, tmax-SD:std
#	tmaxの標準偏差
#   t1/2-mean, t1/2-mean:std
#	t1/2の平均値
#   t1/2-SD, t1/2-SD:std
#	t1/2の標準偏差
#   remarks
#	備考。subcodeで複数のエントリを用意する場合はその理由を記載する。
#	未作業のデータとの区別を容易にするため、AUC等の情報が無い場合は
#	無効エントリとした上で「情報なし」を記す。
#	remarksにおいてよく使われるキーワードは、共通化することが望ましい。
