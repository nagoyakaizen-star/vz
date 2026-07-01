#!/bin/bash

TARGET_DIR="./VZEditor"
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
PROJECT_NAME           = "VZ editor Assembly Source Code Analysis"
OUTPUT_DIRECTORY       = ${OUTPUT_DIR}
OUTPUT_LANGUAGE        = Japanese
INPUT                  = ${TARGET_DIR}
RECURSIVE              = YES
EXTRACT_ALL            = YES
EXTRACT_PRIVATE        = YES
EXTRACT_STATIC         = YES

# ★アセンブラ対応設定
FILE_PATTERNS          = *.c *.h *.s *.S *.asm *.ASM *.INC *.LNK *.MAK *.OMK *.BAT
EXTENSION_MAPPING      = s=C S=C asm=C ASM=C INC=H
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
