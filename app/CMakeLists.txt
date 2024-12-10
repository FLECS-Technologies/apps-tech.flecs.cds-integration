cmake_minimum_required(VERSION 3.21)

include(flecs-build/cmake/flecs-build.cmake)

project(flecs-tb)

add_subdirectory(cds-plugin)

# install cds-plugin
install(TARGETS
    CmpTowerBridge
    LIBRARY DESTINATION CmpTowerBridge
)
