cmake_minimum_required(VERSION 3.10)

# Set the project name
project(FileDialogs VERSION 1.4)

# Detect if this repo is the top-level project
set(FILEDIALOGS_TOPLEVEL OFF)
if (CMAKE_SOURCE_DIR STREQUAL PROJECT_SOURCE_DIR)
  set(FILEDIALOGS_TOPLEVEL ON)
endif()

# Option to build the demo app: ON if top-level, OFF when used as a subproject
option(FILEDIALOGS_BUILD_DEMO "Build the demo executable for this library" ${FILEDIALOGS_TOPLEVEL})

# Enable Objective-C++ support
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(LIBRARY_NAME FileDialogLibrary)

set(APPLICATION_NAME FileDialogApp)

############## MAC LIBRARY AND TEST APP ########################
if (APPLE)
  # Add the source files for the static library
  set(LIBRARY_SOURCES
      ${CMAKE_CURRENT_LIST_DIR}/Library/Mac/src/FileDialogs.mm
  )

  set(APPLICATION_SOURCES 
      ${CMAKE_CURRENT_LIST_DIR}/Application/Mac/src/main.cpp
  )

  # Create a static library
  add_library( ${LIBRARY_NAME} STATIC ${LIBRARY_SOURCES} )
  
  # Make sure there's no lib...prefixed on the library name
  set_target_properties( ${LIBRARY_NAME} PROPERTIES OUTPUT_NAME ${LIBRARY_NAME} PREFIX "" )

  # Set the include directory for this library
  target_include_directories( ${LIBRARY_NAME} PUBLIC
    ${CMAKE_CURRENT_LIST_DIR}/Library/Mac/src
  )

  # Link Cocoa framework to the static library
  find_library( COCOA_LIBRARY Cocoa )
  target_link_libraries( ${LIBRARY_NAME} PRIVATE ${COCOA_LIBRARY} )

  if (FILEDIALOGS_BUILD_DEMO)
    # Add the main.cpp file for the test application
    add_executable(${APPLICATION_NAME} MACOSX_BUNDLE ${APPLICATION_SOURCES})

    # Include directories (so that main.cpp can find FileDialogs.h)
    target_include_directories(${APPLICATION_NAME} PRIVATE
      ${CMAKE_CURRENT_LIST_DIR}/Library/Mac/src
    )

    # Link the library and Cocoa framework to the executable
    target_link_libraries(${APPLICATION_NAME} PRIVATE ${LIBRARY_NAME} ${COCOA_LIBRARY})

    # Set the app's Info.plist for macOS
    set_target_properties(${APPLICATION_NAME} PROPERTIES
        MACOSX_BUNDLE_INFO_PLIST ${PROJECT_SOURCE_DIR}/Application/Mac/src/Info.plist
    )
  endif()

############## WINDOWS LIBRARY AND TEST APP ########################
elseif(WIN32)

  # Add the source files for the static library
  set(LIBRARY_SOURCES
      ${CMAKE_CURRENT_LIST_DIR}/Library/Windows/src/FileDialogs.cpp
  )

  set(APPLICATION_SOURCES 
      ${CMAKE_CURRENT_LIST_DIR}/Application/Windows/src/main.cpp
  )

  # Create a static library
  add_library(${LIBRARY_NAME} STATIC ${LIBRARY_SOURCES})

  # Set the include directory for this library
  target_include_directories( ${LIBRARY_NAME} PUBLIC
    ${CMAKE_CURRENT_LIST_DIR}/Library/Windows/src
  )

  if (FILEDIALOGS_BUILD_DEMO)
    # Add the main.cpp file for the test application
    add_executable(${APPLICATION_NAME} ${APPLICATION_SOURCES})

    # Link the library and Windows API dependencies to the executable
    target_link_libraries(${APPLICATION_NAME} PRIVATE ${LIBRARY_NAME})

    # Include directories (so that main.cpp can find FileDialogs.h)
    target_include_directories(${APPLICATION_NAME} PRIVATE
      ${CMAKE_CURRENT_LIST_DIR}/Library/Windows/src
    )
  endif()

else()
    message(FATAL_ERROR "Unsupported platform")
endif()
