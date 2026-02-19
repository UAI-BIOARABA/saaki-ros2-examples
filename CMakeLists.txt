cmake_minimum_required(VERSION 3.5)
project(saaki_ros2_examples)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

include_directories(include include/common include/nlohmann)
link_directories(src)

# G1 usa mensajes tanto de unitree_go (mando) como de unitree_hg (low state)
set (
  DEPENDENCY_LIST
  unitree_go
  unitree_hg
  unitree_api
  rclcpp
  std_msgs
  rosbag2_cpp
)

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(unitree_go REQUIRED)
find_package(unitree_hg REQUIRED)
find_package(unitree_api REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rosbag2_cpp REQUIRED)
find_package(Eigen3 REQUIRED)

# ==========================================
# 1. DEFINICIÓN DE EJECUTABLES (Solo G1)
# ==========================================

# Scripts comunes que sí usa el G1
add_executable(read_low_state_hg src/read_low_state_hg.cpp)
add_executable(read_wireless_controller src/read_wireless_controller.cpp)

# Scripts específicos del G1
add_executable(g1_low_level_example src/g1/lowlevel/g1_low_level_example.cpp src/common/motor_crc_hg.cpp)
add_executable(g1_arm_sdk_dds_example src/g1/high_level/g1_arm_sdk_dds_example.cpp src/common/motor_crc.cpp)
target_compile_features(g1_arm_sdk_dds_example PRIVATE cxx_std_20)

add_executable(g1_arm_action_example src/g1/high_level/g1_arm_action_example.cpp src/common/motor_crc.cpp)
target_compile_features(g1_arm_action_example PRIVATE cxx_std_20)

add_executable(g1_dex3_example src/g1/dex3/g1_dex3_example.cpp src/common/motor_crc.cpp)
target_compile_features(g1_dex3_example PRIVATE cxx_std_20)

add_executable(g1_loco_client_example src/g1/high_level/loco_client_example.cpp src/common/motor_crc.cpp)
target_compile_features(g1_loco_client_example PRIVATE cxx_std_20)

add_executable(g1_ankle_swing_example src/g1/lowlevel/g1_ankle_swing_example.cpp src/common/motor_crc_hg.cpp)
target_compile_features(g1_ankle_swing_example PRIVATE cxx_std_20)

add_executable(g1_audio_client_example src/g1/audio_client/g1_audio_client_example.cpp)
target_compile_features(g1_audio_client_example PRIVATE cxx_std_20)

# ==========================================
# 2. ENLACE DE DEPENDENCIAS
# ==========================================

ament_target_dependencies(read_low_state_hg ${DEPENDENCY_LIST})
ament_target_dependencies(read_wireless_controller ${DEPENDENCY_LIST})
ament_target_dependencies(g1_low_level_example ${DEPENDENCY_LIST})
ament_target_dependencies(g1_arm_sdk_dds_example ${DEPENDENCY_LIST})
ament_target_dependencies(g1_arm_action_example ${DEPENDENCY_LIST})

ament_target_dependencies(g1_dex3_example ${DEPENDENCY_LIST})
target_include_directories(g1_dex3_example PRIVATE ${EIGEN3_INCLUDE_DIR})

ament_target_dependencies(g1_loco_client_example ${DEPENDENCY_LIST})
ament_target_dependencies(g1_ankle_swing_example ${DEPENDENCY_LIST})
ament_target_dependencies(g1_audio_client_example ${DEPENDENCY_LIST})

# ==========================================
# 3. EJEMPLO OPCIONAL DUAL ARM
# ==========================================
find_package(yaml-cpp QUIET)
if(yaml-cpp_FOUND)
    if (${yaml-cpp_VERSION} VERSION_GREATER_EQUAL "0.6")
        add_executable(g1_dual_arm_example src/g1/lowlevel/g1_dual_arm_example.cpp src/common/motor_crc_hg.cpp)
        target_compile_features(g1_dual_arm_example PRIVATE cxx_std_20)
        ament_target_dependencies(g1_dual_arm_example yaml-cpp ${DEPENDENCY_LIST})
    endif()
endif()

# ==========================================
# 4. INSTALACIÓN (Rutas corregidas)
# ==========================================
install(TARGETS 
        read_low_state_hg
        read_wireless_controller
        g1_low_level_example
        g1_arm_sdk_dds_example
        g1_arm_action_example
        g1_dex3_example
        g1_loco_client_example
        g1_ankle_swing_example
        g1_audio_client_example
        DESTINATION lib/${PROJECT_NAME}
)

if(yaml-cpp_FOUND)
    if (${yaml-cpp_VERSION} VERSION_GREATER_EQUAL "0.6")
      install(TARGETS g1_dual_arm_example DESTINATION lib/${PROJECT_NAME})
    endif()
endif()

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
