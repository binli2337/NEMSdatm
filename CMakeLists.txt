list(APPEND datm_shared_files
  #Shared List:
  DATM/AtmBundleCreate.F90
  DATM/AtmFieldUtils.F90
  DATM/AtmForce.F90
  DATM/AtmGridUtils.F90
  DATM/AtmInternalFields.F90
  DATM/AtmModel.F90
  DATM/datm.F90
  DATM/LocalDefs.F90
)
if(CMAKE_Fortran_COMPILER_ID MATCHES "Intel")
  set(CMAKE_Fortran_FLAGS "-g -traceback -fp-model precise -O2 -fPIC")
  set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -assume realloc_lhs -m64 -mcmodel=small -threads -qopenmp")
endif()

# Collect IO files for appropriate IO
if(NetCDF_Fortran_FOUND)
  message(STATUS "DATM: Build with NetCDF IO")
else()
  message(FATAL_ERROR "NetCDF required")
endif()

list(APPEND lib_src_files
  ${datm_shared_files}
)

set(libName "datm")
set(moduleDir "${CMAKE_CURRENT_BINARY_DIR}/mod")

add_library(${libName} STATIC ${lib_src_files})

target_link_libraries(${libName} PUBLIC esmf MPI::MPI_Fortran)
if(NetCDF_Fortran_FOUND)
  target_link_libraries(${libName} PUBLIC NetCDF::NetCDF_Fortran)
endif()

target_compile_definitions(${libName} PRIVATE -DESMF_VERSION_MAJOR=${ESMF_VERSION_MAJOR}
					      -DESMF_VERSION_MINOR=${ESMF_VERSION_MINOR})
target_compile_definitions(${libName} PRIVATE "-DFORTRANUNDERSCORE -Dcoupled -DUSE_NETCDF")
target_include_directories(${libName} PRIVATE ${ESMF_MOD})

set_target_properties(${libName} PROPERTIES Fortran_MODULE_DIRECTORY ${moduleDir})
target_include_directories(${libName} INTERFACE
  $<BUILD_INTERFACE:${moduleDir}>
  $<INSTALL_INTERFACE:include/${libName}>)

# Install library
install(
  TARGETS ${libName}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  COMPONENT Library)
# Install compiled Fortran modules
install(DIRECTORY ${moduleDir} DESTINATION ${CMAKE_INSTALL_PREFIX})
