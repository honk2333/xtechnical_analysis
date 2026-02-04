cmake_minimum_required(VERSION 3.10)
project(xtechnical_analysis)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 添加头文件搜索路径
include_directories(include)

# 检查是否存在Eigen库（用于ssa.cpp和其他依赖它的文件）
find_package(Eigen3 QUIET)
if(Eigen3_FOUND)
    include_directories(${EIGEN3_INCLUDE_DIR})
    message(STATUS "Found Eigen3: ${EIGEN3_INCLUDE_DIR}")
else()
    message(WARNING "Eigen3 not found. Some tests may fail to compile.")
    # 定义一个宏，用于条件编译
    add_definitions(-DNO_EIGEN)
endif()

# 测试文件列表
set(TEST_FILES
    code_blocks/test/body_filter.cpp
    code_blocks/test/trend_direction_force_index.cpp
    code_blocks/test/super_trend.cpp
    code_blocks/test/period_stats.cpp
    code_blocks/test/cluster_shaper.cpp
    code_blocks/test/cci.cpp
    code_blocks/test/atr.cpp
    code_blocks/test/fractals.cpp
    code_blocks/test/ssa.cpp
)

# 为每个测试文件创建可执行目标
foreach(TEST_FILE ${TEST_FILES})
    # 从文件路径中提取目标名称
    get_filename_component(TEST_NAME ${TEST_FILE} NAME_WE)
    
    # 创建可执行目标
    add_executable(${TEST_NAME} ${TEST_FILE})
    
    # 对于需要特殊处理的测试文件，可以在这里添加额外的配置
    if(${TEST_NAME} STREQUAL "ssa")
        if(NOT Eigen3_FOUND)
            # 如果没有找到Eigen库，禁用ssa测试
            set_target_properties(${TEST_NAME} PROPERTIES EXCLUDE_FROM_ALL TRUE)
        endif()
    elseif(${TEST_NAME} STREQUAL "cluster_shaper")
        # 使用vcpkg安装的OpenCV
        set(OpenCV_DIR /home/wanghk/code/vcpkg/installed/x64-linux/share/opencv4)
        find_package(OpenCV REQUIRED)
        if(OpenCV_FOUND)
            # 添加包含路径和链接库
            include_directories(${OpenCV_INCLUDE_DIRS})
            # 添加libjpeg-turbo的包含路径
            include_directories(/home/wanghk/code/vcpkg/installed/x64-linux/include)
            # 添加libpng的包含路径
            include_directories(/home/wanghk/code/vcpkg/installed/x64-linux/include)
            # 链接OpenCV库和所有需要的依赖库
            target_link_libraries(${TEST_NAME} 
                ${OpenCV_LIBS} 
                /home/wanghk/code/vcpkg/installed/x64-linux/lib/libjpeg.a
                /home/wanghk/code/vcpkg/installed/x64-linux/lib/libpng16.a
                /home/wanghk/code/vcpkg/installed/x64-linux/lib/libz.a
            )
            message(STATUS "Found OpenCV: ${OpenCV_VERSION}")
        else()
            # 如果找不到OpenCV，禁用cluster_shaper测试
            set_target_properties(${TEST_NAME} PROPERTIES EXCLUDE_FROM_ALL TRUE)
            message(WARNING "OpenCV not found. cluster_shaper test will be disabled.")
        endif()
    endif()
endforeach()

# 添加一个目标来编译所有测试
add_custom_target(all_tests)
foreach(TEST_FILE ${TEST_FILES})
    get_filename_component(TEST_NAME ${TEST_FILE} NAME_WE)
    add_dependencies(all_tests ${TEST_NAME})
endforeach()

message(STATUS "CMake configuration completed successfully!")
message(STATUS "To compile all tests, run: cmake --build .")
message(STATUS "To run a specific test, run: ./<test_name>")
