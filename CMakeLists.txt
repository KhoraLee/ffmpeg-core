if (NOT DEFINED FFMPEG_CORE_NAME)
	set(FFMPEG_CORE_NAME ffmpeg)
endif()

add_library(${FFMPEG_CORE_NAME} INTERFACE)

target_include_directories(${FFMPEG_CORE_NAME} INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/include")

if (WIN32 AND NOT MINGW)
	set(FFMPEG_PREBUILTS_NAME "ffmpeg-windows")
elseif (APPLE)
	set(FFMPEG_PREBUILTS_NAME "ffmpeg-macOS")
elseif (UNIX)
	set(FFMPEG_PREBUILTS_NAME "ffmpeg-linux")
else ()
	message(FATAL_ERROR "Unsupported OS.")
endif ()

if(NOT DEFINED ARCHITECTURE)
	message(FATAL_ERROR "ARCHITECTURE variable is not set up")
elseif (ARCHITECTURE STREQUAL "x86_64")
	set(FFMPEG_PREBUILTS_NAME "${FFMPEG_PREBUILTS_NAME}-x64.zip")
elseif(ARCHITECTURE STREQUAL "arm64")
	set(FFMPEG_PREBUILTS_NAME "${FFMPEG_PREBUILTS_NAME}-arm64.zip")
else ()
	if (APPLE)
		set(FFMPEG_PREBUILTS_NAME "${FFMPEG_PREBUILTS_NAME}-universal.zip")
	else ()
		message(FATAL_ERROR "Unsupported architecture.")
	endif()
endif()

if(NOT EXISTS "${CMAKE_BINARY_DIR}/external/ffmpeg.zip")
	message(STATUS "Downloading FFMPEG prebuilts...")
	file(DOWNLOAD https://github.com/Vita3K/ffmpeg-core/releases/download/continuous/${FFMPEG_PREBUILTS_NAME}
		"${CMAKE_BINARY_DIR}/external/ffmpeg.zip" SHOW_PROGRESS)
endif()

if(NOT EXISTS "${CMAKE_BINARY_DIR}/external/ffmpeg/lib")
	file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/external/ffmpeg/lib")
	execute_process(COMMAND ${CMAKE_COMMAND} -E tar xzf "${CMAKE_BINARY_DIR}/external/ffmpeg.zip"
		WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/external/ffmpeg/lib")
endif()

set(LIB_PREFIX "lib")
set(LIB_EXT "a")
if (WIN32 AND NOT MINGW)
	set(LIB_PREFIX "")
	set(LIB_EXT "lib")
	target_link_libraries(${FFMPEG_CORE_NAME} INTERFACE
		"psapi;strmiids;uuid;oleaut32;shlwapi;ws2_32;ole32;user32;bcrypt")
elseif (APPLE)
	target_link_libraries(${FFMPEG_CORE_NAME} INTERFACE
		"-framework CoreServices" "-framework CoreFoundation" "-framework AudioUnit"
		"-framework AudioToolbox" "-framework CoreAudio" "-framework CoreMedia"
		"-framework VideoToolbox" "-framework CoreVideo" "-framework Security")
endif ()

target_link_libraries(${FFMPEG_CORE_NAME} INTERFACE
	"${CMAKE_BINARY_DIR}/external/ffmpeg/lib/${LIB_PREFIX}avformat.${LIB_EXT}"
	"${CMAKE_BINARY_DIR}/external/ffmpeg/lib/${LIB_PREFIX}avcodec.${LIB_EXT}"
	"${CMAKE_BINARY_DIR}/external/ffmpeg/lib/${LIB_PREFIX}swscale.${LIB_EXT}"
	"${CMAKE_BINARY_DIR}/external/ffmpeg/lib/${LIB_PREFIX}avutil.${LIB_EXT}"
	"${CMAKE_BINARY_DIR}/external/ffmpeg/lib/${LIB_PREFIX}avfilter.${LIB_EXT}"
	"${CMAKE_BINARY_DIR}/external/ffmpeg/lib/${LIB_PREFIX}swresample.${LIB_EXT}")