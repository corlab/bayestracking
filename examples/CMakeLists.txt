include_directories(../include)

find_package( OpenCV REQUIRED )

if(${OpenCV_FOUND})
  add_executable(simple_2d_tracking simple_2d_tracking.cpp)

  target_link_libraries(simple_2d_tracking bayes_tracking ${OpenCV_LIBS})
else()
  message(WARNING "Cannot build example due to missing OpenCV!")
endif(${OpenCV_FOUND})
