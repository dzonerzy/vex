cmake_minimum_required(VERSION 3.10)

# Set the project name
project(vex)

# Set the C standard
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Define the output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# Define CFLAGS based on Makefile-gcc
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Ipub -Ipriv -std=gnu99 -fstrict-aliasing -fPIC -DPYVEX ${EXTRA_CFLAGS}")

# Include directories
include_directories(
    ${CMAKE_SOURCE_DIR}/pub
    ${CMAKE_SOURCE_DIR}/priv
    ${CMAKE_BINARY_DIR}/pub # For generated headers
)

# Define source files based on common.mk
set(PUB_HEADERS
    ${CMAKE_SOURCE_DIR}/pub/libvex_basictypes.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_ir.h
    ${CMAKE_SOURCE_DIR}/pub/libvex.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_trc_values.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_emnote.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_guest_x86.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_guest_amd64.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_guest_arm.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_guest_ppc32.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_guest_ppc64.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_guest_riscv64.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_guest_s390x.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_s390x_common.h
    ${CMAKE_SOURCE_DIR}/pub/libvex_guest_mips32.h
)

set(GEN_HEADERS
    ${CMAKE_SOURCE_DIR}/pub/libvex_guest_offsets.h
)

set(PRIV_HEADERS
    ${CMAKE_SOURCE_DIR}/priv/host_x86_defs.h
    ${CMAKE_SOURCE_DIR}/priv/host_amd64_defs.h
    ${CMAKE_SOURCE_DIR}/priv/host_arm_defs.h
    ${CMAKE_SOURCE_DIR}/priv/host_ppc_defs.h
    ${CMAKE_SOURCE_DIR}/priv/host_riscv64_defs.h
    ${CMAKE_SOURCE_DIR}/priv/host_s390_defs.h
    ${CMAKE_SOURCE_DIR}/priv/host_mips_defs.h
    ${CMAKE_SOURCE_DIR}/priv/host_generic_maddf.h
    ${CMAKE_SOURCE_DIR}/priv/host_generic_regs.h
    ${CMAKE_SOURCE_DIR}/priv/host_generic_simd64.h
    ${CMAKE_SOURCE_DIR}/priv/host_generic_simd128.h
    ${CMAKE_SOURCE_DIR}/priv/host_generic_simd256.h
    ${CMAKE_SOURCE_DIR}/priv/main_globals.h
    ${CMAKE_SOURCE_DIR}/priv/main_util.h
    ${CMAKE_SOURCE_DIR}/priv/guest_generic_x87.h
    ${CMAKE_SOURCE_DIR}/priv/guest_generic_bb_to_IR.h
    ${CMAKE_SOURCE_DIR}/priv/guest_x86_defs.h
    ${CMAKE_SOURCE_DIR}/priv/guest_amd64_defs.h
    ${CMAKE_SOURCE_DIR}/priv/guest_arm_defs.h
    ${CMAKE_SOURCE_DIR}/priv/guest_ppc_defs.h
    ${CMAKE_SOURCE_DIR}/priv/guest_riscv64_defs.h
    ${CMAKE_SOURCE_DIR}/priv/guest_mips_defs.h
    ${CMAKE_SOURCE_DIR}/priv/s390_disasm.h
    ${CMAKE_SOURCE_DIR}/priv/s390_defs.h
    ${CMAKE_SOURCE_DIR}/priv/ir_match.h
    ${CMAKE_SOURCE_DIR}/priv/ir_opt.h
)

