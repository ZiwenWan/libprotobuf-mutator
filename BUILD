package(default_visibility = ["//visibility:public"])
cc_library(
    name = "libprotobuf_mutator_lib",
    srcs = glob(["src/**/*.cc","src/**/*.h","port/protobuf.h"],exclude=["**/*_test.cc"]),
    hdrs = ["src/libfuzzer/libfuzzer_macro.h"],
    deps = ["@com_google_protobuf//:protobuf"],
)