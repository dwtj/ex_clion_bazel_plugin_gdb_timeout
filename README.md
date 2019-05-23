## Cannot Debug in CLion on Ubuntu: GDB gives timeout error

A CLion/GDB timeout error occurs whenever I try to debug a `cc_test` target in
a modestly-sized Bazel project with manually set `--cxxopt` in the `.bazelrc`
file. Therefore, I can no longer use the CLion debugger on Bazel `cc_test`
targets in one of my projects.

Here is what the timeout error looks like:





### Testing Configurations

I've confirmed that this problem exists in at least three configurations:

- Ubuntu 19.04
- CLion v2019.1.3
- Bazel v0.25.2
- `bazelbuild/intellij` plugin:
    - v2019.05.01.0.3
    - v2019.05.13.0.2
    - commit `eaeaf473`

(I've also seen this problem with other recent versions of the plugin, but I
haven't recorded exactly which. I describe an older working configuration below
under "Successful Workaround", so the problem probably started sometime since
January.)


### Test Project

I've made a simple Bazel project, [dwtj/ex_clion_bazel_plugin_gdb_timeout](https://github.com/dwtj/ex_clion_bazel_plugin_gdb_timeout),
to help demonstrate the problem.

This project just contains:

- A `.bazelrc` file with explicitly set `--cxxopt='-std=c++14'`
- A trivial `cc_test` target
- A protobuf target on which the `cc_test` depends.

(The protobuf dependency is just here to extends compile time long enough to
trigger the timeout. If you can't reproduce the GDB timeout on your system,
try adding some more work for the compiler to do.)

On my system, attempts to debug the `hello` target always time out 10-15
seconds after the "Debug" toolbar opens. The error message always looks like
this:


### Steps to Reproduce

1. Import test project into CLion.
2. Follow the two workarounds which I described in issue #525:
    - select a custom GDB executable (e.g. `/usr/bin/gdb`)
    - run `chmod u+x gdbserver`
3. Create a CLion Run/Debug Configuration from the "Bazel Command" template
with target expression `//:hello` and Bazel command `test`.
4. Debug this configuration.

On my system, I then observe the following:

1. "Bazel Console" starts "Building debug binary". (See attached log file.)
2. Once compilation completes, the "Debug" Toolbar becomes visible, and its
"Console" tab starts "Running debug binary". However, this console does not
just run/debug the binary; *it appears to perform the compilation all over
again*. (See attached log file.)
3. About 10-15 seconds after this 


### Unsuccessful Workaround 1: Modify CLion Registry to Lengthen Debugger Timeout

After some googling, I found this JetBrains article: [Adjusting GDB start-up
time out value](https://www.jetbrains.com/help/clion/configuring-debugger-options.html#daddf152).
However, making this value either small or large made no difference: the
red timeout error happened after about fifteen seconds either way.


### Unsuccessful Workaround 2: Lengthen the `--test_timeout` Argument of the `bazel` invocation.

I tried modifying the plugin slightly so that the `bazel` invocations being
run have a much longer `--test_timeout`. Increasing it from 3600 to 36000000
did not prevent the GDB timeout error.


### Successful Workaround: Revert To Old Versions

I've reverted to older releases, specifically, CLion v2018.3.4, Bazel v0.22.0,
and `bazelbuild/intellij` plugin v2019.01.14.0.5.