set(NORMAL_OBJS
    priv/ir_defs.c
    priv/ir_match.c
    priv/ir_opt.c
    priv/ir_inject.c
    priv/main_globals.c
    priv/main_util.c
    priv/s390_disasm.c
    priv/host_x86_defs.c
    priv/host_amd64_defs.c
    priv/host_arm_defs.c
    priv/host_arm64_defs.c
    priv/host_ppc_defs.c
    priv/host_riscv64_defs.c
    priv/host_s390_defs.c
    priv/host_mips_defs.c
    priv/host_x86_isel.c
    priv/host_amd64_isel.c
    priv/host_arm_isel.c
    priv/host_arm64_isel.c
    priv/host_ppc_isel.c
    priv/host_riscv64_isel.c
    priv/host_s390_isel.c
    priv/host_mips_isel.c
    priv/host_generic_maddf.c
    priv/host_generic_regs.c
    priv/host_generic_simd64.c
    priv/host_generic_simd128.c
    priv/host_generic_simd256.c
    priv/host_generic_reg_alloc2.c
    priv/host_generic_reg_alloc3.c
    priv/guest_generic_x87.c
    priv/guest_generic_bb_to_IR.c
    priv/guest_x86_helpers.c
    priv/guest_amd64_helpers.c
    priv/guest_arm_helpers.c
    priv/guest_arm64_helpers.c
    priv/guest_ppc_helpers.c
    priv/guest_riscv64_helpers.c
    priv/guest_s390_helpers.c
    priv/guest_mips_helpers.c
    priv/guest_x86_toIR.c
    priv/guest_amd64_toIR.c
    priv/guest_arm_toIR.c
    priv/guest_arm64_toIR.c
    priv/guest_ppc_toIR.c
    priv/guest_riscv64_toIR.c
    priv/guest_s390_toIR.c
    priv/guest_mips_toIR.c
)

set(SINGLEARCH_OBJS priv/main_main.c)
set(MULTIARCH_OBJS priv/multiarch_main_main.c)

# Ensure the directory for the generated executable exists
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/auxprogs)

# Custom command to generate headers
add_custom_command(
    OUTPUT ${CMAKE_SOURCE_DIR}/pub/libvex_guest_offsets.h
    COMMAND ${CMAKE_C_COMPILER} ${CMAKE_SOURCE_DIR}/auxprogs/genoffsets.c -o ${CMAKE_BINARY_DIR}/auxprogs/genoffsets${CMAKE_EXECUTABLE_SUFFIX}
    COMMAND ${CMAKE_BINARY_DIR}/auxprogs/genoffsets${CMAKE_EXECUTABLE_SUFFIX} > ${CMAKE_SOURCE_DIR}/pub/libvex_guest_offsets.h
    DEPENDS ${PUB_HEADERS} ${CMAKE_SOURCE_DIR}/auxprogs/genoffsets.c
    COMMENT "Generating dynamic header files..."
)

# Create a custom target to generate headers
add_custom_target(generate_headers ALL
    DEPENDS ${GEN_HEADERS}
)

# Handle multiarch compilation
option(MULTIARCH "Enable multiarch compilation" OFF)
if (MULTIARCH)
    add_definitions(-DMULTIARCH)
    set(SOURCES ${NORMAL_OBJS} ${MULTIARCH_OBJS})
else()
    set(SOURCES ${NORMAL_OBJS} ${SINGLEARCH_OBJS})
endif()

# Create the static library
add_library(${PROJECT_NAME}_static STATIC ${SOURCES})

# Create the shared library
add_library(${PROJECT_NAME}_shared SHARED ${SOURCES})

# Add dependencies to ensure headers are generated before compilation
add_dependencies(${PROJECT_NAME}_static generate_headers)
add_dependencies(${PROJECT_NAME}_shared generate_headers)

# Platform-specific settings
if (WIN32)
    set(CMAKE_C_COMPILER "gcc")
    set(CMAKE_CXX_COMPILER "g++")
    target_link_libraries(${PROJECT_NAME}_static)
    target_link_libraries(${PROJECT_NAME}_shared)
else()
    target_compile_options(${PROJECT_NAME}_static PRIVATE -Wall -Wextra)
    target_compile_options(${PROJECT_NAME}_shared PRIVATE -Wall -Wextra)
    target_link_libraries(${PROJECT_NAME}_static pthread)
    target_link_libraries(${PROJECT_NAME}_shared pthread)
endif()

# Handle debug mode
option(DEBUG "Enable debug mode" OFF)
if (DEBUG)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O0")
else()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2")
endif()

# Handle extra C flags
if (EXTRA_CFLAGS)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${EXTRA_CFLAGS}")
endif()
