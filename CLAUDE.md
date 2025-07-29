# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build System and Development Commands

JACK2 uses the WAF build system (Python-based):

```bash
# Configure the build (one-time setup)
python3 ./waf configure

# Build the project
python3 ./waf build

# Clean build artifacts
python3 ./waf clean

# Install to system (requires configure first)
python3 ./waf install

# Run tests (if built with --tests)
python3 ./waf build --tests
./build/tests/testAtomic
./build/tests/testMutex
# Other test binaries in build/tests/
```

### Build Configuration Options

Key configure options include:
- `--debug` - Build with debug symbols
- `--dbus` - Enable D-Bus JACK (jackdbus)
- `--alsa` - Enable ALSA driver (Linux)
- `--tests` - Build test suite
- `--mixed` - 32/64-bit mixed mode
- `--profile` - Engine profiling support

Example: `python3 ./waf configure --debug --alsa --tests`

## Architecture Overview

JACK2 is a low-latency audio server with a sophisticated real-time architecture:

### Core Components

- **JackServer**: Central orchestrator managing the entire audio system
- **JackEngine**: Heart of audio processing pipeline, manages client lifecycle and execution graph
- **JackGraphManager**: Manages audio connection graph using lock-free techniques
- **JackConnectionManager**: Handles port-to-port connections and matrices
- **JackClient**: Base for external/internal clients with callback management
- **JackDriver**: Audio driver abstraction (ALSA, CoreAudio, PortAudio, etc.)

### Key Directories

- `common/` - Platform-independent core logic, API headers in `common/jack/`
- `linux/` - ALSA, FFADO (FireWire), MIDI drivers
- `macosx/` - CoreAudio, CoreMIDI, Mach threading
- `windows/` - PortAudio, WinMME, Windows threading
- `posix/` - POSIX threading, sockets, shared memory
- `tests/` - Test suite and performance benchmarks
- `dbus/` - D-Bus integration for jackdbus

### Audio Processing Flow

1. **Driver Callback**: OS audio system triggers driver callback
2. **Engine Process**: Engine coordinates graph execution
3. **Client Activation**: Clients process audio based on connection topology
4. **Buffer Management**: Shared memory buffers route audio between clients

### JACK2 vs JACK1 Innovations

- SMP-aware design for parallel client execution
- Lock-free graph access (glitch-free connections)
- Asynchronous activation mode (better fault tolerance)
- C++ modular architecture

## Development Guidelines

### Code Location by Function

- Audio drivers: Platform-specific directories (`linux/alsa/`, `macosx/coreaudio/`)
- Client management: `common/JackClient*`
- Graph/connection logic: `common/JackGraphManager*`, `common/JackConnectionManager*`
- Server core: `common/JackServer*`, `common/JackEngine*`
- Inter-process communication: Platform-specific `*Channel*` classes
- Public API: Headers in `common/jack/`

### Testing

Individual test binaries are built to `build/tests/`:
- `testAtomic` - Atomic operations testing
- `testMutex` - Mutex implementation testing
- `testSem` - Semaphore testing
- `testThread` - Threading primitives
- `jack_property_test.sh` - Property API testing

### Platform Support

JACK2 supports Linux, macOS, Windows, FreeBSD, and Android. Platform-specific code is isolated in respective directories with common interfaces defined in `common/`.