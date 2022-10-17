cmake_minimum_required(VERSION 3.5)
project(ros_workspace NONE)

find_package(ament_cmake_core REQUIRED)

find_package (Python COMPONENTS Interpreter Development)

# Locate ament_package template files.
if(WIN32)
  set(PYTHON_INSTALL_DIR "lib/site-packages")
  set(SHELL_EXT "bat")
else()
  set(PYTHON_INSTALL_DIR "lib/python${Python_VERSION_MAJOR}${Python_VERSION_MINOR}/site-packages")
  set(SHELL_EXT "sh")
endif()
set(AMENT_PACKAGE_DIR "${Python_SITELIB}/ament_package")
if(NOT EXISTS "${AMENT_PACKAGE_DIR}")
  # Check for an .egg-link file and use the listed directory if it exists
  get_filename_component(AMENT_PACKAGE_EGG_LINK "${AMENT_PACKAGE_DIR}" DIRECTORY)
  set(AMENT_PACKAGE_EGG_LINK "${AMENT_PACKAGE_EGG_LINK}/ament-package.egg-link")
  if(NOT EXISTS "${AMENT_PACKAGE_EGG_LINK}")
    message(FATAL_ERROR "Could neither find '${AMENT_PACKAGE_DIR}' nor '${AMENT_PACKAGE_EGG_LINK}'")
  endif()
  file(STRINGS "${AMENT_PACKAGE_EGG_LINK}" AMENT_PACKAGE_DIR)
  list(GET AMENT_PACKAGE_DIR 0 AMENT_PACKAGE_DIR)
  set(AMENT_PACKAGE_DIR "${AMENT_PACKAGE_DIR}/ament_package")
  if(NOT EXISTS "${AMENT_PACKAGE_DIR}")
    message(FATAL_ERROR "Could not find '${AMENT_PACKAGE_DIR}' listed in '${AMENT_PACKAGE_EGG_LINK}'")
  endif()
endif()
set(AMENT_PACKAGE_TEMPLATE_DIR "${AMENT_PACKAGE_DIR}/template")
set(BINARY_PATH_HOOK "${AMENT_PACKAGE_TEMPLATE_DIR}/environment_hook/path.${SHELL_EXT}")
set(PYTHONPATH_HOOK "${AMENT_PACKAGE_TEMPLATE_DIR}/environment_hook/pythonpath.${SHELL_EXT}.in")

# register information for .dsv generation
set(
  AMENT_CMAKE_ENVIRONMENT_HOOKS_DESC_pythonpath
  "prepend-non-duplicate;PYTHONPATH;${PYTHON_INSTALL_DIR}")

# Set environment hooks for default environment.
if(WIN32)
  ament_environment_hooks("${BINARY_PATH_HOOK}" "${PYTHONPATH_HOOK}")
else()
  set(LIBRARY_PATH_HOOK "${AMENT_PACKAGE_TEMPLATE_DIR}/environment_hook/library_path.${SHELL_EXT}")
  # enable C language so that a trycompile can determine what the
  # anticipated libdir will be.
  enable_language(C)
  include(GNUInstallDirs)
  if(NOT ${CMAKE_INSTALL_LIBDIR} STREQUAL "lib")
    # register multiarch information for .dsv generation
    if(APPLE)
      set(LIBRARY_PATH_ENV_VAR "DYLD_LIBRARY_PATH")
    else()
      set(LIBRARY_PATH_ENV_VAR "LD_LIBRARY_PATH")
    endif()
    set(
      AMENT_CMAKE_ENVIRONMENT_HOOKS_DESC_multiarch_library_paths
      "prepend-non-duplicate;${LIBRARY_PATH_ENV_VAR};${CMAKE_INSTALL_LIBDIR}")
    set(MULTIARCH_LIBRARY_PATH_HOOK "env-hooks/multiarch_library_paths.sh.in")
  endif()
  ament_environment_hooks("${BINARY_PATH_HOOK}" "${LIBRARY_PATH_HOOK}" "${PYTHONPATH_HOOK}" ${MULTIARCH_LIBRARY_PATH_HOOK})
endif()

# skip using ament_index/resource_index/parent_prefix_path
# if for Debian packages it is known that there are no underlays
# note the template only checks for an empty (use) / non-empty (skip) string
set(SKIP_PARENT_PREFIX_PATH "SKIP_PARENT_PREFIX_PATH")

ament_package()
ament_generate_environment()

