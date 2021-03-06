cmake_minimum_required( VERSION 3.7.0 )

option (VRAYGOLAEM_STANDALONE_BUILD "Standalone build: ON/OFF" ON)

if(VRAYGOLAEM_STANDALONE_BUILD)
	############################################################
	# BEGIN Declare various macros
	############################################################
	macro( list_files FOLDER_NAME FILE_EXTENSIONS )

		set( SKIP_INCLUDE_DIR OFF )
		if( ( ${ARGC} GREATER 2 ) AND ( NOT "${ARGV2}" STREQUAL "" ) )
			set( DIRECTORY_REFERENCE "${ARGV2}/${FOLDER_NAME}" )
			if( ( ${ARGC} GREATER 3 ) AND ( "${ARGV3}" STREQUAL "SKIP_INCLUDE_DIR" ) )
				set( SKIP_INCLUDE_DIR ON )
			endif()
		else()
			set( DIRECTORY_REFERENCE "${CMAKE_CURRENT_SOURCE_DIR}/${FOLDER_NAME}" )
		endif()
		if( EXISTS ${DIRECTORY_REFERENCE} )
			if( NOT SKIP_INCLUDE_DIR )
				include_directories( ${DIRECTORY_REFERENCE} )
			endif()
			if( "${FOLDER_NAME}" STREQUAL "." )
				set( DIRECTORY_FILTER "Source Files" )
				set( DIRECTORY_LABEL "ROOT" )
			else()
				set( DIRECTORY_FILTER "Source Files/${FOLDER_NAME}" )
				string( REGEX REPLACE "/\\." "" DIRECTORY_FILTER ${DIRECTORY_FILTER} )
				string( REGEX REPLACE "\\./" "" DIRECTORY_FILTER ${DIRECTORY_FILTER} )
				string( REGEX REPLACE "/" "\\\\\\\\" DIRECTORY_FILTER ${DIRECTORY_FILTER} )
				set( DIRECTORY_LABEL "${FOLDER_NAME}" )
				string( REGEX REPLACE "/\\." "" DIRECTORY_LABEL ${DIRECTORY_LABEL} )
				string( REGEX REPLACE "\\./" "" DIRECTORY_LABEL ${DIRECTORY_LABEL} )
				string( REGEX REPLACE "/" "_" DIRECTORY_LABEL ${DIRECTORY_LABEL} )
				string( TOUPPER "${DIRECTORY_LABEL}" DIRECTORY_LABEL )
			endif()
			foreach( file_ext ${FILE_EXTENSIONS} )
				string( TOUPPER "${file_ext}" _FILE_EXT )
				unset( LIST_FILES_${DIRECTORY_LABEL}_${_FILE_EXT} )
				file( GLOB LIST_FILES_${DIRECTORY_LABEL}_${_FILE_EXT} ${DIRECTORY_REFERENCE}/*.${file_ext} )
				list( APPEND LIST_FILES "${LIST_FILES_${DIRECTORY_LABEL}_${_FILE_EXT}}" )
				list( APPEND LIST_FILES_${_FILE_EXT} "${LIST_FILES_${DIRECTORY_LABEL}_${_FILE_EXT}}" )
				source_group( "${DIRECTORY_FILTER}" FILES ${LIST_FILES_${DIRECTORY_LABEL}_${_FILE_EXT}} )
				# unset( LIST_FILES_${DIRECTORY_LABEL}_${_FILE_EXT} )
			endforeach()
		else()
		endif()

	endmacro()

	############################################################
	# END Declare various macros
	############################################################


	############################################################
	# BEGIN Project
	############################################################
	project (vraygolaem)

	add_definitions(-DWITH_DR1)

	set(CMAKE_CXX_FLAGS "/DWIN32 /D_WINDOWS /GR /EHsc /MD")

	# Enable multi core compilation
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")

	if(MSVC_VERSION VERSION_GREATER 1800)
		# Write pdb from multiple cl.exe instances
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /FS")
	endif()

	set(CMAKE_CXX_FLAGS_RELEASE "/Ob1 /W0 /MD")
	set(CMAKE_CXX_FLAGS_DEBUG   "/Ob0 /Od /RTC1 /Zi")

	set(CMAKE_C_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")
	set(CMAKE_C_FLAGS_DEBUG   "${CMAKE_CXX_FLAGS_DEBUG}")
	set(CMAKE_C_FLAGS         "${CMAKE_CXX_FLAGS}")

	# Automatically install
	set( CMAKE_VS_INCLUDE_INSTALL_TO_DEFAULT_BUILD 1 )

	# Add dependencies
	set( 3DSMAXSDK_ROOTDIR "3dsMaxSDK Root Dir" CACHE PATH "3dsMaxSDK Root directory" )
	set( 3DSMAXSDK_VERSION 2014 CACHE STRING "3dsMaxSDK Version" )
	set( VRAYFOR3DSMAX_ROOTDIR "VRay Root Dir" CACHE PATH "VRay Root directory" )
	set( GOLAEMDEVKIT_ROOTDIR "Golaem Devkit Root Dir" CACHE PATH "Golaem DevKit directory" )

	set( 3DSMAXSDK_INCDIR "${3DSMAXSDK_ROOTDIR}/include" "${3DSMAXSDK_ROOTDIR}/include/maxscript" CACHE PATH "")
	set( 3DSMAXSDK_LIBDIR "${3DSMAXSDK_ROOTDIR}/lib/x64/Release"  CACHE PATH "")
	set( VRAYFOR3DSMAX_INCDIR "${VRAYFOR3DSMAX_ROOTDIR}/include" "${VRAYFOR3DSMAX_ROOTDIR}/include/" )
	set( VRAYFOR3DSMAX_LIBDIR "${VRAYFOR3DSMAX_ROOTDIR}/lib/x64" )
	set( GOLAEMDEVKIT_INCDIR "${GOLAEMDEVKIT_ROOTDIR}/include" )
	set( GOLAEMDEVKIT_LIBDIR "${GOLAEMDEVKIT_ROOTDIR}/lib" )

	project( Golaem-Max${3DSMAXSDK_VERSION})

	link_directories(
		${VRAYFOR3DSMAX_LIBDIR}/vc11
		${VRAYFOR3DSMAX_LIBDIR}/vc14
		${VRAYFOR3DSMAX_LIBDIR}
	)

	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/assetmanagement.lib")
	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/bmm.lib")
	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/core.lib")
	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/maxutil.lib")
	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/geom.lib")
	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/gfx.lib")
	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/mesh.lib")
	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/maxscrpt.lib")
	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/manipsys.lib")
	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/paramblk2.lib")
	list(APPEND 3DSMAXSDK_LIBS "${3DSMAXSDK_LIBDIR}/MNMath.lib")

	list(APPEND VRAYFOR3DSMAX_LIBS "vray${3DSMAXSDK_VERSION}")
	list(APPEND VRAYFOR3DSMAX_LIBS "vrender${3DSMAXSDK_VERSION}")
	list(APPEND VRAYFOR3DSMAX_LIBS "zlib_s")
	list(APPEND VRAYFOR3DSMAX_LIBS "tiff_s")
	list(APPEND VRAYFOR3DSMAX_LIBS "plugman_s")
	list(APPEND VRAYFOR3DSMAX_LIBS "vutils_s")
	list(APPEND VRAYFOR3DSMAX_LIBS "meshes_s")
	list(APPEND VRAYFOR3DSMAX_LIBS "putils_s")

	list(APPEND GOLAEMDEVKIT_LIBS "${GOLAEMDEVKIT_LIBDIR}/glmCrowdIO.lib")

	list(APPEND SYSTEM_LIBS "comctl32.lib")

	#MESSAGE(${3DSMAXSDK_INCDIR})

	set(TARGET_CROWD3DSMAXVRAY "vraygolaem${3DSMAXSDK_VERSION}")
	set(TARGET_CROWD3DSMAXVRAYHSL "vraygolaemhsl${3DSMAXSDK_VERSION}")
	set(TARGET_CROWD3DSMAXVRAYSWITCH "vraygolaemswitch${3DSMAXSDK_VERSION}")

	add_subdirectory (vrayGolaem)
	add_subdirectory (vrayGolaemHSL)
	add_subdirectory (vrayGolaemSwitch)
endif()

############################################################
# END Project
############################################################
