cmake_minimum_required(VERSION 3.27)

# bsplugins includes uibase headers unprefixed (e.g. <iplugin.h> rather than
# <uibase/iplugin.h>); opt in to the compatibility include dirs uibase still
# offers for that instead of rewriting every #include across the plugin.
set(MO2_CMAKE_DEPRECATED_UIBASE_INCLUDE ON)

find_package(mo2-cmake CONFIG REQUIRED)
find_package(mo2-uibase CONFIG REQUIRED)
find_package(mo2-bsatk CONFIG REQUIRED)
find_package(mo2-lootcli-header CONFIG REQUIRED)
find_package(Boost REQUIRED CONFIG COMPONENTS thread interprocess)
find_package(ZLIB REQUIRED)
find_package(Qt6 CONFIG REQUIRED COMPONENTS WebEngineWidgets)

include(FetchContent)

FetchContent_Declare(
	ryml
	EXCLUDE_FROM_ALL
	GIT_REPOSITORY https://github.com/biojppm/rapidyaml.git
	GIT_TAG master
)

FetchContent_MakeAvailable(ryml)

include(${CMAKE_CURRENT_SOURCE_DIR}/ParseTES.cmake)

add_library(bsplugins SHARED)

set_property(TARGET bsplugins PROPERTY CXX_STANDARD 20)
target_sources(bsplugins PRIVATE ${TES_INCLUDE_FILES})
target_include_directories(
	bsplugins
	PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}
	PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/include
)
target_link_libraries(
	bsplugins
	PRIVATE
		ryml
		mo2::uibase mo2::bsatk mo2::lootcli-header
		Boost::thread Boost::interprocess
		ZLIB::ZLIB
		Qt6::WebEngineWidgets
)

if(MSVC)
	target_compile_options(
		bsplugins
		PRIVATE
			/bigobj # increase number of sections in .obj file
			/W4     # level 4 warnings
			/WX     # treat all compiler warnings as errors
			/wd4201 # nonstandard extension used: nameless struct/union
			/wd4458 # declaration of 'identifier' hides class member
	)
endif()

mo2_configure_plugin(bsplugins WARNINGS OFF)
mo2_install_plugin(bsplugins)
# ryml is fetched as a shared library (ryml.dll), so bsplugins.dll has a runtime
# dependency nothing else in the tree installs. A plugin DLL is loaded well after
# process startup via QPluginLoader, which does not add the loading DLL's own
# directory to the search path (that's only true of the exe's own imports) -- so a
# copy sitting next to bsplugins.dll in plugins/ would not be found. bin/dlls IS on
# the search path: MOApplication::addDllsToPath() prepends it to PATH at startup,
# which is how Qt's own DLLs resolve despite living outside bin/. Same fix as the
# root CMakeLists.txt's 7zip.dll install.
install(FILES "$<TARGET_FILE:ryml>" DESTINATION ${MO2_INSTALL_BIN}/dlls)
