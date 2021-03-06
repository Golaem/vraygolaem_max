############################################################
# BEGIN Project
############################################################

project("${TARGET_CROWD3DSMAX${3DSMAX_VERSION}VRAYSWITCH50}")

# List Files
list_files( "." "h;cpp;c;def;rc" "${CMAKE_CURRENT_SOURCE_DIR}/../_shared")

# Create a library
add_library( ${PROJECT_NAME} SHARED ${LIST_FILES} )

# Flags
target_compile_options(${PROJECT_NAME} PRIVATE "-DGLM_USE_VRAY50")
target_compile_options(${PROJECT_NAME} PRIVATE "-DUNICODE" "-D_UNICODE")
target_compile_options(${PROJECT_NAME} PRIVATE "-DWITH_DR1")

if(NOT VRAYGOLAEM_STANDALONE_BUILD)
    crowd_label_project( ${PROJECT_NAME} )

    crowd_load_dependencies( ${PROJECT_NAME} )
    set_target_prefix( ${PROJECT_NAME} PLUGIN )
    set_target_postfix( ${PROJECT_NAME} PLUGIN )

    set_target_extension( ${PROJECT_NAME} CUSTOM ".dlt" )

	target_compile_options(${PROJECT_NAME} PRIVATE -MD -U_DEBUG)

    #CROWD_INSTALL_RUNTIME_LIBRARY( ${PROJECT_NAME} "procedurals/3dsmax" "vrayGolaemSwitch${3DSMAX_VERSION}")
    CROWD_INSTALL_PLUGIN( ${PROJECT_NAME} "procedurals/3dsmax" )
    crowd_install_dependencies()
else()
    # Include
    target_include_directories(${PROJECT_NAME} PRIVATE ${3DSMAXSDK_INCDIR_${3DSMAX_VERSION}} )
    target_include_directories(${PROJECT_NAME} PRIVATE ${VRAYFOR3DSMAX_INCDIR_${3DSMAX_VERSION}} )

    # Link
    target_link_libraries( ${PROJECT_NAME} "${SYSTEM_LIBS}")
    target_link_libraries( ${PROJECT_NAME} "${3DSMAXSDK_LIBS_${3DSMAX_VERSION}}" )
    target_link_libraries( ${PROJECT_NAME} "${VRAYFOR3DSMAX_LIBS_${3DSMAX_VERSION}}" )

    # Extension dlt
    set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".dlt")

    #Install
    install (TARGETS ${PROJECT_NAME} RUNTIME DESTINATION "${CMAKE_INSTALL_PREFIX}/vrayplugins/")
endif()

############################################################
# END Project
############################################################
