load("//tensorflow/core/platform:rules_cc.bzl", "cc_library")
load("@com_github_grpc_grpc//bazel:cc_grpc_library.bzl", "cc_grpc_library")
load(
    "//tensorflow/core/platform:build_config.bzl",
    "tf_additional_all_protos",
    "tf_proto_library",
    "tf_protos_profiler_service",
)

# buildifier: disable=same-origin-load
load("//tensorflow:tensorflow.bzl", "cc_header_only_library")

# buildifier: disable=same-origin-load
load("//tensorflow:tensorflow.bzl", "get_compatible_with_cloud")

# buildifier: disable=same-origin-load
load("//tensorflow:tensorflow.bzl", "tf_grpc_cc_dependency")
load(
    "//tensorflow:tensorflow.bzl",
    "tf_cc_test",
)

package(
    default_visibility = [
        "//tensorflow:internal",
    ],
    licenses = ["notice"],  # Apache 2.0
)

exports_files(["LICENSE"])

tf_proto_library(
    name = "common_proto",
    srcs = ["common.proto"],
    cc_api_version = 2,
    protodeps = tf_additional_all_protos(),
)

tf_proto_library(
    name = "dispatcher_proto",
    srcs = ["dispatcher.proto"],
    has_services = 1,
    cc_api_version = 2,
    protodeps = tf_additional_all_protos() + [
        ":common_proto",
    ],
)

tf_proto_library(
    name = "worker_proto",
    srcs = ["worker.proto"],
    has_services = 1,
    cc_api_version = 2,
    protodeps = tf_additional_all_protos() + [
        ":common_proto",
        "//tensorflow/core/data:dataset_proto",
    ],
)

cc_library(
    name = "credentials_factory",
    srcs = ["credentials_factory.cc"],
    hdrs = ["credentials_factory.h"],
    deps = [
        "//tensorflow/core:lib",
        "@com_google_absl//absl/strings",
        tf_grpc_cc_dependency(),
    ],
)

tf_cc_test(
    name = "credentials_factory_test",
    srcs = ["credentials_factory_test.cc"],
    deps = [
        ":credentials_factory",
        "//tensorflow/core:lib",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core:testlib",
    ],
)

cc_library(
    name = "data_service",
    srcs = ["data_service.cc"],
    hdrs = [
        "data_service.h",
    ],
    deps = [
        ":credentials_factory",
        ":dispatcher_cc_grpc_proto",
        ":dispatcher_proto_cc",
        ":grpc_util",
        ":worker_cc_grpc_proto",
        ":worker_proto_cc",
        "//tensorflow/core:framework",
        "//tensorflow/core:lib",
        "//tensorflow/core:lib_internal",
        "//tensorflow/core:protos_all_cc",
        tf_grpc_cc_dependency(),
    ],
)

tf_cc_test(
    name = "data_service_test",
    srcs = ["data_service_test.cc"],
    tags = ["no_windows"],
    deps = [
        ":data_service",
        ":dispatcher_cc_grpc_proto",
        ":dispatcher_proto_cc",
        ":grpc_dispatcher_impl",
        ":grpc_util",
        ":grpc_worker_impl",
        ":local_credentials_factory",
        ":server_lib",
        ":test_cluster",
        ":test_util",
        ":worker_cc_grpc_proto",
        ":worker_proto_cc",
        "@com_google_absl//absl/strings",
        "//tensorflow/core:lib",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core/data:compression_utils",
        "//tensorflow/core/kernels/data:dataset_test_base",
        tf_grpc_cc_dependency(),
    ] + tf_protos_profiler_service(),
)

cc_library(
    name = "dataset_store",
    srcs = ["dataset_store.cc"],
    hdrs = ["dataset_store.h"],
    deps = [
        ":common_proto_cc",
        ":dispatcher_state",
        ":utils",
        "//tensorflow/core:lib",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/memory",
        "@com_google_absl//absl/strings",
    ],
)

tf_cc_test(
    name = "dataset_store_test",
    srcs = ["dataset_store_test.cc"],
    deps = [
        ":common_proto_cc",
        ":dataset_store",
        "//tensorflow/core:lib",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core:testlib",
        "@com_google_absl//absl/memory",
    ],
)

