include_directories(../libjpeg_api .)


###############################################################################
# TARGETS
###############################################################################

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

foreach(file ${JPEG_SOURCES})
  set(JPEG_SOURCES_TJ ${JPEG_SOURCES_TJ} ../libjpeg_api/${file})
endforeach()

if(ENABLE_SHARED)
  set(TURBOJPEG_SOURCES ${JPEG_SOURCES_TJ} $<TARGET_OBJECTS:simd> ${SIMD_OBJS}
    turbojpeg.c ../libjpeg_api/transupp.c jdatadst-tj.c jdatasrc-tj.c
    ../libjpeg_api/rdbmp.c ../libjpeg_api/rdppm.c ../libjpeg_api/wrbmp.c
    ../libjpeg_api/wrppm.c)
  set(TJMAPFILE ${CMAKE_CURRENT_SOURCE_DIR}/turbojpeg-mapfile)
  if(WITH_JAVA)
    set(TURBOJPEG_SOURCES ${TURBOJPEG_SOURCES} turbojpeg-jni.c)
    include_directories(${JAVA_INCLUDE_PATH} ${JAVA_INCLUDE_PATH2})
    set(TJMAPFILE ${CMAKE_CURRENT_SOURCE_DIR}/turbojpeg-mapfile.jni)
  endif()
  add_library(turbojpeg SHARED ${TURBOJPEG_SOURCES})
  set_property(TARGET turbojpeg PROPERTY COMPILE_FLAGS
    "-DBMP_SUPPORTED -DPPM_SUPPORTED")
  if(WIN32)
    set_target_properties(turbojpeg PROPERTIES DEFINE_SYMBOL DLLDEFINE)
  endif()
  if(MINGW)
    set_target_properties(turbojpeg PROPERTIES LINK_FLAGS -Wl,--kill-at)
  endif()
  if(APPLE)
    if(NOT CMAKE_SHARED_LIBRARY_RUNTIME_C_FLAG)
      set(CMAKE_SHARED_LIBRARY_RUNTIME_C_FLAG "-Wl,-rpath,")
    endif()
    set_target_properties(turbojpeg PROPERTIES MACOSX_RPATH 1)
  endif()
  set_target_properties(turbojpeg PROPERTIES
    SOVERSION ${TURBOJPEG_SO_MAJOR_VERSION} VERSION ${TURBOJPEG_SO_VERSION})
  if(TJMAPFLAG)
    set_target_properties(turbojpeg PROPERTIES
      LINK_FLAGS "${TJMAPFLAG}${TJMAPFILE}")
  endif()

  add_executable(tjunittest tjunittest.c tjutil.c ../md5/md5.c ../md5/md5hl.c)
  target_link_libraries(tjunittest turbojpeg)

  add_executable(tjbench tjbench.c tjutil.c)
  target_link_libraries(tjbench turbojpeg)
  if(UNIX)
    target_link_libraries(tjbench m)
  endif()

  add_executable(tjexample tjexample.c)
  target_link_libraries(tjexample turbojpeg)
endif()

if(ENABLE_STATIC)
  add_library(turbojpeg-static STATIC ${JPEG_SOURCES_TJ} $<TARGET_OBJECTS:simd>
    ${SIMD_OBJS} turbojpeg.c ../libjpeg_api/transupp.c jdatadst-tj.c
    jdatasrc-tj.c ../libjpeg_api/rdbmp.c ../libjpeg_api/rdppm.c
    ../libjpeg_api/wrbmp.c ../libjpeg_api/wrppm.c)
  set_property(TARGET turbojpeg-static PROPERTY COMPILE_FLAGS
    "-DBMP_SUPPORTED -DPPM_SUPPORTED")
  if(NOT MSVC)
    set_target_properties(turbojpeg-static PROPERTIES OUTPUT_NAME turbojpeg)
  endif()

  add_executable(tjunittest-static tjunittest.c tjutil.c ../md5/md5.c
    ../md5/md5hl.c)
  target_link_libraries(tjunittest-static turbojpeg-static)

  add_executable(tjbench-static tjbench.c tjutil.c)
  target_link_libraries(tjbench-static turbojpeg-static)
  if(UNIX)
    target_link_libraries(tjbench-static m)
  endif()
endif()


###############################################################################
# TESTS
###############################################################################

configure_file(tjbenchtest.in ${CMAKE_BINARY_DIR}/tjbenchtest @ONLY)
configure_file(tjexampletest.in ${CMAKE_BINARY_DIR}/tjexampletest @ONLY)
if(WIN32)
  set(BASH bash)
endif()
if(WITH_JAVA)
  configure_file(tjbenchtest.java.in ${CMAKE_BINARY_DIR}/tjbenchtest.java
    @ONLY)
  configure_file(tjexampletest.java.in ${CMAKE_BINARY_DIR}/tjexampletest.java
    @ONLY)
  add_custom_target(tjtest
    COMMAND echo tjbenchtest
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest
    COMMAND echo tjbenchtest -alloc
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest -alloc
    COMMAND echo tjbenchtest -yuv
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest -yuv
    COMMAND echo tjbenchtest -yuv -alloc
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest -yuv -alloc
    COMMAND echo tjbenchtest -progressive
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest -progressive
    COMMAND echo tjexampletest
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjexampletest
    COMMAND echo tjbenchtest.java
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest.java
    COMMAND echo tjbenchtest.java -yuv
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest.java -yuv
    COMMAND echo tjbenchtest.java -progressive
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest.java -progressive
    COMMAND echo tjexampletest.java
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjexampletest.java
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest
      ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest.java
      ${CMAKE_CURRENT_BINARY_DIR}/tjexampletest)
else()
  add_custom_target(tjtest
    COMMAND echo tjbenchtest
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest
    COMMAND echo tjbenchtest -alloc
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest -alloc
    COMMAND echo tjbenchtest -yuv
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest -yuv
    COMMAND echo tjbenchtest -yuv -alloc
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest -yuv -alloc
    COMMAND echo tjexampletest
    COMMAND ${BASH} ${CMAKE_CURRENT_BINARY_DIR}/tjexampletest
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/tjbenchtest)
endif()


###############################################################################
# INSTALLATION
###############################################################################

if(WIN32)
  set(EXE ".exe")
endif()

if(ENABLE_SHARED)
  install(TARGETS turbojpeg tjbench
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
endif()
if(ENABLE_STATIC)
  install(TARGETS turbojpeg-static ARCHIVE
    DESTINATION ${CMAKE_INSTALL_LIBDIR})
  if(NOT ENABLE_SHARED)
    install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/tjbench-static${EXE}
      DESTINATION ${CMAKE_INSTALL_BINDIR} RENAME tjbench${EXE})
  endif()
endif()
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/turbojpeg.h
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/tjexample.c
  DESTINATION ${CMAKE_INSTALL_DOCDIR})

if(UNIX OR MINGW)
  install(FILES ${CMAKE_BINARY_DIR}/pkgscripts/libturbojpeg.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
endif()
