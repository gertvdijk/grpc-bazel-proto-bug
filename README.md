# Demo bug with gRPC's `bazel/python_rules.bzl` dependencies cross-workspace

THIS IS THE BRANCH WITH THE PROPOSED FIX AS PATCH APPLIED IN `WORKSPACE`.

In this minimal example, I use of Google's `grpc.io`'s Bazel rules for Python for the purpose of demonstrating
[a bug][bug] when these targets are used in another workspace.

* `foo_lib/` is supposed to be a library, providing a proto file with a service, but is a separate project. In this
  example it's inside the same git repository, but Bazel-wise it's a different workspace.
* `foo_app/` is a Python project, and depends on `foo_lib`'s `py_proto_library` & `py_grpc_library` targets.

Run

    $ cd foo_app
    $ bazelisk build //:foo_app

Or even a much simpler case (in `foo_app/`):

    $ cd foo_app
    $ bazelisk build @myorg_foo_lib//myorg/foo:hello_py_grpc_library

The error shown is:

    ERROR: /path/to/bazel/cache/external/myorg_foo_lib/myorg/foo/BUILD.bazel:23:1: output 'external/myorg_foo_lib/myorg/foo/hello_pb2_grpc.py' was not created
    ERROR: /path/to/bazel/cache/external/myorg_foo_lib/myorg/foo/BUILD.bazel:23:1: not all outputs were created or valid

Tested with Bazel 2.0.0 and 2.1.0rc1.

With Bazel 1.2.1 (release before 2.0.0), it even fails inside the same workspace:

    $ cd foo_lib
    $ USE_BAZEL_VERSION=1.2.1 bazelisk build @myorg_foo_lib//myorg/foo:hello_py_grpc_library

I believe the error is that the logic in the declaration of the output files is different than the logic used for
invoking the protobuf compiler. The generated files appear to be in the genfiles directory, but declared in the external
directory.

More info; see [bug #21830][bug].

[bug]: https://github.com/grpc/grpc/issues/21830

# LICENSE

[Creative Commons Zero](https://creativecommons.org/publicdomain/zero/1.0/legalcode)
