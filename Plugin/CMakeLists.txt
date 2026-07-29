add_library(DemoPlugin SHARED
    src/DemoPage.cpp
    src/DemoPage.h
    src/PluginEntry.cpp
    src/PluginEntry.h
)

target_link_libraries(DemoPlugin
    PRIVATE
        Qt5::Core
        Qt5::Widgets
        DynamicCreateDemo::Core
)

set_target_properties(DemoPlugin PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARIES_DIR}"
)
