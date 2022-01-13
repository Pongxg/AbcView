cmake_minimum_required (VERSION 3.9)

project (Imath)

set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Zi")
set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} /DEBUG /OPT:REF /OPT:ICF")
set(CMAKE_CXX_STANDARD 14)


# Find source files
#
file (GLOB h_src "*.h" )
file (GLOB cpp_src "*.cpp" )

include_directories (
	.
)

# Make Source groups
source_group ("Header Files" FILES ${h_src})
source_group ("Source Files" FILES ${cpp_src})

add_library (${PROJECT_NAME} STATIC 
	${h_src} 
	${cpp_src} 
)

target_link_libraries (${PROJECT_NAME})


