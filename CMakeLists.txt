cmake_minimum_required(VERSION 3.9)
project(android_build)

configure_file(cmake/index.html.in ${CMAKE_BINARY_DIR}/index.html)
execute_process()

