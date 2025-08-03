project(caca)

# Include required modules
include(CheckIncludeFile)
include(CheckFunctionExists)
include(CheckSymbolExists)
include(CheckCSourceCompiles)
include(CheckLibraryExists)

find_package(Curses)
find_package(OpenGL)
find_package(GLUT)
find_package(ZLIB)
find_package(X11)

# Package information
set(PACKAGE_NAME "libcaca")
set(PACKAGE_VERSION "0.99.beta20")
set(PACKAGE_STRING "${PACKAGE_NAME} ${PACKAGE_VERSION}")
set(PACKAGE_BUGREPORT "")
set(PACKAGE_URL "http://caca.zoy.org/wiki/libcaca")
set(PACKAGE_TARNAME "libcaca")

# Check for standard headers
check_include_file(stdio.h HAVE_STDIO_H)
check_include_file(stdlib.h HAVE_STDLIB_H)
check_include_file(string.h HAVE_STRING_H)
check_include_file(strings.h HAVE_STRINGS_H)
check_include_file(memory.h HAVE_MEMORY_H)
check_include_file(stdarg.h HAVE_STDARG_H)
check_include_file(errno.h HAVE_ERRNO_H)
check_include_file(unistd.h HAVE_UNISTD_H)
check_include_file(signal.h HAVE_SIGNAL_H)
check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(sys/stat.h HAVE_SYS_STAT_H)
check_include_file(sys/time.h HAVE_SYS_TIME_H)
check_include_file(sys/ioctl.h HAVE_SYS_IOCTL_H)
check_include_file(termios.h HAVE_TERMIOS_H)
check_include_file(inttypes.h HAVE_INTTYPES_H)
check_include_file(stdint.h HAVE_STDINT_H)
check_include_file(locale.h HAVE_LOCALE_H)
check_include_file(getopt.h HAVE_GETOPT_H)
check_include_file(dlfcn.h HAVE_DLFCN_H)
check_include_file(endian.h HAVE_ENDIAN_H)
check_include_file(arpa/inet.h HAVE_ARPA_INET_H)
check_include_file(netinet/in.h HAVE_NETINET_IN_H)
check_include_file(windows.h HAVE_WINDOWS_H)
check_include_file(winsock2.h HAVE_WINSOCK2_H)
check_include_file(conio.h HAVE_CONIO_H)
check_include_file(zlib.h HAVE_ZLIB_H)

# Windows-specific headers
if(WIN32)
    check_include_file(sys/socket.h HAVE_SYS_SOCKET_H)
endif()

# Check for curses headers
check_include_file(curses.h HAVE_CURSES_H)
check_include_file(ncurses.h HAVE_NCURSES_H)
check_include_file(ncurses/ncurses.h HAVE_NCURSES_NCURSES_H)
check_include_file(ncursesw/ncurses.h HAVE_NCURSESW_NCURSES_H)

# Check for OpenGL headers
check_include_file(GL/gl.h HAVE_GL_GL_H)
check_include_file(GL/glut.h HAVE_GL_GLUT_H)
check_include_file(OpenGL/gl.h HAVE_OPENGL_GL_H)

# Check for X11 headers
check_include_file(X11/XKBlib.h HAVE_X11_XKBLIB_H)

# Check for other graphics libraries
check_include_file(imlib2.h HAVE_IMLIB2_H)
check_include_file(slang.h HAVE_SLANG_H)
check_include_file(slang/slang.h HAVE_SLANG_SLANG_H)

# Check for functions
check_function_exists(getenv HAVE_GETENV)
check_function_exists(putenv HAVE_PUTENV)
check_function_exists(atexit HAVE_ATEXIT)
check_function_exists(signal HAVE_SIGNAL)
check_function_exists(getopt_long HAVE_GETOPT_LONG)
check_function_exists(gettimeofday HAVE_GETTIMEOFDAY)
check_function_exists(usleep HAVE_USLEEP)
check_function_exists(sleep HAVE_SLEEP)
check_function_exists(snprintf HAVE_SNPRINTF)
check_function_exists(vsnprintf HAVE_VSNPRINTF)
check_function_exists(strcasecmp HAVE_STRCASECMP)
check_function_exists(ioctl HAVE_IOCTL)
check_function_exists(htons HAVE_HTONS)

