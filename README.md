# libzmq Native Build Project

[![Build Status](https://github.com/ulala-x/libzmq-native/actions/workflows/build.yml/badge.svg)](https://github.com/ulala-x/libzmq-native/actions/workflows/build.yml)
[![GitHub Release](https://img.shields.io/github/v/release/ulala-x/libzmq-native?include_prereleases&sort=semver)](https://github.com/ulala-x/libzmq-native/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![libzmq](https://img.shields.io/badge/libzmq-v4.3.5-blue.svg)](https://github.com/zeromq/libzmq)
[![libsodium](https://img.shields.io/badge/libsodium-v1.0.20-blue.svg)](https://github.com/jedisct1/libsodium)

[![Windows](https://img.shields.io/badge/Windows-x64_|_ARM64-0078D6?logo=windows&logoColor=white)](https://github.com/ulala-x/libzmq-native/releases)
[![Linux](https://img.shields.io/badge/Linux-x64_|_ARM64-FCC624?logo=linux&logoColor=black)](https://github.com/ulala-x/libzmq-native/releases)
[![macOS](https://img.shields.io/badge/macOS-Intel_|_Apple_Silicon-000000?logo=apple&logoColor=white)](https://github.com/ulala-x/libzmq-native/releases)

---

Pre-built native binaries for [libzmq](https://github.com/zeromq/libzmq) (ZeroMQ) with [libsodium](https://github.com/jedisct1/libsodium) statically linked for CURVE encryption support.

## Features

- **Cross-platform**: Windows, Linux, macOS (Intel & Apple Silicon)
- **CURVE Encryption**: Full security support with statically linked libsodium
- **Single File Distribution**: No external dependencies required
- **Production Ready**: Optimized release builds
- **CI/CD Automated**: Consistent builds via GitHub Actions

## Download

Download pre-built binaries from [GitHub Releases](https://github.com/ulala-x/libzmq-native/releases).

| Platform | Architecture | File |
|----------|--------------|------|
| Windows | x64 | `libzmq-windows-x64.zip` |
| Windows | ARM64 | `libzmq-windows-arm64.zip` |
| Linux | x64 | `libzmq-linux-x64.zip` |
| Linux | ARM64 | `libzmq-linux-arm64.zip` |
| macOS | Intel x64 | `libzmq-macos-x64.zip` |
| macOS | Apple Silicon | `libzmq-macos-arm64.zip` |

## Library Versions

| Library | Version |
|---------|---------|
| libzmq | 4.3.5 |
| libsodium | 1.0.20 |

## Output Files

```
dist/
├── windows-x64/
│   ├── libzmq.dll
│   └── libzmq.lib
├── windows-arm64/
│   ├── libzmq.dll
│   └── libzmq.lib
├── linux-x64/
│   └── libzmq.so
├── linux-arm64/
│   └── libzmq.so
├── macos-x86_64/
│   └── libzmq.dylib
└── macos-arm64/
    └── libzmq.dylib
```

## Usage Examples

### C++ (CMake)

**CMakeLists.txt**
```cmake
cmake_minimum_required(VERSION 3.15)
project(zmq_example)

set(CMAKE_CXX_STANDARD 17)

# Set libzmq path
set(LIBZMQ_DIR "${CMAKE_SOURCE_DIR}/libs/libzmq")

# Find library based on platform
if(WIN32)
    set(LIBZMQ_LIB "${LIBZMQ_DIR}/windows-x64/libzmq.lib")
    set(LIBZMQ_DLL "${LIBZMQ_DIR}/windows-x64/libzmq-v143-mt-4_3_5.dll")
elseif(APPLE)
    if(CMAKE_SYSTEM_PROCESSOR MATCHES "arm64")
        set(LIBZMQ_LIB "${LIBZMQ_DIR}/macos-arm64/libzmq.dylib")
    else()
        set(LIBZMQ_LIB "${LIBZMQ_DIR}/macos-x86_64/libzmq.dylib")
    endif()
else()
    set(LIBZMQ_LIB "${LIBZMQ_DIR}/linux-x64/libzmq.so")
endif()

add_executable(zmq_example main.cpp)
target_link_libraries(zmq_example ${LIBZMQ_LIB})

# Windows: Copy DLL to output directory
if(WIN32)
    add_custom_command(TARGET zmq_example POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "${LIBZMQ_DLL}" "$<TARGET_FILE_DIR:zmq_example>")
endif()
```

**main.cpp**
```cpp
#include <zmq.h>
#include <iostream>
#include <string>

int main() {
    // Initialize context
    void* context = zmq_ctx_new();

    // Create REP socket
    void* responder = zmq_socket(context, ZMQ_REP);
    zmq_bind(responder, "tcp://*:5555");

    std::cout << "Server listening on port 5555..." << std::endl;

    while (true) {
        char buffer[256];
        int size = zmq_recv(responder, buffer, 255, 0);
        buffer[size] = '\0';

        std::cout << "Received: " << buffer << std::endl;

        // Send reply
        std::string reply = "World";
        zmq_send(responder, reply.c_str(), reply.size(), 0);
    }

    zmq_close(responder);
    zmq_ctx_destroy(context);
    return 0;
}
```

### .NET (C#)

**Using NetZeroMQ (Native libzmq binding)**
```csharp
// Install: dotnet add package NetZeroMQ
using NetZeroMQ;

// Server
using var context = new ZContext();
using var responder = new ZSocket(context, ZSocketType.REP);
responder.Bind("tcp://*:5555");

while (true)
{
    using var request = responder.ReceiveFrame();
    Console.WriteLine($"Received: {request.ReadString()}");

    using var reply = new ZFrame("World");
    responder.Send(reply);
}
```

```csharp
// Client
using NetZeroMQ;

using var context = new ZContext();
using var requester = new ZSocket(context, ZSocketType.REQ);
requester.Connect("tcp://localhost:5555");

using var request = new ZFrame("Hello");
requester.Send(request);

using var reply = requester.ReceiveFrame();
Console.WriteLine($"Received: {reply.ReadString()}");
```

**Using P/Invoke with native libzmq**
```csharp
using System;
using System.Runtime.InteropServices;

public static class LibZmq
{
    private const string LibraryName = "libzmq";

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    public static extern IntPtr zmq_ctx_new();

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    public static extern IntPtr zmq_socket(IntPtr context, int type);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    public static extern int zmq_bind(IntPtr socket, string endpoint);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    public static extern int zmq_connect(IntPtr socket, string endpoint);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    public static extern int zmq_send(IntPtr socket, byte[] buf, int len, int flags);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    public static extern int zmq_recv(IntPtr socket, byte[] buf, int len, int flags);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    public static extern int zmq_close(IntPtr socket);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    public static extern int zmq_ctx_destroy(IntPtr context);

    // Socket types
    public const int ZMQ_REQ = 3;
    public const int ZMQ_REP = 4;
    public const int ZMQ_PUB = 1;
    public const int ZMQ_SUB = 2;
}

// Usage
class Program
{
    static void Main()
    {
        IntPtr context = LibZmq.zmq_ctx_new();
        IntPtr socket = LibZmq.zmq_socket(context, LibZmq.ZMQ_REQ);

        LibZmq.zmq_connect(socket, "tcp://localhost:5555");

        byte[] request = System.Text.Encoding.UTF8.GetBytes("Hello");
        LibZmq.zmq_send(socket, request, request.Length, 0);

        byte[] reply = new byte[256];
        int size = LibZmq.zmq_recv(socket, reply, reply.Length, 0);
        Console.WriteLine($"Received: {System.Text.Encoding.UTF8.GetString(reply, 0, size)}");

        LibZmq.zmq_close(socket);
        LibZmq.zmq_ctx_destroy(context);
    }
}
```

**.NET Project Configuration (csproj)**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>

  <!-- Copy native library based on runtime -->
  <ItemGroup>
    <None Include="libs\windows-x64\libzmq-v143-mt-4_3_5.dll"
          CopyToOutputDirectory="PreserveNewest"
          Condition="$([MSBuild]::IsOSPlatform('Windows'))" />
    <None Include="libs\linux-x64\libzmq.so"
          CopyToOutputDirectory="PreserveNewest"
          Condition="$([MSBuild]::IsOSPlatform('Linux'))" />
    <None Include="libs\macos-x64\libzmq.dylib"
          CopyToOutputDirectory="PreserveNewest"
          Condition="$([MSBuild]::IsOSPlatform('OSX'))" />
  </ItemGroup>
</Project>
```

### Java (FFM API - JDK 22+)

**Using Foreign Function & Memory API (Project Panama)**
```java
// Requires: JDK 22+ with --enable-native-access=ALL-UNNAMED

import java.lang.foreign.*;
import java.lang.invoke.MethodHandle;

public class ZmqExample {
    private static final Linker LINKER = Linker.nativeLinker();
    private static final SymbolLookup ZMQ;

    static {
        System.load("/path/to/dist/linux-x64/libzmq.so");
        ZMQ = SymbolLookup.loaderLookup();
    }

    private static final MethodHandle zmq_ctx_new = LINKER.downcallHandle(
        ZMQ.find("zmq_ctx_new").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.ADDRESS)
    );
    private static final MethodHandle zmq_socket = LINKER.downcallHandle(
        ZMQ.find("zmq_socket").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.ADDRESS, ValueLayout.ADDRESS, ValueLayout.JAVA_INT)
    );
    private static final MethodHandle zmq_connect = LINKER.downcallHandle(
        ZMQ.find("zmq_connect").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.JAVA_INT, ValueLayout.ADDRESS, ValueLayout.ADDRESS)
    );
    private static final MethodHandle zmq_send = LINKER.downcallHandle(
        ZMQ.find("zmq_send").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.JAVA_INT, ValueLayout.ADDRESS, ValueLayout.ADDRESS,
                              ValueLayout.JAVA_LONG, ValueLayout.JAVA_INT)
    );
    private static final MethodHandle zmq_recv = LINKER.downcallHandle(
        ZMQ.find("zmq_recv").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.JAVA_INT, ValueLayout.ADDRESS, ValueLayout.ADDRESS,
                              ValueLayout.JAVA_LONG, ValueLayout.JAVA_INT)
    );
    private static final MethodHandle zmq_close = LINKER.downcallHandle(
        ZMQ.find("zmq_close").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.JAVA_INT, ValueLayout.ADDRESS)
    );
    private static final MethodHandle zmq_ctx_destroy = LINKER.downcallHandle(
        ZMQ.find("zmq_ctx_destroy").orElseThrow(),
        FunctionDescriptor.of(ValueLayout.JAVA_INT, ValueLayout.ADDRESS)
    );

    private static final int ZMQ_REQ = 3;

    public static void main(String[] args) throws Throwable {
        try (var arena = Arena.ofConfined()) {
            MemorySegment ctx = (MemorySegment) zmq_ctx_new.invokeExact();
            MemorySegment socket = (MemorySegment) zmq_socket.invokeExact(ctx, ZMQ_REQ);

            MemorySegment endpoint = arena.allocateFrom("tcp://localhost:5555");
            zmq_connect.invokeExact(socket, endpoint);

            byte[] msgBytes = "Hello".getBytes();
            MemorySegment msg = arena.allocateFrom(ValueLayout.JAVA_BYTE, msgBytes);
            zmq_send.invokeExact(socket, msg, (long) msgBytes.length, 0);

            MemorySegment buffer = arena.allocate(256);
            int size = (int) zmq_recv.invokeExact(socket, buffer, 256L, 0);
            System.out.println("Received: " + new String(buffer.toArray(ValueLayout.JAVA_BYTE), 0, size));

            zmq_close.invokeExact(socket);
            zmq_ctx_destroy.invokeExact(ctx);
        }
    }
}
// Run: java --enable-native-access=ALL-UNNAMED ZmqExample.java
```

## CURVE Encryption Example

```cpp
#include <zmq.h>
#include <cstring>

int main() {
    void* context = zmq_ctx_new();

    // Generate key pair
    char public_key[41];
    char secret_key[41];
    zmq_curve_keypair(public_key, secret_key);

    // Server setup
    void* server = zmq_socket(context, ZMQ_REP);
    zmq_setsockopt(server, ZMQ_CURVE_SERVER, &(int){1}, sizeof(int));
    zmq_setsockopt(server, ZMQ_CURVE_SECRETKEY, secret_key, 40);
    zmq_bind(server, "tcp://*:5556");

    // Client setup
    char client_public[41], client_secret[41];
    zmq_curve_keypair(client_public, client_secret);

    void* client = zmq_socket(context, ZMQ_REQ);
    zmq_setsockopt(client, ZMQ_CURVE_SERVERKEY, public_key, 40);
    zmq_setsockopt(client, ZMQ_CURVE_PUBLICKEY, client_public, 40);
    zmq_setsockopt(client, ZMQ_CURVE_SECRETKEY, client_secret, 40);
    zmq_connect(client, "tcp://localhost:5556");

    // Encrypted communication...

    zmq_close(client);
    zmq_close(server);
    zmq_ctx_destroy(context);
    return 0;
}
```

## Building from Source

### Prerequisites

| Platform | Requirements |
|----------|--------------|
| Windows | Visual Studio 2022, CMake 3.15+, Git, PowerShell 5.0+ |
| Linux | GCC 7.0+, CMake 3.15+, make, autotools, pkg-config |
| macOS | Xcode Command Line Tools, CMake 3.15+ |

### Build Commands

**Windows**
```powershell
.\build-scripts\windows\build.ps1
```

**Linux**
```bash
./build-scripts/linux/build.sh
```

**macOS**
```bash
# Intel
./build-scripts/macos/build.sh x86_64

# Apple Silicon
./build-scripts/macos/build.sh arm64
```

## Project Structure

```
libzmq-native/
├── .github/workflows/     # CI/CD workflows
├── build-scripts/
│   ├── windows/          # Windows build (PowerShell)
│   ├── linux/            # Linux build (Bash)
│   └── macos/            # macOS build (Bash)
├── tests/                # Integration tests
├── dist/                 # Build output (generated)
├── VERSION               # Version configuration
└── README.md
```

## Verification

Each build automatically verifies:
- libsodium is statically linked
- CURVE encryption symbols are available
- Correct binary architecture
- Minimal external dependencies

```bash
# Linux - Check dependencies
ldd dist/linux-x64/libzmq.so

# macOS - Check dependencies
otool -L dist/macos-arm64/libzmq.dylib

# Windows - Check dependencies (requires Visual Studio)
dumpbin /DEPENDENTS dist\windows-x64\libzmq.dll
```

## Troubleshooting

### Windows: DLL not found
Ensure the versioned DLL (`libzmq-v143-mt-4_3_5.dll`) is in the same directory as your executable or in the system PATH.

### Linux: Library not found at runtime
```bash
export LD_LIBRARY_PATH=/path/to/libzmq:$LD_LIBRARY_PATH
```

### macOS: Library not found at runtime
```bash
export DYLD_LIBRARY_PATH=/path/to/libzmq:$DYLD_LIBRARY_PATH
```

## License

This build project is licensed under [MIT License](LICENSE).

The underlying libraries have their own licenses:
- **libzmq**: [MPLv2](https://github.com/zeromq/libzmq/blob/master/LICENSE) / [LGPL v3](https://www.gnu.org/licenses/lgpl-3.0.html)
- **libsodium**: [ISC License](https://github.com/jedisct1/libsodium/blob/master/LICENSE)

## Resources

- [ZeroMQ Guide](https://zguide.zeromq.org/)
- [libzmq API Reference](http://api.zeromq.org/)
- [libsodium Documentation](https://doc.libsodium.org/)
- [NetZeroMQ (.NET Native Binding)](https://github.com/ulala-x/netzeromq)

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

**Maintained by [ulala-x](https://github.com/ulala-x)**
