project(libcaca)

# Add Include Path:
include_directories(caca caca/codec)

file(GLOB SRC_FILES
    caca/driver/conio.c
    caca/driver/gl.c
    caca/driver/ncurses.c
    caca/driver/null.c
    caca/driver/raw.c
    caca/driver/slang.c
    caca/driver/vga.c
    caca/driver/win32.c
    caca/driver/x11.c
    caca/codec/export.c
    caca/codec/import.c
    caca/codec/text.c
    caca/attr.c
    caca/box.c
    caca/caca.c
    caca/caca_conio.c
    caca/canvas.c
    caca/charset.c
    caca/conic.c
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
    caca/transform.c
    caca/triangle.c
)

# Build:
add_library(${PROJECT_NAME} SHARED ${SRC_FILES})

add_compile_definitions(
    _LIB
    __LIBCACA__
    DLL_EXPORT
)

include_directories(build/win32)
if (WIN32)
    add_compile_definitions(WIN32)
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "caca/caca.h;caca/caca_types.h;caca/caca_conio.h;caca/caca0.h")
