# 项目信息
project(QCloud_IoT_SDK C)

# CMake 最低版本号要求
cmake_minimum_required(VERSION 3.5)

# #####################CONFIG BEGIN####################################
# 编译类型: release/debug
set(BUILD_TYPE "release")

# 代码抽取，ON表示根据配置抽取源码到ouput/qcloud_iot_c_sdk目录
set(EXTRACT_SRC OFF)

# 编译工具链
set(COMPILE_TOOLS "gcc")
set(PLATFORM "linux")

# set(COMPILE_TOOLS "MSVC")
# set(PLATFORM 	  "windows")

# 是否使能MQTT功能
set(FEATURE_MQTT_COMM_ENABLED ON)

# 是否使能设备动态注册
set(FEATURE_DEV_DYN_REG_ENABLED ON)

# 是否使能网关功能
set(FEATURE_GATEWAY_ENABLED ON)

# 是否使能网关子设备动态绑定解绑功能
set(FEATURE_GATEWAY_DYN_BIND_SUBDEV_ENABLED OFF)

# 是否使能网关本地联动功能
set(FEATURE_GATEWAY_AUTOMATION_ENABLED ON)

# 是否使能网关本地场景功能
set(FEATURE_GATEWAY_SCENE_ENABLED ON)

# 是否使能网关设备组功能
set(FEATURE_GATEWAY_GROUP_ENABLED ON)

# 是否使能多线程
set(FEATURE_MULTITHREAD_ENABLED OFF)

# 是否使能数据模板事件上报功能
set(FEATURE_EVENT_POST_ENABLED OFF)

# 是否使能数据模板行为功能
set(FEATURE_ACTION_ENABLED ON)

# 是否使能OTA固件升级总开关
set(FEATURE_OTA_COMM_ENABLED ON)

# 是否使能资源管理
set(FEATURE_RESOURCE_UPDATE_ENABLED ON)

# 是否使能文件管理
set(FEATURE_FILE_MANAGE_ENABLE ON)

# 是否使能语音识别功能
set(FEATURE_ASR_ENABLED OFF)

# 接入认证方式，使用证书认证：CERT；使用密钥认证：KEY
set(FEATURE_AUTH_MODE "KEY")

# 接入认证是否不使用TLS，证书方式必须选择使用TLS，密钥认证可选择不使用TLS
set(FEATURE_AUTH_WITH_NOTLS ON)

# 是否打开代码中获取设备信息功能，OFF时将从device_info.json中读取设备信息
set(FEATURE_DEBUG_DEV_INFO_USED OFF)

# 是否使用HTTPS下载固件
set(FEATURE_OTA_USE_HTTPS ON)

# 是否使能获取iot后台时间功能
set(FEATURE_SYSTEM_COMM_ENABLED OFF)

# 是否使能日志上报云端功能
set(FEATURE_LOG_UPLOAD_ENABLED OFF)

# 是否打开 WIFI 配网功能
set(FEATURE_WIFI_CONFIG_ENABLED OFF)

# 是否打开酷狗音乐功能
set(FEATURE_KGMUSIC_ENABLED ON)

# 是否打开QQ音乐功能
set(FEATURE_QQMUSIC_ENABLED ON)

# 是否打位置功能
set(FEATURE_LOCATION_ENABLED ON)

# 使用SDK AT组件实现通用TCP模组网络读写需要配置 --->begin
# 是否打开AT模组TCP功能
set(FEATURE_AT_TCP_ENABLED OFF)

# 设置AT模组设备（目前只支持esp8266，如果是其他模组请参照at_device_esp8266.c开发at_device驱动）
set(AT_DEVICE_NAME "esp8266")

# 是否打开AT模组中断接收功能
set(FEATURE_AT_UART_RECV_IRQ ON)

# 是否打开AT模组多线程功能
set(FEATURE_AT_OS_USED OFF)

# 是否打开AT模组调试功能
set(FEATURE_AT_DEBUG OFF)

# 使用SDK AT组件实现通用TCP模组网络读写需要配置 <---end

# #####################CONFIG END######################################

# 设置CMAKE使用编译工具及编译选项
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC -ffunction-sections -fdata-sections")

if((NOT COMPILE_TOOLS STREQUAL "gcc") AND(NOT COMPILE_TOOLS STREQUAL "MSVC"))
	# 用户自定义
	set(CMAKE_C_COMPILER ${COMPILE_TOOLS}) # 编译工具
	message(WARNING "User defined platform or compile tools!")
endif()

if(CMAKE_COMPILER_IS_GNUCC)
	if(${BUILD_TYPE} STREQUAL "release")
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Werror -Wall -Wno-error=sign-compare -Wno-error=format -std=gnu99 -Os -pthread") # 编译选项
	elseif(${BUILD_TYPE} STREQUAL "debug")
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -Werror -Wall -Wno-error=sign-compare -Wno-error=format -std=gnu99 -Os -pthread") # 编译选项
	endif()
endif()

