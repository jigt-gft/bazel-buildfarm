[![Build status](https://badge.buildkite.com/45f4fd4c0cfb95f7705156a4119641c6d5d6c310452d6e65a4.svg)](https://buildkite.com/bazel/buildfarm-postsubmit)

# Bazel Buildfarm

This repository hosts the [Bazel](https://bazel.build) remote caching and execution system.

This project is just getting started; background information on the status of caching and remote execution in bazel can be
found in the [bazel documentation](https://github.com/bazelbuild/bazel/blob/master/src/main/java/com/google/devtools/build/lib/remote/README.md#remote-caching-using-the-grpc-protocol).

Read the [meeting notes](https://docs.google.com/document/d/1EtQMTn-7sKFMTxIMlb0oDGpvGCMAuzphVcfx58GWuEM/edit).
Get involved by joining the discussion on the [dedicated mailing list](https://groups.google.com/forum/#!forum/bazel-buildfarm).

## Usage

In general do not execute server binaries with bazel run, since bazel does not support running multiple targets.

All commandline options override corresponding config settings.

### Bazel Buildfarm Server

Run via

    bazel run //src/main/java/build/buildfarm:buildfarm-server <configfile> [-p PORT] [--port PORT]

- **`configfile`** has to be in (undocumented) Protocol Buffer text format.

  For an example, see the [examples](examples) directory, which contains the working example [examples/worker.config.example](config).
  For format details see [here](https://stackoverflow.com/questions/18873924/what-does-the-protobuf-text-format-look-like). Protocol Buffer structure at [src/main/protobuf/build/buildfarm/v1test/buildfarm.proto](src/main/protobuf/build/buildfarm/v1test/buildfarm.proto)

- **`PORT`** to expose service endpoints on

### Bazel Buildfarm Worker

Run via

    bazel run //src/main/java/build/buildfarm:buildfarm-operationqueue-worker <configfile> [--root ROOT] [--cas_cache_directory CAS_CACHE_DIRECTORY]

- **`configfile`** has to be in (undocumented) Protocol Buffer text format.

  For an example, see the [examples](examples) directory, which contains the working example [examples/server.config.example](config).
  For format details see [here](https://stackoverflow.com/questions/18873924/what-does-the-protobuf-text-format-look-like). Protocol Buffer structure at [src/main/protobuf/build/buildfarm/v1test/buildfarm.proto](src/main/protobuf/build/buildfarm/v1test/buildfarm.proto)

- **`ROOT`** base directory path for all work being performed.

- **`CAS_CACHE_DIRECTORY`** is (absolute or relative) directory path to cached files from CAS.

### Bazel Itself

To have bazel use the bazel buildfarm configured using the example configs provided in the [examples](examples) directory, you could configure your
`.bazelrc` as follows:

```
$ cat .bazelrc
build --spawn_strategy=remote --genrule_strategy=remote --strategy=Javac=remote --strategy=Closure=remote --remote_executor=localhost:8980
```

Then run your build as you would normally do.

### Debugging

Buildfarm uses [Java's Logging framework](https://docs.oracle.com/javase/10/core/java-logging-overview.htm) and outputs all routine behavior to the NICE [Level](https://docs.oracle.com/javase/8/docs/api/java/util/logging/Level.html).

You can use typical Java logging configuration to filter these results and observe the flow of executions through your running services.
An example `logging.properties` file has been provided at [examples/debug.logging.properties](examples/debug.logging.properties) for use as follows:

    bazel-bin/src/main/java/build/buildfarm/buildfarm-server --jvm_flag=-Djava.util.logging.config.file=examples/debug.logging.properties ...

and

    bazel run //src/main/java/build/buildfarm/buildfarm-operationqueue-worker --jvm_flag=-Djava.util.logging.config.file=examples/debug.logging.properties ...

To attach a remote debugger, run the executable with the `--debug=<PORT>` flag. For example:

    bazel run src/main/java/build/buildfarm/buildfarm-server --debug=5005 \
        $PWD/config/server.config

## Developer Information

### Setting up intelliJ

1. Follow the instructions in https://github.com/bazelbuild/intellij to install the bazel plugin for intelliJ
1. Import the project using `ij.bazelproject`

### Third-party Dependencies

Most third-party dependencies (e.g. protobuf, gRPC, ...) are managed automatically via
[bazel-deps](https://github.com/johnynek/bazel-deps). After changing the `dependencies.yaml` file,
just run this to regenerate the 3rdparty folder:

```bash
git clone https://github.com/johnynek/bazel-deps.git ../bazel-deps
cd ../bazel-deps
bazel build //src/scala/com/github/johnynek/bazel_deps:parseproject_deploy.jar
cd ../bazel-buildfarm
../bazel-deps/gen_maven_deps.sh generate -r `pwd` -s 3rdparty/workspace.bzl -d dependencies.yaml
```

Things that aren't supported by bazel-deps are being imported as manually managed remote repos via
the `WORKSPACE` file.

# Personal Notes JIGT

## Local Execution RepoByTech
```
bazel@ubuntu1604:/src/RepoByTech$ bazel build workspace
Starting local Bazel server and connecting to it...
INFO: Writing tracer profile to '/home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/command.profile.gz'
DEBUG: Rule 'io_bazel_rules_k8s' indicated that a canonical reproducible form can be obtained by modifying arguments shallow_since = "1563971973 -0400"
DEBUG: Call stack for the definition of repository 'io_bazel_rules_k8s' which is a git_repository (rule definition at /home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/external/bazel_tools/tools/build_defs/repo/git.bzl:195:18):
 - /src/RepoByTech/WORKSPACE:123:1
DEBUG: Rule 'build_bazel_rules_apple' indicated that a canonical reproducible form can be obtained by modifying arguments commit = "6c9fcae7a3597aabd43f28be89466afe0eab18de", shallow_since = "1565379803 -0700" and dropping ["tag"]
DEBUG: Call stack for the definition of repository 'build_bazel_rules_apple' which is a git_repository (rule definition at /home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/external/bazel_tools/tools/build_defs/repo/git.bzl:195:18):
 - /src/RepoByTech/WORKSPACE:245:1
DEBUG: /home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/external/npm_bazel_karma/browser_repositories.bzl:21:5:
    WARNING: @npm_bazel_karma//:browser_repositories.bzl is deprecated.
    replace this with @io_bazel_rules_webtesting//web/versioned:browsers-0.3.2.bzl
    and choose some specific browsers to test on (eg. chromium=True)


INFO: Analyzed target //:workspace (392 packages loaded, 20312 targets configured).
INFO: Found 1 target...
Target //:workspace up-to-date:
...
INFO: Elapsed time: 271.552s, Critical Path: 48.57s
INFO: 35 processes: 17 linux-sandbox, 18 worker.
INFO: Build completed successfully, 62 total actions
```

## Local Execution Buildfarm

```
bazel@ubuntu1604:/src/RepoByTech$ bazel build workspace
INFO: Options provided by the client:
  Inherited 'common' options: --isatty=1 --terminal_columns=112
INFO: Reading rc options for 'build' from /src/RepoByTech/.bazelrc:
  'build' options: --spawn_strategy=remote --genrule_strategy=remote --remote_executor=grpc://localhost:8980
INFO: Writing tracer profile to '/home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/command.profile.gz'
INFO: Invocation ID: eb005717-4624-438b-a1c4-4c3c0dfd035f
ERROR: Failed to query remote execution capabilities: UNAVAILABLE: io exception
bazel@ubuntu1604:/src/RepoByTech$ bazel build workspace
INFO: Writing tracer profile to '/home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/command.profile.gz'
INFO: Invocation ID: 9d794dc0-5de0-42ed-bf57-8cddcda1ca34
DEBUG: Rule 'io_bazel_rules_k8s' indicated that a canonical reproducible form can be obtained by modifying arguments shallow_since = "1563971973 -0400"
DEBUG: Call stack for the definition of repository 'io_bazel_rules_k8s' which is a git_repository (rule definition at /home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/external/bazel_tools/tools/build_defs/repo/git.bzl:195:18):
 - /src/RepoByTech/WORKSPACE:123:1
DEBUG: Rule 'build_bazel_rules_apple' indicated that a canonical reproducible form can be obtained by modifying arguments commit = "6c9fcae7a3597aabd43f28be89466afe0eab18de", shallow_since = "1565379803 -0700" and dropping ["tag"]
DEBUG: Call stack for the definition of repository 'build_bazel_rules_apple' which is a git_repository (rule definition at /home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/external/bazel_tools/tools/build_defs/repo/git.bzl:195:18):
 - /src/RepoByTech/WORKSPACE:245:1
DEBUG: /home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/external/npm_bazel_karma/browser_repositories.bzl:21:5:
    WARNING: @npm_bazel_karma//:browser_repositories.bzl is deprecated.
    replace this with @io_bazel_rules_webtesting//web/versioned:browsers-0.3.2.bzl
    and choose some specific browsers to test on (eg. chromium=True)


INFO: Analyzed target //:workspace (1 packages loaded, 565 targets configured).
INFO: Found 1 target...
Target //:workspace up-to-date:
...
INFO: Elapsed time: 365.406s, Critical Path: 58.65s
INFO: 35 processes: 35 remote.
INFO: Build completed successfully, 62 total actions
```

## Remote Execution and Remote Cache

```
bazel@bazelclient:/src/RepoByTech$ bazel build --spawn_strategy=remote --genrule_strategy=remote --remote_cache=grpc://bazelservercache.northeurope.cloudapp.azure.com:9090 --verbose_failures workspINFO: Writing tracer profile to '/home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/command.profile.gz'
INFO: Invocation ID: 378b6a5f-3104-41d2-9cd3-81f174fa70bd
DEBUG: Rule 'io_bazel_rules_k8s' indicated that a canonical reproducible form can be obtained by modifying arguments shallow_since = "1563971973 -0400"
DEBUG: Call stack for the definition of repository 'io_bazel_rules_k8s' which is a git_repository (rule definition at /home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/externalfs/repo/git.bzl:195:18):
 - /src/RepoByTech/WORKSPACE:123:1
DEBUG: Rule 'build_bazel_rules_apple' indicated that a canonical reproducible form can be obtained by modifying arguments commit = "6c9fcae7a3597aabd43f28be89466afe0eab18de", shallow_since = "15653"tag"]
DEBUG: Call stack for the definition of repository 'build_bazel_rules_apple' which is a git_repository (rule definition at /home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/extld_defs/repo/git.bzl:195:18):
 - /src/RepoByTech/WORKSPACE:245:1
DEBUG: /home/bazel/.cache/bazel/_bazel_bazel/339921194bbac111878cca1a86309115/external/npm_bazel_karma/browser_repositories.bzl:21:5:
    WARNING: @npm_bazel_karma//:browser_repositories.bzl is deprecated.
    replace this with @io_bazel_rules_webtesting//web/versioned:browsers-0.3.2.bzl
    and choose some specific browsers to test on (eg. chromium=True)


INFO: Analyzed target //:workspace (0 packages loaded, 0 targets configured).
INFO: Found 1 target...
ERROR: /src/RepoByTech/Common/Globile/Web/Components/molecules/tabbar/BUILD:8:1: SassCompiler Common/Globile/Web/Components/molecules/tabbar/tabbar.component.css failed (Exit 34). Note: Remote conn: execution failed java.io.IOException: io.grpc.StatusRuntimeException: INTERNAL: http2 exception
        at com.google.devtools.build.lib.remote.GrpcRemoteCache.getCachedActionResult(GrpcRemoteCache.java:405)
        at com.google.devtools.build.lib.remote.RemoteSpawnRunner.exec(RemoteSpawnRunner.java:196)
        at com.google.devtools.build.lib.exec.SpawnRunner.execAsync(SpawnRunner.java:225)
        at com.google.devtools.build.lib.exec.AbstractSpawnStrategy.exec(AbstractSpawnStrategy.java:123)
        at com.google.devtools.build.lib.exec.AbstractSpawnStrategy.exec(AbstractSpawnStrategy.java:88)
        at com.google.devtools.build.lib.actions.SpawnActionContext.beginExecution(SpawnActionContext.java:41)
        at com.google.devtools.build.lib.exec.ProxySpawnActionContext.beginExecution(ProxySpawnActionContext.java:56)
        at com.google.devtools.build.lib.actions.SpawnContinuation$1.execute(SpawnContinuation.java:80)
        at com.google.devtools.build.lib.analysis.actions.SpawnAction$SpawnActionContinuation.execute(SpawnAction.java:1342)
        at com.google.devtools.build.lib.analysis.actions.SpawnAction.beginExecution(SpawnAction.java:314)
        at com.google.devtools.build.lib.actions.Action.execute(Action.java:123)
        at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor$4.execute(SkyframeActionExecutor.java:851)
        at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor$ActionRunner.continueAction(SkyframeActionExecutor.java:985)
        at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor$ActionRunner.run(SkyframeActionExecutor.java:957)
        at com.google.devtools.build.lib.skyframe.ActionExecutionState.runStateMachine(ActionExecutionState.java:116)
        at com.google.devtools.build.lib.skyframe.ActionExecutionState.getResultOrDependOnFuture(ActionExecutionState.java:77)
        at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor.executeAction(SkyframeActionExecutor.java:577)
        at com.google.devtools.build.lib.skyframe.ActionExecutionFunction.checkCacheAndExecuteIfNeeded(ActionExecutionFunction.java:760)
        at com.google.devtools.build.lib.skyframe.ActionExecutionFunction.compute(ActionExecutionFunction.java:275)
        at com.google.devtools.build.skyframe.AbstractParallelEvaluator$Evaluate.run(AbstractParallelEvaluator.java:454)
        at com.google.devtools.build.lib.concurrent.AbstractQueueVisitor$WrappedRunnable.run(AbstractQueueVisitor.java:399)
        at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
        at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
        at java.base/java.lang.Thread.run(Unknown Source)
Caused by: io.grpc.StatusRuntimeException: INTERNAL: http2 exception
        at io.grpc.stub.ClientCalls.toStatusRuntimeException(ClientCalls.java:233)
        at io.grpc.stub.ClientCalls.getUnchecked(ClientCalls.java:214)
        at io.grpc.stub.ClientCalls.blockingUnaryCall(ClientCalls.java:139)
        at build.bazel.remote.execution.v2.ActionCacheGrpc$ActionCacheBlockingStub.getActionResult(ActionCacheGrpc.java:349)
        at com.google.devtools.build.lib.remote.GrpcRemoteCache.lambda$getCachedActionResult$6(GrpcRemoteCache.java:395)
        at com.google.devtools.build.lib.remote.Retrier.execute(Retrier.java:237)
        at com.google.devtools.build.lib.remote.RemoteRetrier.execute(RemoteRetrier.java:104)
        at com.google.devtools.build.lib.remote.GrpcRemoteCache.getCachedActionResult(GrpcRemoteCache.java:392)
        ... 23 more
Caused by: io.netty.handler.codec.http2.Http2Exception: First received frame was not SETTINGS. Hex dump for first 5 bytes: 485454502f
        at io.netty.handler.codec.http2.Http2Exception.connectionError(Http2Exception.java:85)
        at io.netty.handler.codec.http2.Http2ConnectionHandler$PrefaceDecoder.verifyFirstFrameIsSettings(Http2ConnectionHandler.java:350)
        at io.netty.handler.codec.http2.Http2ConnectionHandler$PrefaceDecoder.decode(Http2ConnectionHandler.java:251)
        at io.netty.handler.codec.http2.Http2ConnectionHandler.decode(Http2ConnectionHandler.java:450)
        at io.netty.handler.codec.ByteToMessageDecoder.decodeRemovalReentryProtection(ByteToMessageDecoder.java:502)
        at io.netty.handler.codec.ByteToMessageDecoder.callDecode(ByteToMessageDecoder.java:441)
        at io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:278)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:345)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:337)
        at io.netty.channel.DefaultChannelPipeline$HeadContext.channelRead(DefaultChannelPipeline.java:1408)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:345)
        at io.netty.channel.DefaultChannelPipeline.fireChannelRead(DefaultChannelPipeline.java:930)
        at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:163)
        at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:677)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:612)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:529)
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:491)
        at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:905)
        at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
        ... 1 more
Target //:workspace failed to build
ERROR: /src/RepoByTech/Common/Globile/Web/Components/molecules/tabbar/BUILD:19:1 SassCompiler Common/Globile/Web/Components/molecules/tabbar/tabbar.component.css failed (Exit 34). Note: Remote conn: execution failed java.io.IOException: io.grpc.StatusRuntimeException: INTERNAL: http2 exception
        at com.google.devtools.build.lib.remote.GrpcRemoteCache.getCachedActionResult(GrpcRemoteCache.java:405)
        at com.google.devtools.build.lib.remote.RemoteSpawnRunner.exec(RemoteSpawnRunner.java:196)
        at com.google.devtools.build.lib.exec.SpawnRunner.execAsync(SpawnRunner.java:225)
        at com.google.devtools.build.lib.exec.AbstractSpawnStrategy.exec(AbstractSpawnStrategy.java:123)
        at com.google.devtools.build.lib.exec.AbstractSpawnStrategy.exec(AbstractSpawnStrategy.java:88)
        at com.google.devtools.build.lib.actions.SpawnActionContext.beginExecution(SpawnActionContext.java:41)
        at com.google.devtools.build.lib.exec.ProxySpawnActionContext.beginExecution(ProxySpawnActionContext.java:56)
        at com.google.devtools.build.lib.actions.SpawnContinuation$1.execute(SpawnContinuation.java:80)
        at com.google.devtools.build.lib.analysis.actions.SpawnAction$SpawnActionContinuation.execute(SpawnAction.java:1342)
        at com.google.devtools.build.lib.analysis.actions.SpawnAction.beginExecution(SpawnAction.java:314)
        at com.google.devtools.build.lib.actions.Action.execute(Action.java:123)
        at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor$4.execute(SkyframeActionExecutor.java:851)
        at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor$ActionRunner.continueAction(SkyframeActionExecutor.java:985)
        at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor$ActionRunner.run(SkyframeActionExecutor.java:957)
        at com.google.devtools.build.lib.skyframe.ActionExecutionState.runStateMachine(ActionExecutionState.java:116)
        at com.google.devtools.build.lib.skyframe.ActionExecutionState.getResultOrDependOnFuture(ActionExecutionState.java:77)
        at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor.executeAction(SkyframeActionExecutor.java:577)
        at com.google.devtools.build.lib.skyframe.ActionExecutionFunction.checkCacheAndExecuteIfNeeded(ActionExecutionFunction.java:760)
        at com.google.devtools.build.lib.skyframe.ActionExecutionFunction.compute(ActionExecutionFunction.java:275)
        at com.google.devtools.build.skyframe.AbstractParallelEvaluator$Evaluate.run(AbstractParallelEvaluator.java:454)
        at com.google.devtools.build.lib.concurrent.AbstractQueueVisitor$WrappedRunnable.run(AbstractQueueVisitor.java:399)
        at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
        at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
        at java.base/java.lang.Thread.run(Unknown Source)
Caused by: io.grpc.StatusRuntimeException: INTERNAL: http2 exception
        at io.grpc.stub.ClientCalls.toStatusRuntimeException(ClientCalls.java:233)
        at io.grpc.stub.ClientCalls.getUnchecked(ClientCalls.java:214)
        at io.grpc.stub.ClientCalls.blockingUnaryCall(ClientCalls.java:139)
        at build.bazel.remote.execution.v2.ActionCacheGrpc$ActionCacheBlockingStub.getActionResult(ActionCacheGrpc.java:349)
        at com.google.devtools.build.lib.remote.GrpcRemoteCache.lambda$getCachedActionResult$6(GrpcRemoteCache.java:395)
        at com.google.devtools.build.lib.remote.Retrier.execute(Retrier.java:237)
        at com.google.devtools.build.lib.remote.RemoteRetrier.execute(RemoteRetrier.java:104)
        at com.google.devtools.build.lib.remote.GrpcRemoteCache.getCachedActionResult(GrpcRemoteCache.java:392)
        ... 23 more
Caused by: io.netty.handler.codec.http2.Http2Exception: First received frame was not SETTINGS. Hex dump for first 5 bytes: 485454502f
        at io.netty.handler.codec.http2.Http2Exception.connectionError(Http2Exception.java:85)
        at io.netty.handler.codec.http2.Http2ConnectionHandler$PrefaceDecoder.verifyFirstFrameIsSettings(Http2ConnectionHandler.java:350)
        at io.netty.handler.codec.http2.Http2ConnectionHandler$PrefaceDecoder.decode(Http2ConnectionHandler.java:251)
        at io.netty.handler.codec.http2.Http2ConnectionHandler.decode(Http2ConnectionHandler.java:450)
        at io.netty.handler.codec.ByteToMessageDecoder.decodeRemovalReentryProtection(ByteToMessageDecoder.java:502)
        at io.netty.handler.codec.ByteToMessageDecoder.callDecode(ByteToMessageDecoder.java:441)
        at io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:278)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:345)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:337)
        at io.netty.channel.DefaultChannelPipeline$HeadContext.channelRead(DefaultChannelPipeline.java:1408)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:359)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:345)
        at io.netty.channel.DefaultChannelPipeline.fireChannelRead(DefaultChannelPipeline.java:930)
        at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:163)
        at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:677)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:612)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:529)
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:491)
        at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:905)
        at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
        ... 1 more
INFO: Elapsed time: 4.261s, Critical Path: 3.23s
INFO: 0 processes.
FAILED: Build did NOT complete successfully
```