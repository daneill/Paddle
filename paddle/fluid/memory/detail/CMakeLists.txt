include(ExternalProject)

cc_library(memory_block SRCS memory_block.cc memory_block_desc.cc meta_cache.cc DEPS place)

if(${WITH_GPU})
  nv_library(system_allocator SRCS system_allocator.cc DEPS gflags cpu_info gpu_info place)
else(${WITH_GPU})
  cc_library(system_allocator SRCS system_allocator.cc DEPS gflags cpu_info place)
endif(${WITH_GPU})

cc_test(system_allocator_test SRCS system_allocator_test.cc DEPS system_allocator)

cc_library(buddy_allocator SRCS buddy_allocator.cc DEPS memory_block system_allocator glog)

cc_test(buddy_allocator_test SRCS buddy_allocator_test.cc DEPS buddy_allocator)

FUNCTION(file_download_and_uncompress URL NAME)
  MESSAGE(STATUS "Download dependence[${NAME}] from ${URL}")
  SET(${NAME}_INCLUDE_DIR ${THIRD_PARTY_PATH}/${NAME} PARENT_SCOPE)
  ExternalProject_Add(
      extern_download_${NAME}
      ${EXTERNAL_PROJECT_LOG_ARGS}
      PREFIX                ${THIRD_PARTY_PATH}/${NAME}
      URL                   ${URL}
      DOWNLOAD_DIR          ${THIRD_PARTY_PATH}/${NAME}
      SOURCE_DIR            ${THIRD_PARTY_PATH}/${NAME}
      DOWNLOAD_NO_PROGRESS  1
      CONFIGURE_COMMAND     ""
      BUILD_COMMAND         ""
      UPDATE_COMMAND        ""
      INSTALL_COMMAND       ""
    )
  set(third_party_deps ${third_party_deps} extern_download_${NAME} PARENT_SCOPE)
ENDFUNCTION()

if(WITH_TESTING)
  set_tests_properties(buddy_allocator_test PROPERTIES LABELS "RUN_TYPE=EXCLUSIVE")
  set(URL "https://paddle-ci.cdn.bcebos.com/buddy_allocator_test_data.tar")
  file_download_and_uncompress(URL "buddy_allocator")
endif()
