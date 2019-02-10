cmake_minimum_required(VERSION 3.5)
project(stb VERSION 1.0.0 LANGUAGES C)

##############################################
# Create target and set properties

# LIBRARY_NAME\;LIBRARY_IMPLEMENTATION_DEFINE\;LIBRARY_HEADER
set(STB_LIBRARIES
  "stb\;STB_DEFINE\;stb.h"
  "stb_c_lexer\;STB_IMAGE_IMPLEMENTATION\;stb_c_lexer.h"
  "stb_connected_components\;STB_CONNECTED_COMPONENTS_IMPLEMENTATION\;stb_connected_components.h"
  "stb_divide\;STB_DIVIDE_IMPLEMENTATION\;stb_divide.h"
  "stb_dxt\;STB_DXT_IMPLEMENTATION\;stb_dxt.h"
  "stb_easy_font\; \;stb_easy_font.h"
  "stb_herringbone_wang_tile\;STB_HERRINGBONE_WANG_TILE_IMPLEMENTATION\;stb_herringbone_wang_tile.h"
  "stb_image\;STB_IMAGE_IMPLEMENTATION\;stb_image.h"
  "stb_image_resize\;STB_IMAGE_RESIZE_IMPLEMENTATION\;stb_image_resize.h"
  "stb_image_write\;STB_IMAGE_WRITE_IMPLEMENTATION\;stb_image_write.h"
  "stb_leakcheck\;STB_LEAKCHECK_IMPLEMENTATION\;stb_leakcheck.h"
  "stb_perlin\;STB_PERLIN_IMPLEMENTATION\;stb_perlin.h"
  "stb_rect_pack\;STB_RECT_PACK_IMPLEMENTATION\;stb_rect_pack.h"
  "stb_sprintf\;STB_SPRINTF_IMPLEMENTATION\;stb_sprintf.h"
  "stb_textedit\;STB_TEXTEDIT_IMPLEMENTATION\;stb_textedit.h"
  "stb_tilemap_editor\;STB_TILEMAP_EDITOR_IMPLEMENTATION\;stb_tilemap_editor.h"
  "stb_truetype\;STB_TRUETYPE_IMPLEMENTATION\;stb_truetype.h"
  "stb_vorbis\; \;stb_vorbis.c"
  "stb_voxel_render\;STB_VOXEL_RENDER_IMPLEMENTATION\;stb_voxel_render.h"
  "stretchy_buffer\; \;stretchy_buffer.h"
)

function(target_variable_option TARGET NAME DEFAULT)
  option("${NAME}" "${NAME}" ${DEFAULT})

  if(${${NAME}})
    target_compile_definitions(${TARGET} PRIVATE "${NAME}")
  endif()
endfunction()

function(target_variable_definition TARGET NAME DEFAULT)
  set("${NAME}" "${DEFAULT}" CACHE STRING "${NAME}")

  if(NOT ${${NAME}} STREQUAL "")
    target_compile_definitions(${TARGET} PRIVATE "${NAME}=${${NAME}}")
  endif()
endfunction()

