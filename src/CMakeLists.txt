# hlsl を収集
file(GLOB_RECURSE pshaders *_ps.hlsl)
file(GLOB_RECURSE vshaders *_vs.hlsl)

# シェーダをコンパイルするプロジェクト
add_library(compile_shaders STATIC ${vshaders} ${pshaders})
set_target_properties(compile_shaders PROPERTIES LINKER_LANGUAGE CXX)
# hlsl コンパイラの設定
set_source_files_properties(
    ${vshaders}
    PROPERTIES
        VS_SHADER_TYPE Vertex
        VS_SHADER_MODEL 4.0
        VS_SHADER_ENTRYPOINT main
)
set_source_files_properties(
    ${pshaders}
    PROPERTIES
        VS_SHADER_TYPE Pixel
        VS_SHADER_MODEL 4.0
        VS_SHADER_ENTRYPOINT main
)
# 埋め込みのシェーダ
set(shaders "${pshaders};${vshaders}")
list(FILTER shaders INCLUDE REGEX "\/embeded\/")
foreach(shader IN ITEMS ${shaders})
    string(REGEX REPLACE "\.hlsl$" ".cso" cso ${shader})
    # 出力ファイル名の設定
    set_source_files_properties(
        ${shader}
        PROPERTIES
            VS_SHADER_OBJECT_FILE_NAME ${cso}
    )
    # 出力ファイルにGENERATED 設定
    set_source_files_properties(
        ${cso}
        PROPERTIES
            GENERATED true
    )
    list(APPEND csos ${cso})
endforeach()

# コンパイルされたシェーダを埋め込み用のソースコードに変換
add_custom_target(embeded_shaders SOURCES ${csos})
foreach (cso IN ITEMS ${csos})
    string(REGEX REPLACE "\.cso$" ".cpp" cso_cpp ${cso})
    string(REGEX REPLACE "\.cso$" ".h" cso_hpp ${cso})
    add_custom_command(OUTPUT ${cso_cpp} ${cso_hpp}
        COMMAND $<TARGET_FILE:bin2cpp> ${cso}
        MAIN_DEPENDENCY ${cso})
    list(APPEND embeded_files ${cso_cpp})
    list(APPEND embeded_files ${cso_hpp})
endforeach()
# 出力ファイルにGENERATED 設定
set_source_files_properties(
    ${embeded_files}
    PROPERTIES
        GENERATED true
)
# アプリケーション
add_executable(app main.cpp ${embeded_files})
