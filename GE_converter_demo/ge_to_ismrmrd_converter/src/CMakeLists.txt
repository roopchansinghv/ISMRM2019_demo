include_directories(${LIBXML2_INCLUDE_DIR})

include_directories(
    ${ORCHESTRA_INCLUDE_DIRS}
    ${ISMRMRD_INCLUDE_DIR}
    ${LIBXSLT_INCLUDE_DIR}
    ${LIBXML2_INCLUDE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR})

# build P-File to ISMRMRD converter library and tool
set(P2I_LIB "p2i")
add_library(${P2I_LIB} SHARED
    GERawConverter.cpp
    Plugin.cpp)
target_link_libraries(${P2I_LIB}
    ${ORCHESTRA_LIBRARIES}
    ${LIBXSLT_LIBRARIES}
    ${LIBXML2_LIBRARIES}
    dl)
install(TARGETS ${P2I_LIB} DESTINATION lib)
install(FILES SequenceConverter.h DESTINATION include/ge-tools)

set(P2I_EXE "pfile2ismrmrd")
add_executable(${P2I_EXE} main.cpp)
target_link_libraries(${P2I_EXE}
    ${P2I_LIB}
    ${ISMRMRD_LIBRARIES})
install(TARGETS ${P2I_EXE} DESTINATION bin)

set(GENERIC_PLUGIN "p2i-generic")
add_library(${GENERIC_PLUGIN} SHARED GenericConverter.cpp)
target_link_libraries(${GENERIC_PLUGIN}
    ${P2I_LIB}
    ${ISMRMRD_LIBRARIES})
install(TARGETS ${GENERIC_PLUGIN} DESTINATION lib)
install(FILES GenericConverter.h DESTINATION include/ge-tools)

install(DIRECTORY config/
        DESTINATION share/ge-tools/config)

add_subdirectory(NIHPlugins)

# API documentation
find_package(Doxygen)
if(DOXYGEN_FOUND)
    set(DOXYGEN_TARGET "doc")
    set(DOXYGEN_OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${DOXYGEN_TARGET}/html/index.html)
    set(DOXYFILE_IN ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in)
    set(DOXYFILE ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

    # configure Doxyfile.in, generating Doxyfile
    configure_file(${DOXYFILE_IN} ${DOXYFILE} @ONLY)

    add_custom_command(
        OUTPUT ${DOXYGEN_OUTPUT}
        COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYFILE}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMENT "Generating API Documentation with Doxygen..." VERBATIM
        DEPENDS ${DOXYFILE})

    add_custom_target(${DOXYGEN_TARGET} DEPENDS ${DOXYGEN_OUTPUT})
else(DOXYGEN_FOUND)
    message("Doxygen is needed to build ge-ismrmrd's API docs")
endif(DOXYGEN_FOUND)
