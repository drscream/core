# Copyright (C) 2013-2018 Christian Dywan <christian@twotoasts.de>

set(EXTENSIONDIR "${CMAKE_INSTALL_FULL_LIBDIR}/${CMAKE_PROJECT_NAME}")
include_directories(
                    "${CMAKE_SOURCE_DIR}"
                    "${CMAKE_SOURCE_DIR}/core"
                    ${DEPS_INCLUDE_DIRS}
                    ${DEPS_GTK_INCLUDE_DIRS}
                    ${CMAKE_BINARY_DIR}
                    "${CMAKE_BINARY_DIR}/core"
                    )
file(GLOB EXTENSIONS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.c *.vala)
foreach(UNIT_SRC ${EXTENSIONS})
    if (${UNIT_SRC} MATCHES "(.vala)$")
        string(REPLACE ".vala" "" UNIT ${UNIT_SRC})
        include(ValaPrecompile)
        vala_precompile(UNIT_SRC_C ${UNIT}
            ${UNIT_SRC}
        PACKAGES
            ${PKGS}
        OPTIONS
            ${VALAFLAGS}
        CUSTOM_VAPIS
            ${CMAKE_BINARY_DIR}/core/${LIBCORE}.vapi
            ${EXTRA_VAPIS}
        )

        add_library(${UNIT} MODULE ${UNIT_SRC_C})
        set_target_properties(${UNIT} PROPERTIES
                              COMPILE_FLAGS "${VALA_CFLAGS}"
                              )
    else()
        string(REPLACE ".c" "" UNIT ${UNIT_SRC})
        add_library(${UNIT} MODULE ${UNIT_SRC})
        set_target_properties(${UNIT} PROPERTIES
                              COMPILE_FLAGS ${CFLAGS}
                              )
    endif()
    target_link_libraries(${UNIT}
                          ${DEPS_LIBRARIES}
                          ${DEPS_GTK_LIBRARIES}
                          ${LIBCORE}
                          )
    install(TARGETS ${UNIT}
            LIBRARY DESTINATION ${EXTENSIONDIR}
            )
    set(MANIFEST "${UNIT}.plugin")
    configure_file(${MANIFEST}.in ${MANIFEST})
    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${MANIFEST}
            DESTINATION ${EXTENSIONDIR}
            )
endforeach ()
