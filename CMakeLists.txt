if(NOT ParaView_SOURCE_DIR)
  cmake_minimum_required(VERSION 2.8)
  set(BUILD_SHARED_LIBS ON CACHE BOOL "Build shared libraries")
  find_package(ParaView REQUIRED)
  include(${PARAVIEW_USE_FILE})
  find_package(PythonLibs REQUIRED)
endif()

find_package(PCL REQUIRED)

include_directories(${VTK_INCLUDE_DIRS})
include_directories(${VTK_PYTHON_INCLUDE_DIR})
include_directories(${PCL_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${CMAKE_CURRENT_BINARY_DIR})
add_definitions(${PCL_DEFINITIONS})


set(gui_sources)
set(gui_interfaces)
set(gui_xml)
set(gui_deps)
set(servermanager_sources)
set(servermanager_xml)
set(server_deps)

# TODO: We really need to know if pcl was configured with openni
# not just whether it was found
option(USE_OPENNI "Build OpenNI features" OFF)

if(PARAVIEW_BUILD_QT_GUI AND USE_OPENNI)
  QT4_WRAP_CPP(moc_srcs pqPCLOpenNISource.h)
  ADD_PARAVIEW_OBJECT_PANEL(
    ifaces iface_srcs
    CLASS_NAME pqPCLOpenNISource
    XML_NAME PCLOpenNISource
    XML_GROUP sources)
  list(APPEND gui_interfaces ${ifaces})
  list(APPEND gui_sources ${iface_srcs} ${moc_srcs})
endif()


list(APPEND servermanager_sources
  vtkAnnotateOBBs.cxx
  vtkPCLConversions.cxx
  vtkPCLEuclideanClusterExtraction.cxx
  vtkPCLNDTRegistration.cxx
  vtkPCLNormalEstimation.cxx
  vtkPCLRadiusOutlierRemoval.cxx
  vtkPCDReader.cxx
  vtkPCLSACSegmentationCylinder.cxx
  vtkPCLSACSegmentationPlane.cxx
  vtkPCLVoxelGrid.cxx
  )

if(USE_OPENNI)
list(APPEND servermanager_sources
  vtkPCLOpenNISource.cxx
  )
endif()

list(APPEND servermanager_xml
  xml/vtkAnnotateOBBs.xml
  xml/vtkPCLEuclideanClusterExtraction.xml
  xml/vtkPCLNDTRegistration.xml
  xml/vtkPCLNormalEstimation.xml
  xml/vtkPCLRadiusOutlierRemoval.xml
  xml/vtkPCDReader.xml
  xml/vtkPCLSACSegmentationCylinder.xml
  xml/vtkPCLSACSegmentationPlane.xml
  xml/vtkPCLVoxelGrid.xml
  xml/vtkThresholdPoints.xml
  )

if(USE_OPENNI)
list(APPEND servermanager_xml
  xml/vtkPCLOpenNISource.xml
  )
endif()

list(APPEND gui_xml
  xml/PCLFiltersGUI.xml
  )

if(USE_OPENNI)
list(APPEND gui_xml
  xml/PCLSourcesGUI.xml
  )
endif()

list(APPEND server_deps
  ${VTK_LIBRARIES}
  ${PCL_LIBRARIES}
  )

set(library_name vtkPCLFilters)
include(GenerateExportHeader)

add_paraview_plugin(${library_name} "1.0"
  GUI_INTERFACES ${gui_interfaces}
  GUI_RESOURCE_FILES ${gui_xml}
  GUI_SOURCES ${gui_sources}
  SERVER_MANAGER_XML ${servermanager_xml}
  SERVER_MANAGER_SOURCES ${servermanager_sources}
  )
generate_export_header(${library_name} EXPORT_FILE_NAME ${library_name}Module.h)


target_link_libraries(${library_name} LINK_PUBLIC ${gui_deps} ${server_deps})

if(VTK_WRAP_PYTHON)
  include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/wrap-python.cmake)
  wrap_python(${library_name} "${servermanager_sources}" ${VTK_LIBRARIES})
endif()


if(BUILD_TESTING)
  add_subdirectory(Testing)
endif()
