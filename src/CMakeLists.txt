add_executable(app main.cpp)

file(GLOB_RECURSE pshaders *_ps.hlsl)
file(GLOB_RECURSE vshaders *_vs.hlsl)

add_library(compile_shaders STATIC ${vshaders} ${pshaders})
set_target_properties(compile_shaders PROPERTIES LINKER_LANGUAGE CXX)

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
set(shaders "${pshaders};${vshaders}")
list(FILTER shaders INCLUDE REGEX "\/embeded\/")
foreach(shader IN ITEMS ${shaders})
    string(REGEX REPLACE "\.hlsl$" ".cso" cso ${shader})
    message(STATUS "${cso}")
    set_source_files_properties(
        ${shader}
        PROPERTIES
            VS_SHADER_OBJECT_FILE_NAME ${cso}
    )
    list(APPEND csos ${cso})
endforeach()
message(STATUS "${csos}")
#add_custom_command(TARGET shaders)

add_custom_target(embed_shaders
    BYPRODUCTS ${csos}
)

