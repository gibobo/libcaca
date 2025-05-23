project(caca)

find_package(Curses)
find_package(GLUT)
find_package(ZLIB)
find_package(X11)

# Add Include Path:
include_directories(caca caca/codec)
set(LIBRARY caca)

file(GLOB SRC_FILES
    caca/driver/gl.c
    caca/driver/ncurses.c
    caca/driver/null.c
    caca/driver/raw.c
    caca/driver/win32.c
    caca/driver/x11.c
    caca/codec/export.c
    caca/codec/import.c
    caca/codec/text.c
    caca/attr.c
    caca/box.c
    caca/caca.c
    caca/canvas.c
    caca/charset.c
    caca/dirty.c
    caca/dither.c
    caca/event.c
    caca/figfont.c
    caca/file.c
    caca/font.c
    caca/frame.c
    caca/getopt.c
    caca/graphics.c
    caca/line.c
    caca/prof.c
    caca/string.c
    caca/time.c
)

# Build:
add_library(${PROJECT_NAME} STATIC ${SRC_FILES})

if (WIN32)
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/build/win32)
else()
    include_directories(${CMAKE_CURRENT_SOURCE_DIR})
    list(APPEND LIBRARY m ${CURSES_LIBRARIES} ${ZLIB_LIBRARIES} ${GLUT_LIBRARIES} ${X11_LIBRARIES})
    # add_executable(cacaserver src/cacaserver.c)
    # add_executable(cacademo src/cacademo.c)
    # target_link_libraries(cacaserver ${LIBRARY})
    # target_link_libraries(cacademo ${LIBRARY})
endif()
# set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "caca/caca.h;caca/caca_types.h")
set(CACA_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/caca" PARENT_SCOPE)
set(CACA_LIBRARYS ${LIBRARY} PARENT_SCOPE)
