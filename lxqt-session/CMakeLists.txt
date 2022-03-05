set(PROJECT lwqt-session)

project(${PROJECT})

if (WITH_LIBUDEV)
    find_package(UDev REQUIRED)
    include_directories(${LIBUDEV_INCLUDE_DIR})
    add_definitions(-DWITH_LIBUDEV_MONITOR)
endif()

if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
    include_directories(
        ${PROCPS_INCLUDE_DIRS}
    )
endif()

set(lxqt-session_HDRS "")

set(lxqt-session_SRCS
    src/main.cpp
    src/lxqtmodman.cpp
    src/sessionapplication.cpp
    src/sessiondbusadaptor.h
    src/lockscreenmanager.cpp
    src/lockscreenmanager.h
    src/numlock.h
    src/log.cpp
    src/procreaper.cpp
)
if (WITH_LIBUDEV)
    list(APPEND lxqt-session_SRCS src/UdevNotifier.cpp)
endif()

# Translations
lxqt_translate_ts(lxqt-session_QM_FILES
    UPDATE_TRANSLATIONS
        ${UPDATE_TRANSLATIONS}
    SOURCES
        ${lxqt-session_HDRS}
        ${lxqt-session_SRCS}
        ${lxqt-session_UI}
    INSTALL_DIR
        "${LXQT_TRANSLATIONS_DIR}/${PROJECT_NAME}"
)
lxqt_app_translation_loader(lxqt-session_QM_LOADER ${PROJECT_NAME})

add_executable(${PROJECT}
    ${lxqt-session_SRCS}
    ${RSCS}
    ${lxqt-session_QM_FILES}
    ${lxqt-session_QM_LOADER}
)

target_link_libraries(${PROJECT}
    Qt5::Widgets
    Qt5::DBus
    lxqt
    KF5::WindowSystem
    procps
)
if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
    target_link_libraries(${PROJECT}
        ${PROCPS_LIBRARIES}
    )
elseif (CMAKE_SYSTEM_NAME STREQUAL "FreeBSD")
    target_link_libraries(${PROJECT}
        -lutil
    )
elseif (CMAKE_SYSTEM_NAME STREQUAL "NetBSD")
    target_link_libraries(lxqt-session
        -lkvm
    )
endif()

if (WITH_LIBUDEV)
    target_link_libraries(${PROJECT} ${UDEV_LIBS})
endif()

INSTALL(TARGETS
    ${PROJECT}
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    COMPONENT Runtime
)
install(FILES
    man/${PROJECT}.1.gz
    DESTINATION "${CMAKE_INSTALL_MANDIR}/man1"
    COMPONENT Runtime
)
