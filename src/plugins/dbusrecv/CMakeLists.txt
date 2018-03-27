include (LibAddBinding)

if (DEPENDENCY_PHASE)
	find_package (DBus)

	if (NOT DBUS_FOUND)
		remove_plugin (dbusrecv "dbus package not found")
	endif ()
endif ()

add_plugin (dbusrecv
	SOURCES
		dbusrecv.h dbusrecv.c
		receivemessage.c
	OBJECT_SOURCES
		$<TARGET_OBJECTS:io-adapter-dbus>
	INCLUDE_DIRECTORIES
		${DBUS_INCLUDE_DIR}
		${DBUS_ARCH_INCLUDE_DIR}
	LINK_ELEKTRA
		elektra-io
		elektra-invoke
	LINK_LIBRARIES
		${DBUS_LIBRARIES}
	)

set (NOT_INCLUDED "")
plugin_check_if_included ("${PLUGIN_SHORT_NAME}")
if (DEPENDENCY_PHASE AND NOT NOT_INCLUDED)
	if (BUILD_SHARED)
		#target_compile_options(elektra-dbusrecv PRIVATE "-fPIC")
		#set_property (TARGET ${ADAPTER_OBJECT} PROPERTY CMAKE_POSITION_INDEPENDENT_CODE ON)
	endif ()
	#if (BUILD_FULL OR BUILD_STATIC)
	#
	#endif ()
endif ()

if (ADDTESTING_PHASE)
	# the test requires an IO binding; using io_uv for now
	check_binding_was_added ("io_uv" IS_INCLUDED)
	if (BUILD_TESTING AND IS_INCLUDED)
		if (BUILD_FULL OR BUILD_STATIC)
			find_package (PkgConfig)
			pkg_check_modules (LIBUV QUIET libuv)
			# libuv is present; otherwise io_uv bindings would not be added
			set (STATIC_LIBRARIES ${LIBUV_LIBRARIES})
		endif ()

		add_plugintest (dbusrecv
			TEST_LINK_LIBRARIES
				${STATIC_LIBRARIES}
			TEST_LINK_ELEKTRA
				elektra-io-uv
			)

		# add sources manually because add_plugintest does not support generator
		# expressions for additional source files
		if (BUILD_FULL OR BUILD_STATIC)
			# add sources for elektra-io-uv for static and full builds
			target_sources (testmod_dbusrecv PRIVATE $<TARGET_OBJECTS:OBJ_elektra-io-uv>)
		endif ()

		if (LIBUV_VERSION VERSION_LESS "1.0")
			target_compile_definitions (testmod_dbusrecv PRIVATE "HAVE_LIBUV0")
		else ()
			target_compile_definitions (testmod_dbusrecv PRIVATE "HAVE_LIBUV1")
		endif ()
	else ()
		message (WARNING "io_uv bindings are required for testing, test deactivated")
	endif ()
endif ()