cc_grpc_library(
    name = "dispatcher_cc_grpc_proto",
    srcs = [":dispatcher_proto"],
    compatible_with = get_compatible_with_cloud(),
    generate_mocks = True,
    grpc_only = True,
    deps = [":dispatcher_proto_cc"],
)

cc_library(
    name = "dispatcher_impl",
    srcs = ["dispatcher_impl.cc"],
    hdrs = [
        "dispatcher_impl.h",
    ],
    deps = [
        ":common_proto_cc",
        ":credentials_factory",
        ":data_service",
        ":dataset_store",
        ":dispatcher_proto_cc",
        ":dispatcher_state",
        ":grpc_util",
        ":journal",
        ":worker_cc_grpc_proto",
        "//tensorflow/core:core_cpu",
        "//tensorflow/core:core_cpu_internal",
        "//tensorflow/core:framework_internal",
        "//tensorflow/core:lib",
        "//tensorflow/core:lib_internal",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core/data:standalone",
        "//tensorflow/core/kernels/data:dataset_utils",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/memory",
        tf_grpc_cc_dependency(),
    ],
)

cc_library(
    name = "dispatcher_state",
    srcs = ["dispatcher_state.cc"],
    hdrs = [
        "dispatcher_state.h",
    ],
    deps = [
        ":common_proto_cc",
        ":data_service",
        ":journal",
        ":journal_proto_cc",
        "//tensorflow/core:lib",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/memory",
    ],
)

tf_cc_test(
    name = "dispatcher_state_test",
    srcs = ["dispatcher_state_test.cc"],
    deps = [
        ":common_proto_cc",
        ":dispatcher_state",
        ":journal",
        ":journal_proto_cc",
        "//tensorflow/core:lib",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core:testlib",
        "//tensorflow/core/platform:errors",
    ],
)

cc_library(
    name = "grpc_dispatcher_impl",
    srcs = ["grpc_dispatcher_impl.cc"],
    hdrs = ["grpc_dispatcher_impl.h"],
    deps = [
        ":dispatcher_cc_grpc_proto",
        ":dispatcher_impl",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core/distributed_runtime/rpc:grpc_util",
        tf_grpc_cc_dependency(),
    ],
)

cc_library(
    name = "grpc_util",
    srcs = ["grpc_util.cc"],
    hdrs = [
        "grpc_util.h",
    ],
    deps = [
        "//tensorflow/core:lib",
        "//tensorflow/core/distributed_runtime/rpc:grpc_util",
        tf_grpc_cc_dependency(),
    ],
)

tf_cc_test(
    name = "grpc_util_test",
    srcs = ["grpc_util_test.cc"],
    deps = [
        ":grpc_util",
        "//tensorflow/core:lib",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core:testlib",
    ],
)

cc_library(
    name = "grpc_worker_impl",
    srcs = ["grpc_worker_impl.cc"],
    hdrs = ["grpc_worker_impl.h"],
    deps = [
        ":worker_cc_grpc_proto",
        ":worker_impl",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core/distributed_runtime/rpc:grpc_util",
        tf_grpc_cc_dependency(),
    ],
)

cc_library(
    name = "journal",
    srcs = ["journal.cc"],
    hdrs = ["journal.h"],
    deps = [
        ":journal_proto_cc",
        "//tensorflow/core:lib",
        "//tensorflow/core/platform:regexp",
        "@com_google_absl//absl/memory",
        "@com_google_absl//absl/strings",
    ],
)

tf_proto_library(
    name = "journal_proto",
    srcs = ["journal.proto"],
    cc_api_version = 2,
    protodeps = [
        ":common_proto",
    ],
)

tf_cc_test(
    name = "journal_test",
    srcs = ["journal_test.cc"],
    deps = [
        ":common_proto_cc",
        ":journal",
        ":journal_proto_cc",
        "//tensorflow/core:lib",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core:testlib",
        "@com_google_absl//absl/memory",
    ],
)