foreach(library ${STB_LIBRARIES})
  list(GET library 0 library_name)
  list(GET library 1 library_definition)
  list(GET library 2 library_header)

  string(TOUPPER ${library_name} library_name_upper)
  option("BUILD_${library_name_upper}" "Build the ${library_name} library" ON)

  if(NOT ${BUILD_${library_name_upper}})
    continue()
  endif()

  set(library_temp_c "${CMAKE_BINARY_DIR}/${library_name}.c")
  file(GENERATE
    OUTPUT ${library_temp_c}
    CONTENT "#include <${library_header}>"
  )
  add_library(${library_name} STATIC src/${library_name}/${library_header} ${library_temp_c})

  #Add an alias so that library can be used inside the build tree, e.g. when testing
  add_library("STB::${library_name}" ALIAS ${library_name})

  #Set target properties
  target_include_directories(${library_name}
    PUBLIC
      $<INSTALL_INTERFACE:include/${library_name}>
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/${library_name}>
    PRIVATE
      ${CMAKE_CURRENT_SOURCE_DIR}/src/${library_name}
  )

  target_compile_options(${library_name}
    PRIVATE
      $<$<CXX_COMPILER_ID:GNU>:-Wall>
  )

  if(NOT ${library_definition} STREQUAL " ")
    target_compile_definitions(${library_name}
      PRIVATE
        ${library_definition}
    )
  endif()

  if(${library_name} STREQUAL "stb_connected_components")
    target_variable_definition(${library_name} STB_CONNECTED_COMPONENTS_GRID_COUNT_X_LOG2 10)
    target_variable_definition(${library_name} STB_CONNECTED_COMPONENTS_GRID_COUNT_Y_LOG2 10)
  elseif(${library_name} STREQUAL "stb_divide")
    target_variable_option(${library_name} STB_DIVIDE_C_INTEGER_DIVISION_TRUNCATES OFF)
    target_variable_option(${library_name} STB_DIVIDE_C_INTEGER_DIVISION_FLOORS OFF)
  elseif(${library_name} STREQUAL "stb_dxt")
    target_variable_option(${library_name} STB_DXT_STATIC OFF)
  elseif(${library_name} STREQUAL "stb_herringbone_wang_tile")
    target_variable_option(${library_name} STB_HBWANG_STATIC OFF)
    
    target_variable_definition(${library_name} STB_HBWANG_RAND "")
    target_variable_definition(${library_name} STB_HBWANG_ASSERT "")
    target_variable_option(${library_name} STB_HBWANG_NO_REPITITION_REDUCTION OFF)
    target_variable_definition(${library_name} STB_HBWANG_MAX_X "")
    target_variable_definition(${library_name} STB_HBWANG_MAX_Y "")
  elseif(${library_name} STREQUAL "stb_image")
    target_variable_option(${library_name} STB_IMAGE_STATIC OFF)
    
    target_variable_definition(${library_name} STBI_ASSERT "")
    target_variable_definition(${library_name} STBI_MALLOC "")
    target_variable_definition(${library_name} STBI_REALLOC "")
    target_variable_definition(${library_name} STBI_FREE "")

    target_variable_option(${library_name} STBI_NO_STDIO OFF)
    target_variable_option(${library_name} STBI_WINDOWS_UTF8 OFF)
    target_variable_option(${library_name} STBI_SUPPORT_ZLIB OFF)

    target_variable_option(${library_name} STBI_NO_JPEG OFF)
    target_variable_option(${library_name} STBI_NO_PNG OFF)
    target_variable_option(${library_name} STBI_NO_BMP OFF)
    target_variable_option(${library_name} STBI_NO_PSD OFF)
    target_variable_option(${library_name} STBI_NO_TGA OFF)
    target_variable_option(${library_name} STBI_NO_GIF OFF)
    target_variable_option(${library_name} STBI_NO_HDR OFF)
    target_variable_option(${library_name} STBI_NO_PIC OFF)
    target_variable_option(${library_name} STBI_NO_PNM OFF)

    target_variable_option(${library_name} STBI_ONLY_JPEG OFF)
    target_variable_option(${library_name} STBI_ONLY_PNG OFF)
    target_variable_option(${library_name} STBI_ONLY_BMP OFF)
    target_variable_option(${library_name} STBI_ONLY_PSD OFF)
    target_variable_option(${library_name} STBI_ONLY_TGA OFF)
    target_variable_option(${library_name} STBI_ONLY_GIF OFF)
    target_variable_option(${library_name} STBI_ONLY_HDR OFF)
    target_variable_option(${library_name} STBI_ONLY_PIC OFF)
    target_variable_option(${library_name} STBI_ONLY_PNM OFF)
  elseif(${library_name} STREQUAL "stb_image_resize")
    target_variable_option(${library_name} STB_IMAGE_RESIZE_STATIC OFF)

    target_variable_option(${library_name} STBIR_NON_IEEE_FLOAT OFF)
    target_variable_definition(${library_name} STBIR_MALLOC "")
    target_variable_definition(${library_name} STBIR_FREE "")
    target_variable_definition(${library_name} STBIR_ASSERT "")
    target_variable_option(${library_name} STBIR_SATURATE_INT OFF)
    target_variable_definition(${library_name} STBIR_DEFAULT_FILTER_UPSAMPLE "")
    target_variable_definition(${library_name} STBIR_DEFAULT_FILTER_DOWNSAMPLE "")
    target_variable_definition(${library_name} STBIR_PROGRESS_REPORT "")
    target_variable_definition(${library_name} STBIR_MAX_CHANNELS "")
    target_variable_option(${library_name} STBIR_NO_ALPHA_EPSILON OFF)
  elseif(${library_name} STREQUAL "stb_image_write")
    target_variable_option(${library_name} STB_IMAGE_WRITE_STATIC OFF)

    target_variable_option(${library_name} STBI_MSC_SECURE_CRT OFF)
    target_variable_definition(${library_name} STBIW_ASSERT "")
    target_variable_definition(${library_name} STBIW_MALLOC "")
    target_variable_definition(${library_name} STBIW_REALLOC "")
    target_variable_definition(${library_name} STBIW_FREE "")
    target_variable_definition(${library_name} STBIW_MEMMOVE "")
    target_variable_definition(${library_name} STBIW_ZLIB_COMPRESS "")
    target_variable_option(${library_name} STBIW_WINDOWS_UTF8 OFF)
  elseif(${library_name} STREQUAL "stb_rect_pack")
    target_variable_option(${library_name} STBRP_STATIC OFF)
    
    target_variable_option(${library_name} STBRP_LARGE_RECTS OFF)
    target_variable_definition(${library_name} STBRP_SORT "")
    target_variable_definition(${library_name} STBRP_ASSERT "")
  elseif(${library_name} STREQUAL "stb_sprintf")
    target_variable_option(${library_name} STB_SPRINTF_STATIC OFF)
    
    target_variable_option(${library_name} STB_SPRINTF_NOFLOAT OFF)
    target_variable_definition(${library_name} STB_SPRINTF_MIN "")
  elseif(${library_name} STREQUAL "stb_textedit")
    target_variable_definition(${library_name} STB_TEXTEDIT_CHARTYPE "")
    target_variable_definition(${library_name} STB_TEXTEDIT_POSITIONTYPE "")
    target_variable_definition(${library_name} STB_TEXTEDIT_UNDOSTATECOUNT "")
    target_variable_definition(${library_name} STB_TEXTEDIT_UNDOCHARCOUNT "")

    target_variable_definition(${library_name} STB_TEXTEDIT_STRING "")
    target_variable_definition(${library_name} STB_TEXTEDIT_STRINGLEN "")
    target_variable_definition(${library_name} STB_TEXTEDIT_LAYOUTROW "")
    target_variable_definition(${library_name} STB_TEXTEDIT_GETWIDTH "")
    target_variable_definition(${library_name} STB_TEXTEDIT_KEYTOTEXT "")
    target_variable_definition(${library_name} STB_TEXTEDIT_GETCHAR "")
    target_variable_definition(${library_name} STB_TEXTEDIT_NEWLINE "")
    target_variable_definition(${library_name} STB_TEXTEDIT_DELETECHARS "")
    target_variable_definition(${library_name} STB_TEXTEDIT_INSERTCHARS "")

    target_variable_definition(${library_name} STB_TEXTEDIT_K_SHIFT "")

    target_variable_definition(${library_name} STB_TEXTEDIT_K_LEFT "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_RIGHT "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_UP "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_DOWN "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_LINESTART "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_LINEEND "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_TEXTSTART "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_TEXTEND "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_DELETE "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_BACKSPACE "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_UNDO "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_REDO "")

    #Optional
    target_variable_option(${library_name} STB_TEXTEDIT_K_INSERT OFF)
    target_variable_definition(${library_name} STB_TEXTEDIT_IS_SPACE "")
    target_variable_definition(${library_name} STB_TEXTEDIT_MOVEWORDLEFT "")
    target_variable_definition(${library_name} STB_TEXTEDIT_MOVEWORDRIGHT "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_WORDLEFT "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_WORDRIGHT "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_LINESTART2 "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_LINEEND2 "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_TEXTSTART2 "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_TEXTEND2 "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_PGUP "")
    target_variable_definition(${library_name} STB_TEXTEDIT_K_PGDOWN "")

    target_variable_option(${library_name} STB_SPRINTF_NOFLOAT OFF)
  elseif(${library_name} STREQUAL "stb_tilemap_editor")
    target_variable_definition(${library_name} STBTE_DRAW_RECT "")
    target_variable_definition(${library_name} STBTE_DRAW_TILE "")

    target_variable_definition(${library_name} STBTE_PROP_TYPE "")
    target_variable_definition(${library_name} STBTE_PROP_NAME "")
    target_variable_definition(${library_name} STBTE_PROP_MIN "")
    target_variable_definition(${library_name} STBTE_PROP_MAX "")
    target_variable_definition(${library_name} STBTE_PROP_FLOAT_SCALE "")
    target_variable_definition(${library_name} STBTE_FLOAT_CONTROL_GRANULARITY "")
    target_variable_definition(${library_name} STBTE_ALLOW_LINK "")
    target_variable_definition(${library_name} STBTE_LINK_COLOR "")
    target_variable_definition(${library_name} STBTE_HITTEST_TILE "")

    target_variable_definition(${library_name} STBTE_MAX_TILEMAP_X "")
    target_variable_definition(${library_name} STBTE_MAX_TILEMAP_Y "")
    target_variable_definition(${library_name} STBTE_MAX_LAYERS "")
    target_variable_definition(${library_name} STBTE_MAX_CATEGORIES "")
    target_variable_definition(${library_name} STBTE_UNDO_BUFFER_BYTES "")
    target_variable_definition(${library_name} STBTE_MAX_COPY "")
    target_variable_definition(${library_name} STBTE_MAX_PROPERTIES "")
  elseif(${library_name} STREQUAL "stb_truetype")
    target_variable_option(${library_name} STBTT_STATIC OFF)

    target_variable_definition(${library_name} STBTT_RASTERIZER_VERSION "")

    target_variable_definition(${library_name} STBTT_ifloor "")    
    target_variable_definition(${library_name} STBTT_iceil "")    
    target_variable_definition(${library_name} STBTT_sqrt "")    
    target_variable_definition(${library_name} STBTT_pow "")    
    target_variable_definition(${library_name} STBTT_fmod "")    
    target_variable_definition(${library_name} STBTT_cos "")    
    target_variable_definition(${library_name} STBTT_acos "")    
    target_variable_definition(${library_name} STBTT_fabs "")    
    target_variable_definition(${library_name} STBTT_malloc "")    
    target_variable_definition(${library_name} STBTT_free "")    
    target_variable_definition(${library_name} STBTT_assert "")    
    target_variable_definition(${library_name} STBTT_strlen "")    
    target_variable_definition(${library_name} STBTT_memcpy "")    
    target_variable_definition(${library_name} STBTT_memset "") 
  elseif(${library_name} STREQUAL "stb_voxel_render")   
    target_variable_option(${library_name} STB_VOXEL_RENDER_STATIC OFF)

    target_variable_definition(${library_name} STBVOX_CONFIG_MODE "")
    target_variable_definition(${library_name} STBVOX_CONFIG_PRECISION_Z "")

    target_variable_option(${library_name} STBVOX_CONFIG_BLOCKTYPE_SHORT OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_OPENGL_MODELVIEW OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_HLSL OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_PREFER_TEXBUFFER OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_LIGHTING_SIMPLE OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_LIGHTING OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_FOG_SMOOTHSTEP OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_FOG OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_DISABLE_TEX2 OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_TEX1_EDGE_CLAMP OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_TEX2_EDGE_CLAMP OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_ROTATION_IN_LIGHTING OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_VHEIGHT_IN_LIGHTING OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_PREMULTIPLIED_ALPHA OFF)
    target_variable_option(${library_name} STBVOX_CONFIG_UNPREMULTIPLY OFF)
  elseif(${library_name} STREQUAL "stretcy_buffer")
    target_variable_option(${library_name} STRETCHY_BUFFER_NO_SHORT_NAMES OFF)
    target_variable_definition(${library_name} STRETCHY_BUFFER_OUT_OF_MEMORY "")
  endif()
endforeach()

##############################################
# Installation instructions

include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/STB)

set(stb_targets)

foreach(library ${STB_LIBRARIES})
  list(GET library 0 library_name)
  list(GET library 1 library_definition)
  list(GET library 2 library_header)

  string(TOUPPER ${library_name} library_name_upper)

  if(NOT ${BUILD_${library_name_upper}})
    continue()
  endif()

  list(APPEND stb_targets ${library_name})
endforeach()

install(TARGETS ${stb_targets}
  EXPORT stb-targets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

foreach(library ${STB_LIBRARIES})
  list(GET library 0 library_name)
  list(GET library 1 library_definition)
  list(GET library 2 library_header)

  string(TOUPPER ${library_name} library_name_upper)

  if(NOT ${BUILD_${library_name_upper}})
    continue()
  endif()

  install(DIRECTORY src/${library_name} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
endforeach()

#Export the targets to a script
install(EXPORT stb-targets
  FILE
    STBTargets.cmake
  NAMESPACE
    STB::
  DESTINATION
    ${INSTALL_CONFIGDIR}
)

#Create a ConfigVersion.cmake file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/STBConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/STBConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/STBConfig.cmake
  INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

#Install the config, configversion and custom find modules
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/STBConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/STBConfigVersion.cmake
  DESTINATION ${INSTALL_CONFIGDIR}
)

##############################################
## Exporting from the build tree
export(EXPORT stb-targets FILE ${CMAKE_CURRENT_BINARY_DIR}/STBTargets.cmake NAMESPACE STB::)

#Register package in user's package registry
export(PACKAGE STB)
