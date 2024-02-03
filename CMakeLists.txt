cmake_minimum_required(VERSION 3.14)
project(backtrace C)


set(BT_SOURCES
        "${PROJECT_SOURCE_DIR}/atomic.c"
        "${PROJECT_SOURCE_DIR}/backtrace.c"
        "${PROJECT_SOURCE_DIR}/dwarf.c"
        "${PROJECT_SOURCE_DIR}/fileline.c"
        "${PROJECT_SOURCE_DIR}/mmapio.c"
        "${PROJECT_SOURCE_DIR}/mmap.c"
        "${PROJECT_SOURCE_DIR}/posix.c"
        "${PROJECT_SOURCE_DIR}/print.c"
        "${PROJECT_SOURCE_DIR}/simple.c"
        "${PROJECT_SOURCE_DIR}/sort.c"
        "${PROJECT_SOURCE_DIR}/state.c"
)
set(BACKTRACE_SUPPORTED 1)
set(BACKTRACE_ELF_SIZE 64)
set(HAVE_GETIPINFO 1)
set(BACKTRACE_USES_MALLOC 1)
set(BACKTRACE_SUPPORTS_THREADS 1)
set(BACKTRACE_SUPPORTS_DATA 1)
set(HAVE_SYNC_FUNCTIONS 1)
if(APPLE)
    set(HAVE_MACH_O_DYLD_H 1)
    list(APPEND BT_SOURCES "${PROJECT_SOURCE_DIR}/macho.c")
else()
    set(HAVE_MACH_O_DYLD_H 0)
    list(APPEND BT_SOURCES "${PROJECT_SOURCE_DIR}/elf.c")
endif()
# Generate backtrace-supported.h based on the above.
configure_file(
        ${CMAKE_SOURCE_DIR}/cmake/backtrace-supported.h.in
        ${PROJECT_SOURCE_DIR}/backtrace-supported.h)
configure_file(
        ${CMAKE_SOURCE_DIR}/cmake/backtrace-config.h.in
        ${PROJECT_SOURCE_DIR}/config.h)
add_library(backtrace STATIC ${BT_SOURCES})
target_include_directories(backtrace BEFORE PRIVATE "${PROJECT_SOURCE_DIR}")
set_target_properties(backtrace PROPERTIES
        COMPILE_FLAGS "-funwind-tables -D_GNU_SOURCE"
        POSITION_INDEPENDENT_CODE ON)

install(TARGETS backtrace
        EXPORT backtrace
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)
install(FILES "${PROJECT_SOURCE_DIR}/backtrace.h"
        DESTINATION include)