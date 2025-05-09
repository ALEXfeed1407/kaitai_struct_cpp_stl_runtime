cmake_minimum_required (VERSION 3.11)
project (kaitai_struct_qt_runtime CXX)
enable_testing()

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)
option(BUILD_TESTS "Build tests" ON)

set (CMAKE_INCLUDE_CURRENT_DIR ON)

find_package(ZLIB)
find_package(Iconv)

if(NOT DEFINED QtPkgList)
    set(QtPkgList "Qt5;Qt6")
endif()

foreach(QtPkg ${QtPkgList})
  find_package(QT NAMES ${QtPkg} HINTS "${QT_ROOT_DIR}" CONFIG REQUIRED COMPONENTS Widgets)
  if (QT_FOUND)
    break()
  endif()
endforeach()

find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Core REQUIRED)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON) # must have if lib exports QObject-based classes
set(CMAKE_AUTORCC ON)

set (HEADERS
    kaitai-qt/kaitaistream.h
    kaitai-qt/kaitaistruct.h
    kaitai-qt/exceptions.h
    kaitai-qt/custom_decoder.h
)

set (SOURCES
    kaitai-qt/kaitaistream.cpp
    kaitai-qt/kaitaistruct.cpp
)

set(STRING_ENCODING_TYPE "ICONV" CACHE STRING "Set the way strings have to be encoded (ICONV|WIN32API|NONE|...)")

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

add_library (${PROJECT_NAME} ${HEADERS} ${SOURCES})
set_property(TARGET ${PROJECT_NAME} PROPERTY PUBLIC_HEADER ${HEADERS})
target_include_directories(${PROJECT_NAME} INTERFACE ${PROJECT_SOURCE_DIR})

if (ZLIB_FOUND)
    target_link_libraries(${PROJECT_NAME} PRIVATE ZLIB::ZLIB Qt${QT_VERSION_MAJOR}::Core)
    target_compile_definitions(${PROJECT_NAME} PUBLIC -DKS_ZLIB)
endif()

if(Iconv_FOUND)
    target_link_libraries(${PROJECT_NAME} PRIVATE Iconv::Iconv Qt${QT_VERSION_MAJOR}::Core)
endif()

include(Common.cmake)

install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
    PUBLIC_HEADER DESTINATION include/kaitai-qt
)

# Add the tests
if(BUILD_TESTS)
    add_subdirectory(tests)
endif()

add_subdirectory(packaging)
