cmake_minimum_required(VERSION 3.10)

project(webrtc-plugin)

# Common source files
file(GLOB sources
    *.cpp
    *.h
    GraphicsDevice/*.h
    GraphicsDevice/*.cpp
    Codec/*.h
    Codec/*.cpp
)

# Apple/Unix specific source files
if(APPLE)
    file(GLOB append_source
        GraphicsDevice/Metal/*.h
        GraphicsDevice/Metal/*.mm
        Codec/SoftwareCodec/*.h
        Codec/SoftwareCodec/*.cpp
        #Codec/VideoToolbox/VTEncoderMetal.mm
        #Codec/VideoToolbox/VTEncoderMetal.h
    )

    list (APPEND sources ${append_source})
elseif(UNIX)
    file(GLOB append_source
        GraphicsDevice/OpenGL/*.h
        GraphicsDevice/OpenGL/*.cpp
        GraphicsDevice/Vulkan/*.h
        GraphicsDevice/Vulkan/*.cpp
        GraphicsDevice/Vulkan/Cuda/*.h
        GraphicsDevice/Vulkan/Cuda/*.cpp
        Codec/NvCodec/NvEncoder.cpp
        Codec/NvCodec/NvEncoder.h
        Codec/NvCodec/NvEncoderGL.cpp
        Codec/NvCodec/NvEncoderGL.h
        Codec/NvCodec/NvEncoderCuda.cpp
        Codec/NvCodec/NvEncoderCuda.h
    )
    list (APPEND sources ${append_source})
endif()



add_plugin(webrtc
  SOURCES ${sources}
  PLUGINS_DIR ${CMAKE_SOURCE_DIR}/../Runtime/Plugins/x86_64
)

if(MSVC)
  # Force to always compile with W4
  if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
    string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
  endif()
elseif(APPLE OR UNIX)
  message(STATUS "macos or linux")
  target_compile_options(webrtc PRIVATE -Wall -Wextra -Wno-unknown-pragmas -Wno-unused-parameter -Wno-missing-field-initializers -Wno-long-long)
  target_compile_features(webrtc PRIVATE cxx_std_14)
endif()

if(APPLE)
    EXECUTE_PROCESS( COMMAND uname -m COMMAND tr -d '\n' OUTPUT_VARIABLE ARCHITECTURE )
    message( STATUS "Architecture: ${ARCHITECTURE}" )
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-lto -fno-rtti")
    set(CMAKE_C_FLAGS "-x objective-c")
    set(CMAKE_CXX_FLAGS "-x objective-c++")
    set(PROJECT_BINARY_DIR "../../Runtime/Plugins/${ARCHITECTURE}")

    target_compile_definitions(webrtc
        PRIVATE
          WEBRTC_MAC
          WEBRTC_POSIX
    )
    target_link_libraries(webrtc
        PRIVATE
        ${WEBRTC_LIBRARIES}
        ${OPENGL_LIBRARIES}
        ${FRAMEWORK_LIBS}
    )
    set_target_properties( webrtc PROPERTIES LIBRARY_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR})
    set_target_properties( webrtc PROPERTIES LIBRARY_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR})
elseif(UNIX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-lto -fno-rtti -stdlib=libc++")

    target_compile_definitions(webrtc
    PRIVATE
        WEBRTC_LINUX
        WEBRTC_POSIX
    )
    target_link_libraries(webrtc
        PRIVATE
        ${WEBRTC_LIBRARIES}
        ${OPENGL_opengl_LIBRARY}
        ${Vulkan_LIBRARY}
        cuda
    )
    target_include_directories(webrtc
        PRIVATE
        ${CUDA_INCLUDE_DIRS}
    )
endif()

target_include_directories(webrtc
  PRIVATE
    .
    ${CMAKE_SOURCE_DIR}/unity/include
    ${WEBRTC_INCLUDE_DIR}
    ${OPENGL_INCLUDE_DIR}
)
