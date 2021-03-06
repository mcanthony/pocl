#=============================================================================
#   CMake build system files
#
#   Copyright (c) 2015 pocl developers
#
#   Permission is hereby granted, free of charge, to any person obtaining a copy
#   of this software and associated documentation files (the "Software"), to deal
#   in the Software without restriction, including without limitation the rights
#   to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
#   copies of the Software, and to permit persons to whom the Software is
#   furnished to do so, subject to the following conditions:
#
#   The above copyright notice and this permission notice shall be included in
#   all copies or substantial portions of the Software.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
#   AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
#   OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
#   THE SOFTWARE.
#
#=============================================================================

include("bitcode_rules")

set(KERNEL_SOURCES ${SOURCES_WITHOUT_VML})

foreach(FILE printf.c barrier.ll get_image_depth.cl get_image_dim.cl
  get_image_height.cl get_image_width.cl read_image.cl write_image.cl)
  list(REMOVE_ITEM KERNEL_SOURCES "${FILE}")
endforeach()


foreach(FILE atomics.cl atomic_impl.ll barrier.c
        get_global_id.c get_global_size.c get_group_id.c
        get_local_id.c get_local_size.c get_num_groups.c get_work_dim.c
        native_cos.cl native_exp.cl native_exp10.cl native_exp2.cl
        native_log.cl native_log10.cl native_log2.cl native_recip.cl
        native_rsqrt.cl  native_sin.cl  native_sqrt.cl
        fabs.cl floor.cl rint.cl trunc.cl remainder.cl
        fma.cl mad.cl mad_hi.cl mul_hi.cl mul24.cl mad24.cl
        sqrt.cl sqrt_default.ll cbrt.cl hypot.cl copysign.cl
        exp.cl exp2.cl exp10.cl expm1.cl
        log.cl log2.cl log10.cl log1p.cl
        sin.cl tan.cl cos.cl sinh.cl tanh.cl cosh.cl
        asin.cl acos.cl atan.cl asinh.cl acosh.cl atanh.cl
        ilogb.cl ldexp.cl fract.cl frexp.cl atan2.cl pow.cl)

  list(REMOVE_ITEM KERNEL_SOURCES "${FILE}")
  list(APPEND KERNEL_SOURCES "hsail64/${FILE}")
endforeach()

set(CLANG_FLAGS "-Xclang" "-ffake-address-space-map" "-emit-llvm" "-target" "hsail64" "-D_CL_DISABLE_HALF")
set(LLC_FLAGS "")

make_kernel_bc(KERNEL_BC "hsail64" ${KERNEL_SOURCES})

# just debug
message(STATUS "HSAIL64 Kernel BC: ${KERNEL_BC}")

# a target is needed...
add_custom_target("kernel_hsail64" ALL
    DEPENDS ${KERNEL_BC} "${CMAKE_BINARY_DIR}/kernellib_hash.h")

install(FILES "${KERNEL_BC}"
        DESTINATION "${POCL_INSTALL_PRIVATE_DATADIR}")
