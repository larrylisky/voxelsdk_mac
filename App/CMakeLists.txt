find_package(PCL 1.8 REQUIRED COMPONENTS common io visualization)

IF(WINDOWS)
#CMAKE_POLICY(SET CMP0028 OLD)
FIND_PACKAGE(Qt5Widgets REQUIRED)
ENDIF()

add_definitions(${PCL_DEFINITIONS})

add_executable(SimpleVoxelViewer SimpleVoxelViewer.cpp PCLViewer.cpp)
target_include_directories(SimpleVoxelViewer PUBLIC ${PCL_INCLUDE_DIRS} ${EIGEN_INCLUDE_DIRS})
target_link_libraries(SimpleVoxelViewer voxelpcl ${PCL_COMMON_LIBRARY} ${PCL_IO_LIBRARY} ${PCL_VISUALIZATION_LIBRARY})

add_executable(VoxelCLI VoxelCLI.cpp CLIManager.cpp LineNoise.cpp PCLViewer.cpp)
target_include_directories(VoxelCLI PUBLIC ${PCL_INCLUDE_DIRS} ${EIGEN_INCLUDE_DIRS})
target_link_libraries(VoxelCLI voxelpcl ${PCL_COMMON_LIBRARY} ${PCL_IO_LIBRARY} ${PCL_VISUALIZATION_LIBRARY})

IF(APPLE)
      SET_TARGET_PROPERTIES(VoxelCLI SimpleVoxelViewer PROPERTIES LINK_FLAGS "-undefined dynamic_lookup")
ENDIF()

install(TARGETS SimpleVoxelViewer VoxelCLI
  RUNTIME DESTINATION bin
  COMPONENT apps
)

IF(LINUX)
  set(CPACK_COMPONENTS_ALL apps)
  set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Voxel sample applications")
  set(CPACK_DEBIAN_PACKAGE_DEPENDS "libvoxel (>= ${VOXEL_VERSION}), libvoxel-pcl (>= ${VOXEL_VERSION})")
  create_cpack_config(libvoxel-apps ${VOXEL_VERSION})
ELSEIF(WINDOWS)
  
  INSTALL(PROGRAMS VoxelCLIStart.cmd CLIStart.cmd SimpleVoxelViewer.cmd
    DESTINATION bin
    COMPONENT apps
  )
ENDIF()
