cmake_minimum_required(VERSION 3.0.0)
project(bgfx_proj VERSION 0.1.0)
add_executable(bgfx_proj main.cpp)

#           DOWNLOAD ALL THE SUBMODULES
find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
# Update submodules as needed
    option(GIT_SUBMODULE "Check submodules during build" ON)
    if(GIT_SUBMODULE)
        message(STATUS "Submodule update")
        execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                        RESULT_VARIABLE GIT_SUBMOD_RESULT)
        if(NOT GIT_SUBMOD_RESULT EQUAL "0")
            message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
        endif()
    endif()
endif()


# make sure they exist
if(NOT EXISTS "${PROJECT_SOURCE_DIR}/submods/bx/makefile")
	message(FATAL_ERROR "The bx submodules was not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/submods/bimg/makefile")
	message(FATAL_ERROR "The bimg submodules was not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/submods/bgfx/makefile")
	message(FATAL_ERROR "The bgfx submodules was not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/submods/bnet/makefile")
	message(FATAL_ERROR "The bnet submodules was not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()


if(UNIX)
  #include dirs
  target_include_directories(bgfx_proj 
	PRIVATE
		submods/bgfx/include
		submods/bx/include
		submods/bimg/include
  )

  target_include_directories(bgfx_proj 
	PRIVATE 
		submods/bgfx/3rdparty/
		submods/bgfx/examples/common
	)

  #LINK LIBS
  target_link_libraries(bgfx_proj GL X11 ${CMAKE_DL_LIBS} pthread)

  #BGFX LINKS LOGIC
  target_link_directories(bgfx_proj PRIVATE ./submods/bgfx/.build/linux64_gcc/bin/)

  if ("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
    if(NOT EXISTS "${PROJECT_SOURCE_DIR}/submods/bgfx/.build/linux64_gcc/bin/libbgfxDebug.a")
      message(FATAL_ERROR "The bgfx submodule was not built as debug! Go to bgfx folder and run: make linux-debug64")
    endif()
	target_link_libraries(bgfx_proj 
		bgfxDebug
		bimgDebug
		bxDebug
		example-glueDebug
		example-commonDebug
		bimg_decodeDebug
	)
  elseif("${CMAKE_BUILD_TYPE}" STREQUAL "Release")
    if(NOT EXISTS "${PROJECT_SOURCE_DIR}/submods/bgfx/.build/linux64_gcc/bin/libbgfxRelease.a")
      message(FATAL_ERROR "The bgfx submodule was not built as release! Go to bgfx folder and run: make linux-release64")
    endif()
	target_link_libraries(bgfx_proj
		example-glueRelease
		example-commonRelease
		bimg_decodeRelease
		bgfxRelease
		bimgRelease 
		bxRelease
	)
  endif()
else()
	message(FATAL_ERROR "This is for linux only right now, what's on your Hard Drive?")
endif()

file (COPY ${CMAKE_SOURCE_DIR}/submods/bgfx/examples/runtime/ DESTINATION ${CMAKE_BINARY_DIR})


