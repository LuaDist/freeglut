# Copyright (C) 2007-2009 LuaDist.
# Created by Peter Kapec
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

PROJECT(freeglut C)
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
INCLUDE(dist.cmake)

FIND_PACKAGE(X11)

###########################################################################################

# generating config.h file:
MESSAGE("Configure: FreeGLUT - generating config.h:")
	INCLUDE (CheckIncludeFiles)
	INCLUDE (CheckFunctionExists)
	INCLUDE (CheckSymbolExists)

	CHECK_INCLUDE_FILES (dlfcn.h HAVE_DLFCN_H)
	CHECK_INCLUDE_FILES (fcntl.h HAVE_FCNTL_H)
	CHECK_INCLUDE_FILES ("GL/glu.h" HAVE_GL_GLU_H)
	CHECK_INCLUDE_FILES ("GL/glx.h" HAVE_GL_GLX_H)
	CHECK_INCLUDE_FILES ("GL/gl.h" HAVE_GL_GL_H)
	CHECK_INCLUDE_FILES (inttypes.h HAVE_INTTYPES_H)
	CHECK_INCLUDE_FILES (limits.h HAVE_LIMITS_H)
	CHECK_INCLUDE_FILES (memory.h HAVE_MEMORY_H)
	CHECK_INCLUDE_FILES (stdint.h HAVE_STDINT_H)
	CHECK_INCLUDE_FILES (stdlib.h HAVE_STDLIB_H)
	CHECK_INCLUDE_FILES (strings.h HAVE_STRINGS_H)
	CHECK_INCLUDE_FILES (string.h HAVE_STRING_H)
	CHECK_INCLUDE_FILES ("sys/ioctl.h" HAVE_SYS_IOCTL_H)
	CHECK_INCLUDE_FILES ("sys/param.h" HAVE_SYS_PARAM_H)
	CHECK_INCLUDE_FILES ("sys/stat.h" HAVE_SYS_STAT_H)
	CHECK_INCLUDE_FILES ("sys/time.h" HAVE_SYS_TIME_H)
	CHECK_INCLUDE_FILES ("sys/types.h" HAVE_SYS_TYPES_H)
	CHECK_INCLUDE_FILES (unistd.h HAVE_UNISTD_H)

	#/* Define to 1 if you have the `vprintf' function. */
	CHECK_FUNCTION_EXISTS(vprintf HAVE_VPRINTF)
	IF(NOT HAVE_VPRINTF)
		#/* Define to 1 if you don't have `vprintf' but do have `_doprnt.' */
		CHECK_FUNCTION_EXISTS(_doprnt HAVE_DOPRNT)
	ENDIF()

	CHECK_INCLUDE_FILES ("X11/extensions/xf86vmode.h" HAVE_X11_EXTENSIONS_XF86VMODE_H)

	SET(PACKAGE_BUGREPORT "expat-bugs@libexpat.org")
	SET(PACKAGE_NAME "freeglut")
	SET(PACKAGE_STRING "freeglut 2.4.0")
	SET(PACKAGE_TARNAME "freeglut")
	SET(PACKAGE_VERSION "2.4.0")

	CHECK_INCLUDE_FILES("stdlib.h;stdarg.h;string.h;float.h" STDC_HEADERS)

	# TODO: needs try_compile
	#/* Define to 1 if you can safely include both <sys/time.h> and <time.h>. */
#~ 	CHECK_INCLUDE_FILES ("time.h" HAVE_TIME_H)
#~ 	IF(HAVE_SYS_TIME_H AND HAVE_TIME_H)
#~ 		SET(TIME_WITH_SYS_TIME 1)
#~ 	ENDIF()

	#/* Define to 1 if the X Window System is missing or not being used. */
	IF(NOT HAVE_X11)
		SET(X_DISPLAY_MISSING 1)
	ENDIF()

	#/* Define to empty if `const' does not conform to ANSI C. */
	#cmakedefine const

# all is defined, so generate config.h
CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)

MESSAGE("Configure: FreeGLUT - done.")
# end of generating config.h file:

ADD_DEFINITIONS(-DHAVE_CONFIG_H )
INCLUDE_DIRECTORIES (${CMAKE_CURRENT_BINARY_DIR})

###########################################################################################

FIND_PACKAGE(OpenGL REQUIRED)

# FreeGLUT Library
	INCLUDE_DIRECTORIES ("include" "src")

	FILE(GLOB SRC_FREEGLUTLIB "src/*.c")

	IF(MSVC)
		SET(DEF_FILE src/freeglut.def)
	ENDIF()

	ADD_LIBRARY(freeglut SHARED ${SRC_FREEGLUTLIB} ${DEF_FILE})

	IF(CYGWIN OR MSYS OR MINGW OR MSVC)
		#ADD_DEFINITIONS(-mthreads -Wall -O2 -DNDEBUG -DWIN32 -D_MBCS -D_USRDLL -DFREEGLUT_EXPORTS)
		ADD_DEFINITIONS(-DWIN32 -DFREEGLUT_EXPORTS)
		TARGET_LINK_LIBRARIES(freeglut winmm user32 gdi32 ${OPENGL_LIBRARIES})
	ELSEIF(UNIX)
		FIND_PACKAGE(X11 REQUIRED)
		INCLUDE_DIRECTORIES(${X11_INCLUDE_DIR})
		TARGET_LINK_LIBRARIES(freeglut ${OPENGL_LIBRARIES} ${X11_LIBRARIES})
	ENDIF ()

# Install FreeGLUT Library
	INSTALL(TARGETS freeglut DESTINATION ${INSTALL_LIB} )

# Install FreeGLUT Headers
	INSTALL(DIRECTORY include/  DESTINATION ${INSTALL_INC}
			PATTERN *.am EXCLUDE
			PATTERN *.in EXCLUDE	)

# Install FindFreeGLUT.cmake
	# INSTALL(FILES FindFreeGLUT.cmake DESTINATION ${INSTALL_CMAKE})

# Install FreeGLUT Documentation
	INSTALL(DIRECTORY doc/ DESTINATION ${INSTALL_DOC}
			PATTERN *.am EXCLUDE
			PATTERN *.in EXCLUDE	)
	INSTALL(FILES AUTHORS COPYING ChangeLog INSTALL NEWS README TODO README.win32 DESTINATION ${INSTALL_DOC})
