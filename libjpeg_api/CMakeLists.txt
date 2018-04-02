include_directories(.)

# Generate files
if(WIN32)
  configure_file(win/jconfig.h.in ${CMAKE_BINARY_DIR}/jconfig.h)
else()
  configure_file(jconfig.h.in ${CMAKE_BINARY_DIR}/jconfig.h)
endif()
configure_file(jconfigint.h.in ${CMAKE_BINARY_DIR}/jconfigint.h)
if(UNIX)
  configure_file(libjpeg.map.in libjpeg.map)
endif()


###############################################################################
# TARGETS
###############################################################################

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

if(ENABLE_SHARED)
  add_subdirectory(sharedlib)
endif()

if(ENABLE_STATIC)
  add_library(jpeg-static STATIC ${JPEG_SOURCES} $<TARGET_OBJECTS:simd>
    ${SIMD_OBJS})
  if(NOT MSVC)
    set_target_properties(jpeg-static PROPERTIES OUTPUT_NAME jpeg)
  endif()
endif()

if(WIN32)
  set(USE_SETMODE "-DUSE_SETMODE")
endif()
if(WITH_12BIT)
  set(COMPILE_FLAGS "-DGIF_SUPPORTED -DPPM_SUPPORTED ${USE_SETMODE}")
else()
  set(COMPILE_FLAGS "-DBMP_SUPPORTED -DGIF_SUPPORTED -DPPM_SUPPORTED -DTARGA_SUPPORTED ${USE_SETMODE}")
  set(CJPEG_BMP_SOURCES rdbmp.c rdtarga.c)
  set(DJPEG_BMP_SOURCES wrbmp.c wrtarga.c)
endif()

if(ENABLE_STATIC)
  add_executable(cjpeg-static cjpeg.c cdjpeg.c rdgif.c rdppm.c rdswitch.c
    ${CJPEG_BMP_SOURCES})
  set_property(TARGET cjpeg-static PROPERTY COMPILE_FLAGS ${COMPILE_FLAGS})
  target_link_libraries(cjpeg-static jpeg-static)

  add_executable(djpeg-static djpeg.c cdjpeg.c rdcolmap.c rdswitch.c wrgif.c
    wrppm.c ${DJPEG_BMP_SOURCES})
  set_property(TARGET djpeg-static PROPERTY COMPILE_FLAGS ${COMPILE_FLAGS})
  target_link_libraries(djpeg-static jpeg-static)

  add_executable(jpegtran-static jpegtran.c cdjpeg.c rdswitch.c transupp.c)
  target_link_libraries(jpegtran-static jpeg-static)
  set_property(TARGET jpegtran-static PROPERTY COMPILE_FLAGS "${USE_SETMODE}")
endif()

add_executable(rdjpgcom rdjpgcom.c)

add_executable(wrjpgcom wrjpgcom.c)


###############################################################################
# INSTALLATION
###############################################################################

if(WIN32)
  set(EXE ".exe")
endif()

if(ENABLE_STATIC)
  install(TARGETS jpeg-static ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
  if(NOT ENABLE_SHARED)
    install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/cjpeg-static${EXE}
      DESTINATION ${CMAKE_INSTALL_BINDIR} RENAME cjpeg${EXE})
    install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/djpeg-static${EXE}
      DESTINATION ${CMAKE_INSTALL_BINDIR} RENAME djpeg${EXE})
    install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/jpegtran-static${EXE}
      DESTINATION ${CMAKE_INSTALL_BINDIR} RENAME jpegtran${EXE})
  endif()
endif()

install(TARGETS rdjpgcom wrjpgcom RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/README.ijg
  ${CMAKE_CURRENT_SOURCE_DIR}/example.txt
  ${CMAKE_CURRENT_SOURCE_DIR}/libjpeg.txt
  ${CMAKE_CURRENT_SOURCE_DIR}/structure.txt
  ${CMAKE_CURRENT_SOURCE_DIR}/usage.txt ${CMAKE_CURRENT_SOURCE_DIR}/wizard.txt
  DESTINATION ${CMAKE_INSTALL_DOCDIR})

if(UNIX OR MINGW)
  install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/cjpeg.1
    ${CMAKE_CURRENT_SOURCE_DIR}/djpeg.1 ${CMAKE_CURRENT_SOURCE_DIR}/jpegtran.1
    ${CMAKE_CURRENT_SOURCE_DIR}/rdjpgcom.1
    ${CMAKE_CURRENT_SOURCE_DIR}/wrjpgcom.1
    DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)
  install(FILES ${CMAKE_BINARY_DIR}/pkgscripts/libjpeg.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
endif()

install(FILES ${CMAKE_BINARY_DIR}/jconfig.h
  ${CMAKE_CURRENT_SOURCE_DIR}/jerror.h ${CMAKE_CURRENT_SOURCE_DIR}/jmorecfg.h
  ${CMAKE_CURRENT_SOURCE_DIR}/jpeglib.h
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
