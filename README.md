# bild_modfor8bhg

> **This is a fork of [anthonynsimon/bild](https://github.com/anthonynsimon/bild)**, modified specifically for use by [8-bit Hubble Golang](https://github.com/amcajal/8_bit_hubble_golang).

The upstream `bild` library is a pure Go image processing library. This fork introduces targeted changes to its noise and Perlin generation pipeline so that the library uses a **caller-supplied random source** instead of managing its own, enabling fully deterministic and reproducible image output when a seed is provided.

No changes have been made to any other part of the library (blending, transforms, effects, etc.).

---

## Changelog

### Modifications vs. upstream bild

#### `noise/noise.go`

- Added a package-level `var randSource *rand.Rand` to hold a caller-supplied random source.
- Added `SetRandSource(r *rand.Rand)` — public function that lets the importing program inject its own seeded `*rand.Rand`. Pass `nil` to revert to the global `rand` source.
- Added `rebuildFns()` — internal helper that rebuilds the `Uniform`, `Binary`, and `Gaussian` noise closures to use `randSource` when set, or the global `rand` package functions otherwise.
- `Uniform`, `Binary`, and `Gaussian` closures are now protected by a `sync.Mutex` (`randMu`) when using a custom source. This is necessary because `parallel.Line` dispatches noise functions concurrently across goroutines, and `*rand.Rand` is not goroutine-safe.
- `GeneratePerlin` now calls `perlin.NewPerlinRandSource(..., randSource)` directly instead of `perlin.NewPerlin(..., rand.Int63())`, routing Perlin generation through the same controlled source.
- **Bug fix:** the pixel-writing loop in `GeneratePerlin` had its axes swapped (`x` iterated over height, `y` over width). At non-square dimensions this left portions of the image unpainted. Fixed to iterate `y` over `[0, height)` and `x` over `[0, width)`, matching Go's `image.Set(column, row)` convention.

#### `perlin/perlin.go`

- `NewPerlin` (existing public entrypoint) now delegates to `NewPerlinRandSource` passing `nil`, preserving backward compatibility.
- `NewPerlinRandSource` updated to accept a `*rand.Rand` parameter. When non-nil, all internal random calls (`rand.Int()`) use that instance; when nil, falls back to the global `rand` source. This replaces the previous version which always used the global source unconditionally.

---

## Usage of `SetRandSource`

```go
import (
    "math/rand"
    "github.com/amcajal/bild_modfor8bhg/noise"
)

r := rand.New(rand.NewSource(42))
noise.SetRandSource(r)

// All subsequent noise.Generate* and noise.GeneratePerlin calls
// will draw randomness from r, making output deterministic for seed 42.
img := noise.GeneratePerlin(800, 600, 0.15)
```

Pass `nil` to revert to non-deterministic behaviour:

```go
noise.SetRandSource(nil)
```

---

## Original library

All credit for the original library goes to [anthonynsimon](https://github.com/anthonynsimon) and the `bild` contributors.
See the original [README](https://github.com/anthonynsimon/bild/blob/master/README.md) and [LICENSE](https://github.com/anthonynsimon/bild/blob/master/LICENSE) for full documentation and licensing terms.