# Link this target to enable LOCAL credentials for the dataset service.
cc_library(
    name = "local_credentials_factory",
    srcs = ["local_credentials_factory.cc"],
    deps = [
        ":credentials_factory",
        tf_grpc_cc_dependency(),
    ],
    alwayslink = 1,
)

cc_library(
    name = "server_lib",
    srcs = ["server_lib.cc"],
    hdrs = ["server_lib.h"],
    linkstatic = True,
    visibility = [
        "//visibility:public",
    ],
    deps = [
        ":credentials_factory",
        ":grpc_dispatcher_impl",
        ":grpc_util",
        ":grpc_worker_impl",
        "//tensorflow/core:lib",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core:tensorflow",
        "//tensorflow/core/profiler/rpc:profiler_service_impl",
        tf_grpc_cc_dependency(),
    ],
    alwayslink = 1,
)

# This needs to be cc_header_only_library - tf_pybind_cc_library_wrapper
# does not pull in the server_lib.h header.
cc_header_only_library(
    name = "server_lib_headers_lib",
    features = ["-parse_headers"],
    deps = [
        ":server_lib",
    ],
)

cc_library(
    name = "split_provider",
    srcs = ["split_provider.cc"],
    hdrs = [
        "split_provider.h",
    ],
    deps = [
        ":data_service",
        ":grpc_util",
        "//tensorflow/core:framework",
        "//tensorflow/core:lib",
    ],
)

cc_library(
    name = "test_cluster",
    testonly = True,
    srcs = ["test_cluster.cc"],
    hdrs = ["test_cluster.h"],
    deps = [
        ":server_lib",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core/platform:errors",
        "@com_google_absl//absl/strings",
    ],
)

cc_library(
    name = "test_util",
    testonly = True,
    srcs = ["test_util.cc"],
    hdrs = [
        "test_util.h",
    ],
    data = glob(["testdata/*.pbtxt"]),
    deps = [
        "//tensorflow/core:framework",
        "//tensorflow/core:lib",
        "//tensorflow/core/framework:protos_all_cc",
        "//tensorflow/core/kernels/data:dataset_test_base",
    ],
)

tf_cc_test(
    name = "test_util_test",
    srcs = ["test_util_test.cc"],
    deps = [
        ":test_util",
        "//tensorflow/core:lib",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core:testlib",
        "//tensorflow/core/data:standalone",
        "//tensorflow/core/kernels/data:dataset_test_base",
    ],
)

cc_library(
    name = "utils",
    srcs = ["utils.cc"],
    hdrs = ["utils.h"],
    deps = [
        ":common_proto_cc",
        "//tensorflow/core:lib",
        "@com_google_absl//absl/memory",
        "@com_google_absl//absl/strings",
    ],
)

tf_cc_test(
    name = "utils_test",
    srcs = ["utils_test.cc"],
    deps = [
        ":common_proto_cc",
        ":utils",
        "//tensorflow/core:lib",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core:testlib",
    ],
)

cc_grpc_library(
    name = "worker_cc_grpc_proto",
    srcs = [":worker_proto"],
    compatible_with = get_compatible_with_cloud(),
    generate_mocks = True,
    grpc_only = True,
    deps = [":worker_proto_cc"],
)

cc_library(
    name = "worker_impl",
    srcs = ["worker_impl.cc"],
    hdrs = [
        "worker_impl.h",
    ],
    deps = [
        ":common_proto_cc",
        ":credentials_factory",
        ":data_service",
        ":dispatcher_cc_grpc_proto",
        ":dispatcher_proto_cc",
        ":grpc_util",
        ":split_provider",
        ":utils",
        ":worker_proto_cc",
        "//tensorflow/c:c_api_internal",
        "//tensorflow/c:tf_status_helper",
        "//tensorflow/core:core_cpu",
        "//tensorflow/core:core_cpu_internal",
        "//tensorflow/core:framework_internal",
        "//tensorflow/core:lib",
        "//tensorflow/core:lib_internal",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core/data:dataset_proto_cc",
        "//tensorflow/core/data:standalone",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/memory",
        tf_grpc_cc_dependency(),
    ],
)
