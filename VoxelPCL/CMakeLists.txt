find_package(Voxel ${VOXEL_VERSION} REQUIRED)
find_package(PCL 1.8 REQUIRED COMPONENTS common io visualization)

IF(WINDOWS)
#CMAKE_POLICY(SET CMP0028 OLD)
FIND_PACKAGE(Qt5Widgets REQUIRED)
ENDIF()

add_definitions(${PCL_DEFINITIONS})

add_library(voxelpcl SHARED PCLGrabber.cpp)

set_library_output_directory(voxelpcl)

add_library(Voxel::voxelpcl ALIAS voxelpcl)

target_include_directories(voxelpcl PRIVATE ${VOXEL_INCLUDE_DIRS} PUBLIC ${PCL_INCLUDE_DIRS} ${EIGEN_INCLUDE_DIRS})
target_link_libraries(voxelpcl LINK_PUBLIC ${VOXEL_LIBRARIES} LINK_PRIVATE ${PCL_COMMON_LIBRARY} ${PCL_IO_LIBRARY} ${PCL_VISUALIZATION_LIBRARY} ${Boost_SYSTEM_LIBRARY_RELEASE})

generate_export_header(voxelpcl
  EXPORT_FILE_NAME "VoxelPCLExports.h"
)

install(TARGETS voxelpcl
  EXPORT VoxelPCLTargets
  LIBRARY DESTINATION lib COMPONENT voxelpcl
  ARCHIVE DESTINATION lib COMPONENT voxelpcl
  RUNTIME DESTINATION lib COMPONENT voxelpcl
)

install(FILES
  PCLGrabber.h
  ${CMAKE_CURRENT_BINARY_DIR}/VoxelPCLExports.h
  DESTINATION include/voxel-${VOXEL_VERSION}/pcl
  COMPONENT voxelpcl_dev
)

set_target_properties(voxelpcl PROPERTIES 
                      VERSION ${VOXEL_VERSION} 
                      SOVERSION ${VOXEL_ABI_VERSION}
		      LINK_FLAGS "-undefined dynamic_lookup")


IF(LINUX)
  get_distribution_codename(codename)

  set(CPACK_COMPONENTS_ALL voxelpcl)
  set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "PCL integration with Grabber and Viewer")
  
  if(${codename} STREQUAL "trusty")
    set(CPACK_DEBIAN_PACKAGE_DEPENDS "libvoxel (>= ${VOXEL_VERSION}), libpcl-visualization-1.7, libpcl-io-1.7, libpcl-common-1.7")
  else() #${codename} STREQUAL "wily"
    set(CPACK_DEBIAN_PACKAGE_DEPENDS "libvoxel (>= ${VOXEL_VERSION}), libpcl-visualization1.7, libpcl-io1.7, libpcl-common1.7")
  endif()
  create_cpack_config(libvoxel-pcl ${VOXEL_VERSION})

  set(CPACK_COMPONENTS_ALL voxelpcl_dev)
  set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Voxel library developer package. Contains header files needed to compile applications linking to Voxel library.")
  if(${codename} STREQUAL "trusty")
    set(CPACK_DEBIAN_PACKAGE_DEPENDS "libvoxel-dev (>= ${VOXEL_VERSION}), libpcl-visualization-1.7-dev, libpcl-io-1.7-dev, libpcl-common-1.7-dev")
  else() #${codename} STREQUAL "wily"
    set(CPACK_DEBIAN_PACKAGE_DEPENDS "libvoxel-dev (>= ${VOXEL_VERSION}), libpcl-dev")
  endif()
  create_cpack_config(libvoxel-pcl-dev ${VOXEL_VERSION})

ELSEIF(APPLE)

  get_distribution_codename(codename)

  set(CPACK_COMPONENTS_ALL voxelpcl)
  set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "PCL integration with Grabber and Viewer")

  if(${codename} STREQUAL "sierra")
    set(CPACK_DEBIAN_PACKAGE_DEPENDS "libvoxel (>= ${VOXEL_VERSION}), libpcl-visualization, libpcl-io, libpcl-common")
  endif()
  create_cpack_config(libvoxel-pcl ${VOXEL_VERSION})

  set(CPACK_COMPONENTS_ALL voxelpcl_dev)
  set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Voxel library developer package. Contains header files needed to compile applications linking to Voxel library.")
  if(${codename} STREQUAL "sierra")
    set(CPACK_DEBIAN_PACKAGE_DEPENDS "libvoxel-dev (>= ${VOXEL_VERSION}), libpcl-visualization, libpcl-io, libpcl-common")
  endif()
  create_cpack_config(libvoxel-pcl-dev ${VOXEL_VERSION})

ENDIF()

create_cmake_config(VoxelPCL "voxelpcl" voxelpcl_dev VoxelPCL)