# Windows-specific function checks
if(WIN32)
    check_function_exists(sprintf_s HAVE_SPRINTF_S)
    check_function_exists(vsnprintf_s HAVE_VSNPRINTF_S)
    # Check for Windows Sleep function (different from POSIX sleep)
    check_c_source_compiles("
        #include <windows.h>
        int main(void) { Sleep(1000); return 0; }
    " HAVE_SLEEP_WIN32)
    if(HAVE_SLEEP_WIN32)
        set(HAVE_SLEEP 1)
    endif()
endif()

# Check for curses functions if curses is available
if(CURSES_FOUND)
    set(CMAKE_REQUIRED_LIBRARIES ${CURSES_LIBRARIES})
    check_function_exists(resizeterm HAVE_RESIZETERM)
    check_function_exists(resize_term HAVE_RESIZE_TERM)
    set(CMAKE_REQUIRED_LIBRARIES)
endif()

# Check for OpenGL functions if OpenGL is available
if(OPENGL_FOUND)
    set(CMAKE_REQUIRED_LIBRARIES ${OPENGL_LIBRARIES})
    check_function_exists(glutCheckLoop HAVE_GLUTCHECKLOOP)
    check_function_exists(glutCloseFunc HAVE_GLUTCLOSEFUNC)
    set(CMAKE_REQUIRED_LIBRARIES)
endif()

# Check for S-Lang functions
check_library_exists(slang SLsmg_utf8_enable "" HAVE_SLSMG_UTF8_ENABLE)

# Platform-specific checks and backend selection
if(WIN32)
    set(USE_WIN32 1)
    # Check for Windows-specific features
    if(HAVE_WINDOWS_H)
        check_symbol_exists(AllocConsole "windows.h" ALLOCCONSOLE_IN_WINDOWS_H)
    endif()
    check_symbol_exists(ScreenUpdate "pc.h" SCREENUPDATE_IN_PC_H)
    
    # Don't automatically enable CONIO on Windows - it's not compatible with modern compilers
    # if(HAVE_CONIO_H)
    #     set(USE_CONIO 1)
    # endif()
elseif(APPLE)
    # macOS/iOS specific settings
    set(USE_COCOA 1)
    find_library(COCOA_FRAMEWORK Cocoa)
    if(COCOA_FRAMEWORK)
        list(APPEND LIBRARY ${COCOA_FRAMEWORK})
    endif()
else()
    # Unix-like systems (Linux, BSD, etc.)
    set(USE_NULL 1)  # Default to null driver for minimal build
    
    # Enable ncurses if available
    if(CURSES_FOUND)
        set(USE_NCURSES 1)
        list(APPEND LIBRARY ${CURSES_LIBRARIES})
    endif()
    
    # Enable X11 if available  
    if(X11_FOUND)
        set(USE_X11 1)
        list(APPEND LIBRARY ${X11_LIBRARIES})
    endif()
    
    # Enable OpenGL if available
    if(OPENGL_FOUND)
        set(USE_GL 1)
        list(APPEND LIBRARY ${OPENGL_LIBRARIES})
    endif()
    
    # Add standard Unix libraries
    list(APPEND LIBRARY m)  # Math library
    if(ZLIB_FOUND)
        list(APPEND LIBRARY ${ZLIB_LIBRARIES})
    endif()
endif()

# Check for inline support
check_c_source_compiles("
    static inline int test_inline(void) { return 0; }
    int main(void) { return test_inline(); }
" HAVE_INLINE)

# Check for STDC headers
if(HAVE_STDIO_H AND HAVE_STDLIB_H AND HAVE_STRING_H)
    set(STDC_HEADERS 1)
endif()

# Compiler-specific checks
if(MSVC)
    # Microsoft Visual C++ specific checks
    check_c_source_compiles("
        #include <stdio.h>
        int main(void) { 
            char buf[100];
            sprintf_s(buf, sizeof(buf), \"test\");
            return 0; 
        }
    " HAVE_SPRINTF_S_WORKS)
    
    # Check MSVC version for snprintf availability
    # MSVC 2015 (version 1900) and later have snprintf
    if(MSVC_VERSION LESS 1900)
        set(NEED_SNPRINTF_COMPAT 1)
    endif()
endif()

# Generate config.h
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/config.h.in
    ${CMAKE_CURRENT_BINARY_DIR}/config.h
    @ONLY
)

# Add Include Path:
include_directories(${CMAKE_CURRENT_BINARY_DIR} caca caca/codec)

# Initialize library list
set(LIBRARY)  # Start with empty list, will be populated by platform checks

# Platform-specific build configuration
if(WIN32)
    # Windows specific libraries
    set(USE_WIN32 1)
    # Don't use CONIO on modern Windows - it's for DOS only
    unset(USE_CONIO)
    # Windows doesn't need many external libraries
    set(LIBRARY user32 gdi32)
    
elseif(APPLE)
    # macOS specific libraries
    set(USE_COCOA 1)
    find_library(COCOA_FRAMEWORK Cocoa)
    if(COCOA_FRAMEWORK)
        list(APPEND LIBRARY ${COCOA_FRAMEWORK})
    endif()
    
    # Enable ncurses if available
    if(CURSES_FOUND)
        set(USE_NCURSES 1)
        list(APPEND LIBRARY ${CURSES_LIBRARIES})
    endif()
    
    # Enable X11 if available  
    if(X11_FOUND)
        set(USE_X11 1)
        list(APPEND LIBRARY ${X11_LIBRARIES})
    endif()
    
    # Enable OpenGL if available
    if(OPENGL_FOUND)
        set(USE_GL 1)
        list(APPEND LIBRARY ${OPENGL_LIBRARIES})
    endif()
    
    # Add standard Unix libraries
    list(APPEND LIBRARY m)  # Math library
    if(ZLIB_FOUND)
        list(APPEND LIBRARY ${ZLIB_LIBRARIES})
    endif()
    
else()
    # Linux and other Unix systems
    # Enable ncurses if available
    if(CURSES_FOUND)
        set(USE_NCURSES 1)
        list(APPEND LIBRARY ${CURSES_LIBRARIES})
    endif()
    
    # Enable X11 if available  
    if(X11_FOUND)
        set(USE_X11 1)
        list(APPEND LIBRARY ${X11_LIBRARIES})
    endif()
    
    # Enable OpenGL if available
    if(OPENGL_FOUND)
        set(USE_GL 1)
        list(APPEND LIBRARY ${OPENGL_LIBRARIES})
    endif()
    
    # Add standard Unix libraries
    list(APPEND LIBRARY m)  # Math library
    if(ZLIB_FOUND)
        list(APPEND LIBRARY ${ZLIB_LIBRARIES})
    endif()
endif()

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

# Build library
add_library(${PROJECT_NAME} STATIC ${SRC_FILES})

# Apply compiler-specific settings after target creation
if(MSVC)
    target_compile_definitions(${PROJECT_NAME} PRIVATE
        _CRT_SECURE_NO_WARNINGS
        _CRT_NONSTDC_NO_DEPRECATE
    )
endif()

# Cross-platform compiler flags
if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(${PROJECT_NAME} PRIVATE
        -Wall -Wextra -Wno-unused-parameter
        -fno-strict-aliasing
    )
endif()

# Platform-specific build configuration
if(WIN32)
    # Use pre-built Windows config
    target_include_directories(${PROJECT_NAME} PRIVATE 
        ${CMAKE_CURRENT_SOURCE_DIR}/build/win32
    )
else()
    # Use generated config.h for Unix-like platforms
    target_include_directories(${PROJECT_NAME} PRIVATE 
        ${CMAKE_CURRENT_BINARY_DIR}
    )
    target_link_libraries(${PROJECT_NAME} PRIVATE ${LIBRARY})
endif()

# Set public headers and export variables
set(CACA_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/caca" PARENT_SCOPE)
set(CACA_LIBRARYS ${PROJECT_NAME} ${LIBRARY} PARENT_SCOPE)
