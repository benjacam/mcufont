cmake_minimum_required(VERSION 3.22)

include(ExternalProject)
ExternalProject_Add(mfencoder
   SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/../encoder
   PREFIX ${CMAKE_CURRENT_BINARY_DIR}/mfencoder
   INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}
   CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}
)

set(MFENC ${CMAKE_CURRENT_BINARY_DIR}/bin/mfencoder)

set(FONTS DejaVuSans)
set(PIXEL_SIZES 12)# 16 32)
set(RANGES "0-255 0x2010-0x2015")
set(ITERATIONS 1)# todo 50)
set(font_h_text "")
set(font_h_filename ${CMAKE_CURRENT_BINARY_DIR}/fonts.h)

set(font_h_list)
foreach(font ${FONTS})
   foreach (size ${PIXEL_SIZES})
      set(output_name ${font}${size}bw)
      set(h_name ${CMAKE_CURRENT_BINARY_DIR}/${output_name}.h)
      set(dat_name ${CMAKE_CURRENT_LIST_DIR}/${output_name}.dat)
      list(APPEND font_h_list ${h_name})
      string(APPEND font_h_text "#include <${h_name}>")
      add_custom_command(
         OUTPUT ${h_name}
         COMMAND ${MFENC} import_ttf ${CMAKE_CURRENT_LIST_DIR}/${font}.ttf ${size} bw
         COMMAND ${MFENC} filter ${dat_name} ${RANGES}
         COMMAND ${MFENC} rlefont_optimize ${dat_name} ${ITERATIONS}
         COMMAND ${MFENC} bwfont_export ${dat_name}
         COMMAND cp ${CMAKE_CURRENT_LIST_DIR}/${output_name}.c ${h_name}
         DEPENDS mfencoder
         DEPENDS ${CMAKE_CURRENT_LIST_DIR}/${font}.ttf
         BYPRODUCTS ${dat_name} ${CMAKE_CURRENT_LIST_DIR}/${output_name}.c
         VERBATIM)
   endforeach()
endforeach()

add_custom_command(
   OUTPUT ${font_h_filename}
   COMMAND echo "${font_h_text}" > ${font_h_filename}
   VERBATIM)

list(APPEND font_h_list ${font_h_filename})

add_custom_target(generate_custom_header DEPENDS ${font_h_list})
add_library(mcufont_fonts INTERFACE)
add_dependencies(mcufont_fonts generate_custom_header)
target_include_directories(mcufont_fonts INTERFACE ${CMAKE_CURRENT_BINARY_DIR})