if(CMAKE_COMPILER_IS_CLANG)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_WINSOCK_DEPRECATED_NO_WARNINGS -D_USE_32BIT_TIME_T)

	if(CMAKE_C_FLAGS MATCHES "/W[0-4]")
		string(REGEX REPLACE "/W[0-4]" "/W2" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
	else()
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /W2")
	endif()
endif()

# 解析配置
if(${BUILD_TYPE} STREQUAL "release")
	option(IOT_DEBUG "Enable IOT_DEBUG" OFF)
elseif(${BUILD_TYPE} STREQUAL "debug")
	option(IOT_DEBUG "Enable IOT_DEBUG" OFF)
else()
	message(FATAL_ERROR "INVAILD BUILD_TYPE:${BUILD_TYPE}!")
endif()

if(${FEATURE_AUTH_MODE} STREQUAL "KEY")
	option(AUTH_MODE_KEY "Enable AUTH_MODE_KEY" ON)
	option(AUTH_MODE_CERT "Enable AUTH_MODE_CERT" OFF)
elseif(${FEATURE_AUTH_MODE} STREQUAL "CERT" AND ${FEATURE_AUTH_WITH_NOTLS} STREQUAL "OFF")
	option(AUTH_MODE_KEY "Enable AUTH_MODE_KEY" OFF)
	option(AUTH_MODE_CERT "Enable AUTH_MODE_CERT" ON)
	option(AUTH_WITH_NOTLS "Enable AUTH_WITH_NOTLS" OFF)
else()
	message(FATAL_ERROR "INVAILD AUTH_MODE:${FEATURE_AUTH_MODE} WITH AUTH_WITH_NOTLS:${FEATURE_AUTH_WITH_NOTLS}!")
endif()

if(${FEATURE_OTA_COMM_ENABLED} STREQUAL "ON")
	option(OTA_MQTT_CHANNEL "Enable OTA_MQTT_CHANNEL" ON)
endif()

option(DEV_DYN_REG_ENABLED "Enable DEV_DYN_REG" ${FEATURE_DEV_DYN_REG_ENABLED})
option(GATEWAY_ENABLED "Enable GATEWAY" ${FEATURE_GATEWAY_ENABLED})
option(GATEWAY_DYN_BIND_SUBDEV_ENABLED "Enable GATEWAY_DYN_BIND_SUBDEV" ${FEATURE_GATEWAY_DYN_BIND_SUBDEV_ENABLED})
option(GATEWAY_AUTOMATION_ENABLED "Enable GATEWAY AUTOMATION" ${FEATURE_GATEWAY_AUTOMATION_ENABLED})
option(GATEWAY_SCENE_ENABLED "Enable GATEWAY SCENE" ${FEATURE_GATEWAY_SCENE_ENABLED})
option(GATEWAY_GROUP_ENABLED "Enable GATEWAY GROUP" ${FEATURE_GATEWAY_GROUP_ENABLED})
option(AUTH_WITH_NOTLS "Enable AUTH_WITH_NOTLS" ${FEATURE_AUTH_WITH_NOTLS})
option(EVENT_POST_ENABLED "Enable EVENT_POST" ${FEATURE_EVENT_POST_ENABLED})
option(ACTION_ENABLED "Enable ACTION" ${FEATURE_ACTION_ENABLED})
option(AT_TCP_ENABLED "Enable AT_TCP" ${FEATURE_AT_TCP_ENABLED})
option(DEBUG_DEV_INFO_USED "Enable DEBUG_DEV_INFO_USED" ${FEATURE_DEBUG_DEV_INFO_USED})
option(SYSTEM_COMM "Enable SYSTEM_COMM" ${FEATURE_SYSTEM_COMM_ENABLED})
option(LOG_UPLOAD "Enable LOG_UPLOAD" ${FEATURE_LOG_UPLOAD_ENABLED})
option(RESOURCE_UPDATE_ENABLED "Enable Resource update" ${FEATURE_RESOURCE_UPDATE_ENABLED})
option(FILE_MANAGE_ENABLED "Enable file manage" ${FEATURE_FILE_MANAGE_ENABLE})
option(ASR_ENABLED "Enable ASR" ${FEATURE_ASR_ENABLED})
option(WIFI_CONFIG_ENABLED "Enable WIFI CONFIG" ${FEATURE_WIFI_CONFIG_ENABLED})
option(MULTITHREAD_ENABLED "Enable Multithread" ${FEATURE_MULTITHREAD_ENABLED})
option(KGMUSIC_ENABLED "Enable KGMUSIC" ${FEATURE_KGMUSIC_ENABLED})
option(QQMUSIC_ENABLED "Enable QQMUSIC" ${FEATURE_QQMUSIC_ENABLED})
option(LOCATION_ENABLED "Enable Electric fence" ${FEATURE_LOCATION_ENABLED})

if(${FEATURE_AUTH_WITH_NOTLS} STREQUAL "ON")
	option(OTA_USE_HTTPS "Enable OTA_USE_HTTPS" OFF)
else()
	option(OTA_USE_HTTPS "Enable OTA_USE_HTTPS" ${FEATURE_OTA_USE_HTTPS})
endif()

if(${FEATURE_AUTH_WITH_NOTLS} STREQUAL "ON")
	option(OTA_USE_HTTPS "Enable OTA_USE_HTTPS" OFF)
else()
	option(OTA_USE_HTTPS "Enable OTA_USE_HTTPS" ${FEATURE_OTA_USE_HTTPS})
endif()

if(${FEATURE_ASR_ENABLED} STREQUAL "ON" AND ${FEATURE_OTA_USE_HTTPS} STREQUAL "OFF")
	message(FATAL_ERROR "Error! ASR NEED HTTPS ENABLE")
endif()

if(${FEATURE_GATEWAY_ENABLED} STREQUAL "ON")
	option(MULTITHREAD_ENABLED "Enable MULTITHREAD" ON)
else()
	option(MULTITHREAD_ENABLED "Enable MULTITHREAD" ${FEATURE_MULTITHREAD_ENABLED})
endif()

if(AT_TCP_ENABLED STREQUAL "ON")
	option(AT_UART_RECV_IRQ "Enable AT_UART_RECV_IRQ" ${FEATURE_AT_UART_RECV_IRQ})
	option(AT_OS_USED "Enable AT_UART_RECV_IRQ" ${FEATURE_AT_OS_USED})
	option(AT_DEBUG "Enable AT_DEBUG" ${FEATURE_AT_DEBUG})
else()
	option(AT_UART_RECV_IRQ "Enable AT_UART_RECV_IRQ" OFF)
	option(AT_OS_USED "Enable AT_UART_RECV_IRQ" OFF)
	option(AT_DEBUG "Enable AT_DEBUG" OFF)
endif()

configure_file(
	"${PROJECT_SOURCE_DIR}/tools/cmake_scripts/config.h.in"
	"${PROJECT_SOURCE_DIR}/include/config.h"
	@ONLY
)

# 头文件目录
include_directories(${PROJECT_SOURCE_DIR}/include
	${PROJECT_SOURCE_DIR}/include/exports
	${PROJECT_SOURCE_DIR}/platform/at_device/esp8266
	${PROJECT_SOURCE_DIR}/sdk_src/internal_inc
	${PROJECT_SOURCE_DIR}/external_libs/mbedtls/include
	${PROJECT_SOURCE_DIR}/external_libs/cJSON)

file(GLOB src_include ${PROJECT_SOURCE_DIR}/include/*.h)
file(GLOB src_inc_export ${PROJECT_SOURCE_DIR}/include/exports/*.h)
file(COPY ${src_include} DESTINATION ${PROJECT_SOURCE_DIR}/output/${BUILD_TYPE}/include)
file(COPY ${src_inc_export} DESTINATION ${PROJECT_SOURCE_DIR}/output/${BUILD_TYPE}/include/exports)

if(${EXTRACT_SRC} STREQUAL "ON" AND NOT DEFINED SAMPLE_ONLY)
	file(GLOB src_internal_inc ${PROJECT_SOURCE_DIR}/sdk_src/internal_inc/*.h)
	file(COPY ${src_include} DESTINATION ${PROJECT_SOURCE_DIR}/output/qcloud_iot_c_sdk/include)
	file(COPY ${src_inc_export} DESTINATION ${PROJECT_SOURCE_DIR}/output/qcloud_iot_c_sdk/include/exports)
	file(COPY ${src_internal_inc} DESTINATION ${PROJECT_SOURCE_DIR}/output/qcloud_iot_c_sdk/sdk_src/internal_inc)

	if(AT_TCP_ENABLED STREQUAL "ON")
		file(GLOB src_at_inc ${PROJECT_SOURCE_DIR}/platform/at_device/${AT_DEVICE_NAME}/*.h)
		file(COPY ${src_at_inc} DESTINATION ${PROJECT_SOURCE_DIR}/output/qcloud_iot_c_sdk/platform)
	endif()
endif()

# 添加源代码目录
if(NOT DEFINED SAMPLE_ONLY)
	add_subdirectory(${PROJECT_SOURCE_DIR}/sdk_src)
	add_subdirectory(${PROJECT_SOURCE_DIR}/platform)
	add_subdirectory(${PROJECT_SOURCE_DIR}/external_libs/mbedtls)

	if(FEATURE_WIFI_CONFIG_ENABLED STREQUAL "ON")
		add_subdirectory(${PROJECT_SOURCE_DIR}/external_libs/cJSON)
	endif()
endif()

add_subdirectory(${PROJECT_SOURCE_DIR}/samples)

if(SAMPLE_ONLY STREQUAL "ON")
	message(STATUS "Only compile samples!")
endif()

# 证书及设备信息
if(NOT DEFINED SAMPLE_ONLY)
	file(COPY ${PROJECT_SOURCE_DIR}/certs DESTINATION ${PROJECT_SOURCE_DIR}/output/${BUILD_TYPE}/bin)
	file(COPY ${PROJECT_SOURCE_DIR}/device_info.json DESTINATION ${PROJECT_SOURCE_DIR}/output/${BUILD_TYPE}/bin)

	if(ASR_ENABLED STREQUAL "ON")
		file(COPY ${PROJECT_SOURCE_DIR}/tools/test_file DESTINATION ${PROJECT_SOURCE_DIR}/output/${BUILD_TYPE}/bin)
	endif()
endif()
