# md

list (APPEND MD_INCLUDE_PATH "include")

file(GLOB IN_SRC src/*.cc src/*.c)

function(_add_ipi_variant variant_name prec_def)
set (ipiname "dp_ipi${variant_name}")
set (libipiname "${LIB_DEEPMD_IPI}${variant_name}")
add_library(${libipiname} SHARED ${IN_SRC})
target_include_directories(${libipiname} PUBLIC ${MD_INCLUDE_PATH})

set(DRIVER_SOURCE_FILES driver.cc)
add_executable(${ipiname} ${DRIVER_SOURCE_FILES})
# link: libdeepmd_cc
target_link_libraries(${ipiname} PRIVATE ${libipiname} ${LIB_DEEPMD_CC})
target_include_directories(${ipiname} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/../3rdparty/)

if (APPLE)
  set_target_properties(
    ${ipiname}
    PROPERTIES
    INSTALL_RPATH "@loader_path/../lib:${TensorFlow_LIBRARY_PATH}"
    COMPILE_DEFINITIONS ${prec_def}
  )
else()
set_target_properties(
  ${ipiname}
  PROPERTIES
  LINK_FLAGS "-Wl,-rpath,'$ORIGIN'/../lib -Wl,-z,defs"
  INSTALL_RPATH "$ORIGIN/../lib:${TensorFlow_LIBRARY_PATH}"
  COMPILE_DEFINITIONS ${prec_def}
)
endif()

if(BUILD_PY_IF)
install(
  TARGETS	${libipiname}
  DESTINATION	lib/
)
install(
  TARGETS	${ipiname}
  DESTINATION	bin/
)
else(BUILD_PY_IF)
  install(
    TARGETS	${libipiname}
    DESTINATION	deepmd/op/
  )
  install(
    TARGETS	${ipiname}
    DESTINATION	deepmd/op/
  )
endif(BUILD_PY_IF)
endfunction()
_add_ipi_variant("${HIGH_PREC_VARIANT}" "${HIGH_PREC_DEF}")
_add_ipi_variant("${LOW_PREC_VARIANT}" "${LOW_PREC_DEF}")