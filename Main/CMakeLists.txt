add_executable(MainApp WIN32
    src/main.cpp
)

target_link_libraries(MainApp
    PRIVATE
        Qt5::Core
        Qt5::Widgets
        DynamicCreateDemo::Core
)

add_dependencies(MainApp DemoPlugin)
