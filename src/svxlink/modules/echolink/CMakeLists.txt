# The name of the module without the Module prefix
set(MODNAME EchoLink)

# Module source code
set(MODSRC QsoImpl.cpp)

# ***
pkg_check_modules(DBUS dbus-1)
include_directories(${DBUS_INCLUDE_DIRS})
set(LIBS ${LIBS} ${DBUS_LIBRARIES})
set(MODSRC ${MODSRC} PatchCord.cpp BrandMeisterBridge.cpp)
# ***

# Project libraries to link to
set(LIBS ${LIBS} echolib)

# Find the TCL library
if(TCL_LIBRARY)
  set(TCL_LIBRARY_CACHED TRUE)
endif(TCL_LIBRARY)
find_package(TCL QUIET)
if(TCL_FOUND)
  if (NOT TCL_LIBRARY_CACHED)
    message("-- Found TCL: ${TCL_LIBRARY}")
  endif(NOT TCL_LIBRARY_CACHED)
else(TCL_FOUND)
  message(FATAL_ERROR "-- Could NOT find the TCL scripting language")
endif(TCL_FOUND)
set(LIBS ${LIBS} ${TCL_LIBRARY})
include_directories(${TCL_INCLUDE_PATH})

# Find the GSM codec library and include directory
find_package(GSM REQUIRED)
if(NOT GSM_FOUND)
  message(FATAL_ERROR "libgsm not found")
endif(NOT GSM_FOUND)
include_directories(${GSM_INCLUDE_DIR})
set(LIBS ${LIBS} ${GSM_LIBRARY})

string(TOUPPER MODULE_${MODNAME} VERNAME)

# Build the plugin
add_library(Module${MODNAME} MODULE Module${MODNAME}.cpp ${MODSRC})
set_target_properties(Module${MODNAME} PROPERTIES PREFIX "")
target_link_libraries(Module${MODNAME} ${LIBS})

# Deduce the version h-file target name from the module name and make the
# module depend on that target
string(REGEX REPLACE \(.\)\([A-Z]\) \\1-\\2 VERSION_TARGET_NAME ${MODNAME})
string(TOLOWER version-module-${VERSION_TARGET_NAME} VERSION_TARGET_NAME)
add_dependencies(Module${MODNAME} version-svxlink ${VERSION_TARGET_NAME})

# Install targets
install(TARGETS Module${MODNAME} DESTINATION ${SVX_MODULE_INSTALL_DIR})
install(FILES ${MODNAME}.tcl DESTINATION ${SVX_SHARE_INSTALL_DIR}/events.d)
install_if_not_exists(Module${MODNAME}.conf
  ${SVX_SYSCONF_INSTALL_DIR}/svxlink.d
  )
