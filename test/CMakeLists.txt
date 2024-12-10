
add_executable(test_main ${CMAKE_CURRENT_SOURCE_DIR}/test_main.cpp)
target_link_libraries(test_main PUBLIC template_lib)

if(ENABLE_CGAL)
add_executable(test_main ${CMAKE_CURRENT_SOURCE_DIR}/test_main.cpp)
target_link_libraries(test_main PUBLIC CGAL::CGAL CGAL::CGAL_Core template_lib)

if(ENABLE_TBB)
	target_include_directories(test_main PUBLIC ${EXTERNAL_DIR}/tbb/include)
	if(TARGET CGAL::TBB_support)
		target_link_libraries(test_main PUBLIC CGAL::TBB_support)
	endif()
endif()

endif()


