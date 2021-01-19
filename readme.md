# シェーダーを埋め込むテスト

hlsl をアプリケーションに埋め込むためにuint8_t の配列にしてcpp ファイルにします。

cmake を利用しhlsl からcso に変換し、cso をcpp/h ファイルに変換します。

## hlsl -> cso

Visual Studio のhlsl コンパイラを利用します。
適切に変換するためシェーダータイプとエントリーポイントの関数名の指定が必要です。
set_source_files_properties() で指定しています。

変換処理は独立したライブラリプロジェクトとしていますが、c++ のソースコードを
含んでいないとライブラリの言語が特定できないとしてcmake 実行時にエラーとなります。
set_target_properties(target PROPERTIES LINTER_LANGUAGE CXX) で言語を指定しています。

## cso -> cpp, h

変換自体は簡単なコマンドラインプログラムで行っています。

変換処理は独立したカスタムターゲットとしています。
各ファイルの変換処理はカスタムコマンドをを登録しています。

元となるcso ファイルはcmake でプロジェクトを初期化する時点で存在しておらず
cmake 実行時にエラーとなります。
set_source_files_properties() でGENERATED true を指定することで
エラーを回避できます。
