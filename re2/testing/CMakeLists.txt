set(TESTING_SOURCES
backtrack.cc
dump.cc
exhaustive_tester.cc
null_walker.cc
regexp_generator.cc
string_generator.cc
tester.cc
../../util/pcre.cc
)

add_library(testing STATIC ${TESTING_SOURCES})
target_include_directories(testing PRIVATE ../../)
target_link_libraries(testing PRIVATE re2::re2)

if(UNIX)
  find_package(Threads)
  target_link_libraries(testing PRIVATE Threads::Threads)
endif()

if(USEPCRE)
  target_compile_definitions(testing PRIVATE -DUSEPCRE)
  target_link_libraries(testing PRIVATE pcre)
endif()


set(TEST_TARGETS
charclass_test
compile_test
filtered_re2_test
mimics_pcre_test
parse_test
possible_match_test
re2_test
re2_arg_test
regexp_test
required_prefix_test
search_test
set_test
simplify_test
string_generator_test

dfa_test
exhaustive1_test
exhaustive2_test
exhaustive3_test
exhaustive_test
random_test
)

set(BENCHMARK_TARGETS
regexp_benchmark
)

foreach(target ${TEST_TARGETS})
add_executable(${target} ${target}.cc ../../util/test.cc)
target_link_libraries(${target} testing re2::re2)
add_test(NAME ${target} COMMAND ${target})
endforeach(target)

foreach(target ${BENCHMARK_TARGETS})
add_executable(${target} ${target}.cc ../../util/benchmark.cc)
target_link_libraries(${target} testing re2::re2)
endforeach(target)