# ndnperf

[![License: GPL v3+](https://img.shields.io/badge/License-GPLv3+-blue.svg)](LICENSE)

ndnperf measures how fast a [Named Data Networking (NDN)](https://named-data.net/) server can generate,
sign and send Data packets, in the way iPerf measures an IP path: a server answers every Interest under
its prefix, and a client keeps a fixed window of Interests in flight and reports throughput, latency and
the server's processing time at regular intervals.

**ndnperf is the work of Xavier Marchal.** It was written for the study published as

> Xavier Marchal, Thibault Cholez and Olivier Festor. *Server-side performance evaluation of NDN.*
> Proceedings of the 3rd ACM Conference on Information-Centric Networking (ICN '16), pages 148–153.
> ACM, 2016. [doi:10.1145/2984356.2984364](https://doi.org/10.1145/2984356.2984364)

and is published at [github.com/Nayald/ndnperf](https://github.com/Nayald/ndnperf). Its original
description is kept in [`README.upstream.md`](README.upstream.md). This repository is a maintained fork:
it keeps the tool working with the current NDN libraries and integrates it into
[MiniNDN-WiFi](https://github.com/brunosfer/minindn-wifi), where it serves as the throughput probe of
wireless NDN experiments.

## Why this fork exists

The original C++ version was last updated in 2021 and targets ndn-cxx 0.5. It no longer builds against
ndn-cxx 0.9, the version used by NFD 24.07 and Mini-NDN 0.7. The fork changes only what the newer library
requires and leaves the tool's behaviour as it was, so that its measurements stay comparable with the
published study.

## What changed

| File | Change | Why |
|---|---|---|
| `c++/server/server.cpp` | Interest filter callbacks written as lambdas | the `bind` expressions no longer compile against ndn-cxx 0.9; the lambdas state the callback signatures (`InterestCallback`, `RegisterPrefixFailureCallback`) explicitly |
| `c++/server/server.cpp` | `Face::getIoService()` → `Face::getIoContext()` | renamed in ndn-cxx 0.9 |
| `c++/server/server.cpp` | `Data::setContent(uint8_t*, size)` → `Data::setContent(span)` | the pointer form was removed in ndn-cxx 0.9 |
| `c++/client/client.h` | `#include <fstream>` | the header no longer arrives through ndn-cxx |
| `c++/*/CMakeLists.txt` | ndn-cxx found through pkg-config, C++17, an install target | the old `find_library` call found nothing; ndn-cxx 0.9 requires C++17 |

The Java version (`java/`) is unchanged and not built.

Branches: `master` mirrors the original repository and is never changed here; the work is on `brunosfer`.

## Requirements

* ndn-cxx 0.9 or later, with its pkg-config file (`libndn-cxx.pc`)
* Boost (system, filesystem, thread, program_options), CMake 3.10 or later and a C++17 compiler
* a running NFD on the machine of the server and of the client

## Installation

With MiniNDN-WiFi, ndnperf is installed with the NDN stack, or on its own into an existing installation:

```bash
./install.sh --project=ndnperf
```

By hand, from this folder:

```bash
for p in server client; do
  cmake -S c++/$p -B c++/$p/build -DCMAKE_BUILD_TYPE=Release &&
  cmake --build c++/$p/build -j "$(nproc)" &&
  sudo cmake --install c++/$p/build
done
```

This installs `ndnperfserver` and `ndnperf` into `/usr/local/bin`.

## Usage

On the producer, and then on the consumer, with a route from the consumer's NFD to the producer's prefix:

```bash
ndnperfserver -p /throughput -c 1024 -t 1     # 1 KB of content per Data, one signing thread
ndnperf -p /throughput -w 32                  # 32 Interests in flight
```

The options of both programs are listed with `-h` and described in [`c++/server/README.md`](c++/server/README.md)
and [`c++/client/README.md`](c++/client/README.md). In an emulation, give the server an explicit `-t`: by
default it starts one signing thread per logical CPU of the host.

## Repository layout

| Path | Content |
|---|---|
| `c++/server/` | `ndnperfserver`, with the lock-free queue by Cameron Desrochers (`concurrentqueue.h`, `blockingconcurrentqueue.h`) |
| `c++/client/` | `ndnperf` |
| `java/` | the original Java version, unchanged |
| `README.upstream.md` | the original description of the tool |

## Acknowledgements

All credit for the design and the code of ndnperf goes to Xavier Marchal and to the co-authors of the
ICN '16 study. The lock-free queue is the work of Cameron Desrochers
([moodycamel::ConcurrentQueue](https://github.com/cameron314/concurrentqueue)).

## How to cite

If you use ndnperf in academic work, please cite the original study above; [`CITATION.cff`](CITATION.cff)
gives it as the preferred citation, and names this fork for the version you used.

## License

The original server (`c++/server/server.cpp`, © 2015–2017 Xavier Marchal) is licensed under the
[GNU General Public License, version 3 or later](https://www.gnu.org/licenses/gpl-3.0.html), as its header
states. The original repository has no licence file, and its other files carry no licence notice; the
author is asked, in an issue on the original repository, to confirm the same licence for the whole project. The changes made in this fork
are released under the GNU General Public License, version 3 or later. See [`LICENSE`](LICENSE). The
lock-free queue keeps its own Simplified BSD licence, stated in its files. ndnperf uses the ndn-cxx
library, distributed under LGPL-3.0.

Questions and bug reports about this fork are welcome as [GitHub issues](https://github.com/brunosfer/ndnperf/issues).
