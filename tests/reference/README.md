# Clipper2 reference fixtures

Each generator builds against Clipper2 (Abundance's vendored `ThirdParty/clipper2`) and prints a
Grail test that recomputes the same operations with `src/clipper.grl` and compares exact results;
the program exits with the number of differing cases, so `grail test` fails on any difference.

| Generator | Test | Cases |
| --- | --- | --- |
| `clipper_cases.cpp` | `tests/boolean.grl` | 154 Clipper64 boolean ops: all clip types and fill rules, holes, shared edges, collinear points, self-intersecting random polygons, axis-aligned grids |
| `tree_cases.cpp` | `tests/trees.grl` | 40 ClipperD polytree results (precision 2, 3, 8) compared by depth-first order, level, hole flag and binary64 point bits |
| `offset_cases.cpp` | `tests/offsets.grl` | 54 InflatePaths results for every join and end type, positive and negative deltas, single points, and Abundance's Round/Round water-bank call at several arc tolerances |

```sh
CL=<path-to-clipper2>
clang++ -std=c++17 -O1 -I$CL/include clipper_cases.cpp $CL/src/clipper.engine.cpp -o cases && ./cases > ../boolean.grl
clang++ -std=c++17 -O1 -I$CL/include tree_cases.cpp $CL/src/clipper.engine.cpp -o trees && ./trees > ../trees.grl
clang++ -std=c++17 -O1 -I$CL/include offset_cases.cpp $CL/src/clipper.engine.cpp $CL/src/clipper.offset.cpp -o offsets && ./offsets > ../offsets.grl
```

The expected values come from a libstdc++ build: Clipper2 sorts its intersection list with
`std::sort`, which is not stable, so the order of tied output points depends on the C++ standard
library. The port reproduces libstdc++'s introsort exactly.
