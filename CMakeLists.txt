cmake_minimum_required(VERSION 3.15) # Or your preferred minimum version
project(DirectX9SDK LANGUAGES C CXX)

add_library(d3d9lib INTERFACE)

# Add the Include directory
target_include_directories(d3d9lib INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Include>
    $<INSTALL_INTERFACE:Include> # Adjust if you plan to install
)

# Add the correct Lib directory based on architecture (x64 or x86)
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(STATUS "Configuring DX9 SDK for 64-bit")
    target_link_directories(d3d9lib BEFORE INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Lib/x64>
        $<INSTALL_INTERFACE:Lib/x64> # Adjust if you plan to install
    )
else()
    message(STATUS "Configuring DX9 SDK for 32-bit")
    target_link_directories(d3d9lib BEFORE INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Lib/x86>
        $<INSTALL_INTERFACE:Lib/x86> # Adjust if you plan to install
    )
endif()

# Specify the core DX9 libraries to link against
# Consumers will link these automatically when linking against d3d9lib
target_link_libraries(d3d9lib INTERFACE
    d3d9
    d3dx9 # Or d3dx9d for debug, CMake usually handles this suffix based on build type
    dinput8 # Often used with DX9
    dsound
    dxguid
    dxerr # Link DxErr.lib
)

# Add common settings needed for older SDKs with modern MSVC
if(MSVC)
    # Needed for compatibility with older headers/libs
    target_compile_definitions(d3d9lib INTERFACE legacy_stdio_definitions)
    # Disable SafeSEH checks as older libraries might not be compatible
    target_link_options(d3d9lib INTERFACE /SAFESEH:NO)
    # Avoid conflicts with default runtime libraries
    target_link_options(d3d9lib INTERFACE /NODEFAULTLIB:libci.lib)
endif()

# Optional: Define a helper macro for consumers
target_compile_definitions(d3d9lib INTERFACE BUILD_WITH_D3D9)

# Optional: Add setup for dxerr library if needed separately,
# but DxErr.lib should be in the Lib/x64 or Lib/x86 folders already.

# Ensure this target is exported if you use FetchContent_MakeAvailable downstream
# FetchContent usually handles this automatically. 