# not sure what to set this to but this seems to work
cmake_minimum_required(VERSION 3.29.2)

# get abseil to shut up about propagating the C++ standard version
set(ABSL_PROPAGATE_CXX_STD ON)

# export compile_commands.json for clangd support
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# set project name
project(libcoral)

# use C++23
set(CMAKE_CXX_STANDARD 23)

# not quite sure what this does ¯\_(ツ)_/¯
set(DOWNLOAD_EXTRACT_TIMESTAMP true)

set(TENSORFLOW_DIR "tensorflow")

# include tensorflow lite's cmake system into this project
add_subdirectory("${TENSORFLOW_DIR}/tensorflow/lite" EXCLUDE_FROM_ALL)

# find required dependencies automagically
find_package(glog REQUIRED)
find_package(GTest CONFIG REQUIRED)
find_package(PkgConfig REQUIRED)
find_package(benchmark REQUIRED)

# use pkgconfig to include GLib and GStreamer
pkg_check_modules(deps REQUIRED IMPORTED_TARGET glib-2.0)
pkg_check_modules(deps REQUIRED IMPORTED_TARGET gstreamer-1.0)

# create a library for libcoral
add_library(coral
${CMAKE_CURRENT_SOURCE_DIR}/coral/bbox_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/classification/adapter.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/classification/adapter_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/cocompiled_models_benchmark.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/detection/adapter.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/detection/adapter_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/detection/models_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/error_reporter.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/error_reporter_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/examples/file_utils.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/inference_repeatability_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/inference_stress_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/backprop/layers.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/backprop/layers_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/backprop/multi_variate_normal_distribution.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/backprop/multi_variate_normal_distribution_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/backprop/softmax_regression_model_benchmark.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/backprop/softmax_regression_model.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/backprop/softmax_regression_model_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/backprop/test_utils.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/backprop/test_utils_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/imprinting_engine.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/imprinting_engine_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/utils.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/learn/utils_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/model_benchmark_reporter.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/model_loading_stress_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/models_benchmark_lib.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/multiple_tpus_inference_stress_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pipeline/detection_models_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pipeline/detection_models_test_lib.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pipeline/internal/memory_pool_allocator.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pipeline/internal/memory_pool_allocator_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pipeline/internal/segment_runner.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pipeline/internal/segment_runner_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pipeline/pipelined_model_runner.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pipeline/pipelined_model_runner_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pipeline/test_utils.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pose_estimation/bodypix_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pose_estimation/movenet_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pose_estimation/posenet_decoder.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pose_estimation/posenet_decoder_op.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pose_estimation/posenet_decoder_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pose_estimation/posenet_decoder_tflite_plugin.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pose_estimation/posenet_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/pose_estimation/test_utils.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/segmentation_models_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/single_model_benchmark.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/test_utils.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/test_utils_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/tflite_utils.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/tflite_utils_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/automl_video_object_tracking_utils.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/model_pipelining_benchmark_util.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/partitioner/parameter_count_based_partitioner.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/partitioner/profiling_based_partitioner.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/partitioner/profiling_based_partitioner_ondevice_lib.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/partitioner/profiling_based_partitioner_ondevice_test.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/partitioner/strategy.cc
${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/partitioner/utils.cc
)

# set up libraries to link to for libcoral
target_link_libraries(coral glog::glog tensorflow-lite GTest::gmock PkgConfig::deps absl::flags_parse absl::statusor benchmark::benchmark :libedgetpu.so.1)

# include various other non-cmake subprojects
target_include_directories(coral PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/libedgetpu ${CMAKE_CURRENT_SOURCE_DIR}/libedgetpu/tflite/public)

# set up libraries to link to for our executable
link_libraries(coral)

# create the actual executables
add_executable(benchmark_main ${CMAKE_CURRENT_SOURCE_DIR}/coral/benchmark_main.cc)
add_executable(classify_image ${CMAKE_CURRENT_SOURCE_DIR}/coral/examples/classify_image.cc)
add_executable(lstpu ${CMAKE_CURRENT_SOURCE_DIR}/coral/examples/lstpu.cc)
add_executable(model_pipelining ${CMAKE_CURRENT_SOURCE_DIR}/coral/examples/model_pipelining.cc)
add_executable(test_main ${CMAKE_CURRENT_SOURCE_DIR}/coral/test_main.cc)
add_executable(two_models_one_tpu ${CMAKE_CURRENT_SOURCE_DIR}/coral/examples/two_models_one_tpu.cc)
add_executable(two_models_two_tpus_threaded ${CMAKE_CURRENT_SOURCE_DIR}/coral/examples/two_models_two_tpus_threaded.cc)
add_executable(utils_test ${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/partitioner/utils_test.cc)
add_executable(partition_with_profiling ${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/partitioner/partition_with_profiling.cc)
add_executable(partition_with_num_ops ${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/partitioner/partition_with_num_ops.cc)
add_executable(parameter_count_based_partitioner_test ${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/partitioner/parameter_count_based_partitioner_test.cc)
add_executable(multiple_tpus_performance_analysis ${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/multiple_tpus_performance_analysis.cc)
add_executable(model_pipelining_performance_analysis ${CMAKE_CURRENT_SOURCE_DIR}/coral/tools/model_pipelining_performance_analysis.cc)
