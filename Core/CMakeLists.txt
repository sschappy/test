add_library(Core SHARED
    src/DynamicCreate.cpp
)

add_library(DynamicCreateDemo::Core ALIAS Core)

target_include_directories(Core
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)

target_compile_definitions(Core
    PRIVATE DYNAMIC_CREATE_CORE_LIBRARY
)
