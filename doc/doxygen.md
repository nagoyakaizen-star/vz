# Q1 アセンブラのソースをdoxygen + graphvizしたいときは何を変更すればいいですか。
A1 by Gemini
Doxygenはデフォルトではアセンブラ（`.s` や `.asm` などのファイル）を直接解析する機能を持っていません。そのため、アセンブラのソースを Doxygen + Graphviz で可視化したい場合は、「アセンブラをC言語のヘッダー風に変換するフィルター」を通すように設定を変更する必要があります。

先ほどのスクリプトの `cat << EOF >> temp_Doxyfile` の中身（設定を追記している部分）を、以下のように変更してください。

---

### 設定ファイル（Doxyfile）への変更・追加内容

Doxyfile の設定に、以下の **4つの変更点** を組み込みます。

```ini
# 1. アセンブラの拡張子（.s や .asm）を解析対象に含める
FILE_PATTERNS          = *.c *.h *.s *.S *.asm

# 2. アセンブラファイルをC言語のファイルとして認識させる
EXTENSION_MAPPING      = s=C S=C asm=C

# 3. Doxygen標準のC入力フィルター（doxyass）を有効にする
FILTER_PATTERNS        = "*.s=doxyass" "*.S=doxyass" "*.asm=doxyass"

# 4. インラインアセンブラやマクロ内の関数関係を拾いやすくする（任意）
OPTIMIZE_OUTPUT_FOR_C  = YES
CLANG_ASSISTED_PARSING = NO

```

### スクリプト全体（修正版）

既存のスクリプトをそのまま置き換えられるように、アセンブラ対応版の全コードを提示します。

```bash
#!/bin/bash

TARGET_DIR="./pcc"
OUTPUT_DIR="./doxygen_output"

if ! command -v doxygen &> /dev/null; then
    echo "エラー: doxygen がインストールされていません。"
    exit 1
fi

if ! command -v dot &> /dev/null; then
    echo "エラー: graphviz (dot) がインストールされていません。"
    exit 1
fi

if [ ! -d "$TARGET_DIR" ]; then
    echo "エラー: 対象の $TARGET_DIR フォルダが存在しません。"
    exit 1
fi

echo "--- Doxygen 設定ファイルを生成中 (アセンブラ対応版) ---"
doxygen -g temp_Doxyfile > /dev/null

cat << EOF >> temp_Doxyfile
# 基本設定
PROJECT_NAME           = "PCC Assembly Source Code Analysis"
OUTPUT_DIRECTORY       = ${OUTPUT_DIR}
OUTPUT_LANGUAGE        = Japanese
INPUT                  = ${TARGET_DIR}
RECURSIVE              = YES
EXTRACT_ALL            = YES
EXTRACT_PRIVATE        = YES
EXTRACT_STATIC         = YES

# ★アセンブラ対応設定
FILE_PATTERNS          = *.c *.h *.s *.S *.asm
EXTENSION_MAPPING      = s=C S=C asm=C
FILTER_PATTERNS        = "*.s=doxyass" "*.S=doxyass" "*.asm=doxyass"
OPTIMIZE_OUTPUT_FOR_C  = YES

# Graphviz (dot) 連動設定
HAVE_DOT               = YES
DOT_NUM_THREADS        = 4
UML_LOOK               = YES

# 生成するグラフの選択
CLASS_DIAGRAMS         = YES
INCLUDE_GRAPH          = YES
INCLUDED_BY_GRAPH      = YES
CALL_GRAPH             = YES
CALLER_GRAPH           = YES
DIRECTORY_GRAPH        = YES

# インタラクティブSVG設定
INTERACTIVE_SVG        = YES
DOT_IMAGE_FORMAT       = svg
DOT_GRAPH_MAX_NODES    = 200
EOF

echo "--- Doxygen 解析を実行中 ---"
doxygen temp_Doxyfile
rm temp_Doxyfile

echo "--------------------------------------------------"
echo "完了しました！"
echo " ${OUTPUT_DIR}/html/index.html"
echo "--------------------------------------------------"

```

---

### 💡 きれいにグラフ化するための注意点（重要）

Doxygen標準の `doxyass` フィルターは、アセンブラソース内のグローバルラベル（関数名に相当するもの）を関数として認識します。

しかし、アセンブラの構文はアーキテクチャ（ARM, x86, RISC-Vなど）やコンパイラによってバラバラなため、もし自動で関数呼び出しグラフ（CALL_GRAPH）が生成されない場合は、ソースコード側に以下のような **Doxygen用のコメント** を手動で少し書き足してあげる必要があります。

```assembly
;/**
; * @brief データの処理を行う関数
; * @details この中で sub_routine を呼び出します
; */
.global main_process
main_process:
    push {lr}
    
    ; Doxygenはアセンブラの bl や call 命令を自動解析できないことがあるため、
    ; グラフに出したい場合は明示的にコメントで書くと確実です。
    ; @callgraph
    bl sub_routine
    
    pop {pc}

```

まずは一度そのままスクリプトを実行してみて、関数やラベルがどのように抽出されるか確認してみてください。
