cmake_minimum_required(VERSION 3.5.1)

add_library(cyber_routine_context detail/routine_context.cc)
target_link_libraries(cyber_routine_context cyber::common)
add_library(cyber::routine_context ALIAS cyber_routine_context)

add_library(cyber_routine_factory routine_factory.h)
set_target_properties(cyber_routine_factory PROPERTIES LINKER_LANGUAGE CXX)
target_link_libraries(cyber_routine_context cyber::common
                                            cyber::perf_event_cache)
add_library(cyber::routine_factory ALIAS cyber_routine_factory)

if($ENV{CPU} MATCHES "x86")
set(SWAP_SRCS detail/swap_x86_64.S)
elseif($ENV{CPU} MATCHES "k8")
set(SWAP_SRCS detail/swap_aarch64.S)
endif()

set(SWAP_SRCS detail/swap_x86_64.S)

add_library(cyber_swap ${SWAP_SRCS})
add_library(cyber::swap ALIAS cyber_swap)

add_library(cyber_croutine croutine.cc)
target_link_libraries(cyber_croutine atomic
                                     cyber::atomic_hash_map
                                     cyber::atomic_rw_lock
                                     cyber::bounded_queue
                                     cyber::concurrent_object_pool
                                     cyber::base_macros
                                     cyber::wait_strategy
                                     cyber::common
                                     cyber::routine_context
                                     cyber::routine_factory
                                     cyber::swap
                                     cyber::perf_event_cache
                                     cyber::time)
add_library(cyber::croutine ALIAS cyber_croutine)

## gtest
add_executable(croutine_test croutine_test.cc)
target_link_libraries(croutine_test gtest gtest_main gmock cyber::cyber)


install(TARGETS cyber_routine_context
                cyber_routine_factory
                cyber_swap
                cyber_croutine
    croutine_test
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION .
        )
