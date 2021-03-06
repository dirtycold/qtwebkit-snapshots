cmake_minimum_required(VERSION 2.8.12)
include(WebKitCommon)

# TODO: Unify usage of prefix headers and PCH with WebCore and WebKit2
macro(ADD_WEBKIT_PREFIX_HEADER _target _header)
    get_target_property(OLD_COMPILE_FLAGS ${_target} COMPILE_FLAGS)
    if (${OLD_COMPILE_FLAGS} STREQUAL "OLD_COMPILE_FLAGS-NOTFOUND")
        set(OLD_COMPILE_FLAGS "")
    endif ()
    set_target_properties(${_target} PROPERTIES COMPILE_FLAGS "${OLD_COMPILE_FLAGS} -include ${_header}")
endmacro()

set(WebKit_SOURCES
    Storage/StorageAreaImpl.cpp
    Storage/StorageAreaSync.cpp
    Storage/StorageNamespaceImpl.cpp
    Storage/StorageSyncManager.cpp
    Storage/StorageThread.cpp
    Storage/StorageTracker.cpp
    Storage/WebDatabaseProvider.cpp
    Storage/WebStorageNamespaceProvider.cpp

    WebCoreSupport/WebResourceLoadScheduler.cpp
    WebCoreSupport/WebViewGroup.cpp
)

set(WebKit_INCLUDE_DIRECTORIES
    Storage
    WebCoreSupport

    "${CMAKE_BINARY_DIR}"
    "${CMAKE_SOURCE_DIR}/Source"
    "${DERIVED_SOURCES_DIR}"
    "${DERIVED_SOURCES_DIR}/ForwardingHeaders"
    "${DERIVED_SOURCES_DIR}/ForwardingHeaders/JavaScriptCore"
    "${DERIVED_SOURCES_DIR}/ForwardingHeaders/WebCore"
    "${DERIVED_SOURCES_DIR}/ForwardingHeaders/WebKitLegacy"
)

set(WebKit_SYSTEM_INCLUDE_DIRECTORIES
    "${ICU_INCLUDE_DIRS}"
)

set(WebKit_LIBRARIES
    PRIVATE JavaScriptCore${DEBUG_SUFFIX}
    PRIVATE WTF${DEBUG_SUFFIX}
    PRIVATE WebCore${DEBUG_SUFFIX}
)

WEBKIT_INCLUDE_CONFIG_FILES_IF_EXISTS()

if (MSVC)
    if (NOT ${PORT} STREQUAL "Qt")
        ADD_PRECOMPILED_HEADER("WebKitPrefix.h" "win/WebKitPrefix.cpp" WebKit_SOURCES)
        list(APPEND WebKit_SOURCES win/WebKit.resources/WebKit.rc)
        add_definitions(/DWEBKIT_EXPORTS /DFRAMEWORK_NAME=WebKit)
    else ()
        ADD_PRECOMPILED_HEADER("WebKitPrefix.h" "qt/WebKitPrefix.cpp" WebKit_SOURCES)
    endif ()
endif ()

set(WebKit_PRIVATE_HEADERS_LOCATION Headers/${PROJECT_VERSION}/QtWebKit/Private)

WEBKIT_FRAMEWORK(WebKit)

set_target_properties(WebKit PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION_MAJOR})
install(TARGETS WebKit EXPORT WebKitTargets
        DESTINATION "${LIB_INSTALL_DIR}"
        INCLUDES DESTINATION "${KDE_INSTALL_INCLUDEDIR}/QtWebKitWidgets"
)

add_dependencies(WebKit WebCore)
if (WIN32)
    add_dependencies(WebKit WebKitGUID)
endif ()

if (NOT MSVC AND ${PORT} STREQUAL "Qt")
    ADD_WEBKIT_PREFIX_HEADER(WebKit "${WEBKIT_DIR}/qt/WebKitPrefix.h")
endif ()
