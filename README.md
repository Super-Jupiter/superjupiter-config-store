# SuperJupiter ConfigStore

<!-- SUPERJUPITER-CONSOLIDATION-MIGRATION:START -->
## Consolidated library available

For new projects, start with **[SuperJupiter Embedded Reliability](https://github.com/Super-Jupiter/superjupiter-embedded-reliability)**. This standalone package and its published releases remain available. Existing installations need no change.

If migrating with PlatformIO, replace the standalone dependency with:

```ini
lib_deps = superjupiter/SuperJupiterEmbeddedReliability@0.1.0
```

Replace this standalone include:

```cpp
#include <SuperJupiterConfigStore.h>
```

with this consolidated include:

```cpp
#include <SuperJupiter/EmbeddedReliability/ConfigStore.h>
```

The component namespace and behaviour are preserved from standalone **0.1.1**. If using an earlier version, review its API migration requirements first. Use one implementation of this component per application; do not mix standalone and consolidated copies.

See the [consolidated migration guide](https://github.com/Super-Jupiter/superjupiter-embedded-reliability/blob/v0.1.0/docs/MIGRATION.md) and consolidated README for Arduino/CMake installation. **Hardware validation pending.**
<!-- SUPERJUPITER-CONSOLIDATION-MIGRATION:END -->

Two-slot versioned configuration with integrity checks and recoverable writes.

**Version 0.1.1: evolving preview. Hardware validation pending.**

Original MIT-licensed portable C++11 by [SuperJupiter](https://www.superjupiter.net).
No mandatory service, hidden clock reads, network calls or heap allocation in the core.

## What it does

Caller supplies byte-rewritable storage of requiredBytes(), initially 0xff. Adapter sync must make prior writes durable.
Two slots contain fixed-size opaque bytes, schema ID, monotonic generation, CRC32 and a final commit marker.
load selects the newest valid slot, falls back from a damaged slot and reports Recovered; schema mismatch is explicit and blocks saves.
Application performs any schema migration itself in a separately controlled process. No implicit reinterpretation of old bytes.
Any failed save locks the instance until load reconciles it; a failed final sync may have committed, so never blindly retry.
No encryption, access control, wear levelling, raw-flash erase support or cross-task locking. Hardware power-loss validation pending.

## Install

PlatformIO exact release (after registry publication):

```ini
lib_deps = superjupiter/SuperJupiterConfigStore@0.1.1
```

Arduino: use the versioned [GitHub release ZIP](https://github.com/Super-Jupiter/superjupiter-config-store/releases/tag/v0.1.1).
Library Manager registration and index propagation are separate steps; check the official index before relying on IDE availability.

## Example

```cpp
#include <SuperJupiterConfigStore.h>
using namespace superjupiter::config_store;
int main() {
class Memory: public Storage {
public:
 uint8_t bytes[256];int budget;bool failSync;
 Memory():budget(-1),failSync(false) {for(size_t i=0;i<sizeof(bytes);++i) bytes[i]=255;}
 bool read(size_t o,uint8_t* p,size_t n) {if(o>sizeof(bytes)||n>sizeof(bytes)-o) return false;for(size_t i=0;i<n;++i) p[i]=bytes[o+i];return true;}
 bool write(size_t o,const uint8_t* p,size_t n) {if(o>sizeof(bytes)||n>sizeof(bytes)-o) return false;for(size_t i=0;i<n;++i) {if(budget==0) return false;bytes[o+i]=p[i];if(budget>0) --budget;}return true;}
 bool sync() {return !failSync;}
};

Memory storage; // RAM illustration, replace with a durable adapter.
ConfigStore<4> settings(storage,1);
uint8_t bytes[4]={10,0,0,0};uint8_t previous[4]={};
settings.load(previous);settings.save(bytes);
return 0;
}

```

For Arduino, open `examples/Minimal`. It uses synthetic data only.
Results are deliberately local so applications can inspect outcomes before adding I/O.

## CMake

```cmake
add_subdirectory(path/to/superjupiter-config-store)
target_link_libraries(your_app PRIVATE SuperJupiter::ConfigStore)
```

For an installed package use `find_package(SuperJupiterConfigStore 0.1 CONFIG REQUIRED)`.
Top-level CMake builds tests/examples; a subproject does not enable developer tools by default.

## Documentation and verification

- [API and boundaries](https://github.com/Super-Jupiter/superjupiter-config-store/blob/v0.1.1/docs/API.md)
- [Tests](https://github.com/Super-Jupiter/superjupiter-config-store/blob/v0.1.1/tests/test.cpp)
- [Validation status](https://github.com/Super-Jupiter/superjupiter-config-store/blob/v0.1.1/docs/VALIDATION.md)
- [Shared conventions](https://github.com/Super-Jupiter/superjupiter-config-store/blob/v0.1.1/docs/CONVENTIONS.md)
- [Migration policy](https://github.com/Super-Jupiter/superjupiter-config-store/blob/v0.1.1/docs/MIGRATION.md)

```sh
cmake -S . -B build/cmake
cmake --build build/cmake
ctest --test-dir build/cmake --output-on-failure
python3 tools/check_release.py --tag v0.1.1
```

Keep raw observations and their provenance in your application. Derived results are not physical truth.
Do not use fast-math or finite-math-only compiler assumptions. AVR double may have float precision.
Software tests, board compilation and physical validation are reported separately.

## Fits the SuperJupiter measurement portfolio

Complements [SensorQuality](https://github.com/Super-Jupiter/superjupiter-sensor-quality) and
[SampleWindow](https://github.com/Super-Jupiter/superjupiter-sample-window) without a required dependency.
Each component can be used independently. Domain models, customer data and proprietary applications are excluded.

## Support and licence

MIT. Report reproducible bugs through repository issues; private security reports: info@superjupiter.net.
Include version, configuration, inputs, expected and observed behaviour. No credentials or private customer data.


## Compatibility review 0.1.1

This patch includes independent regression tests and a reviewed [interoperability contract](https://github.com/Super-Jupiter/superjupiter-config-store/blob/v0.1.1/docs/INTEROPERABILITY.md). Hardware validation pending.

- Removed unused duplicated math helpers and unnecessary includes.
- Sequential stale configuration writers cannot replace newer generations; no wire-format change.
