# grail-clipper

Polygon clipping and offsetting for [Grail](https://github.com/graillang/grail): a port of
[Clipper2](https://github.com/AngusJohnson/Clipper2) by Angus Johnson that reproduces the C++
results **exactly** (same paths, same point order, same binary64 coordinates).

Covered:

- the Vatti clipping engine: intersection, union, difference and xor under the even-odd,
  non-zero, positive and negative fill rules (`Clipper64`, integer coordinates);
- `ClipperD`: double coordinates scaled by a power of two per decimal precision;
- polytree results (outer rings, holes and nested islands) with Clipper2's owner and split
  resolution;
- `InflatePaths`: offsetting closed polygons and open paths with square, bevel, round and miter
  joins and polygon, joined, butt, square and round ends.

Not ported yet: rectangle clipping, Minkowski sums, path simplification, the Z callback and the
delta callback.

## Use

Add the package to your project's `grail.pkg`:

```
dep clipper https://github.com/sashaduke/grail-clipper
```

```grail
use clipper;

fn main() {
    mut subject = clipper::paths_d();
    clipper::push_path_d(&mut subject, f64[0.0, 10.0, 10.0, 0.0], f64[0.0, 0.0, 10.0, 10.0]);
    mut clip = clipper::paths_d();
    clipper::push_path_d(&mut clip, f64[5.0, 15.0, 15.0, 5.0], f64[5.0, 5.0, 15.0, 15.0]);

    merged := clipper::boolean_op_d(clipper::CLIP_UNION, clipper::FILL_NON_ZERO, subject, clip, 2);
    println(merged.len());

    // Polytree: node 0 is the root; tree_is_hole and parent give the ring hierarchy.
    mut a = clipper::paths_d();
    clipper::push_path_d(&mut a, f64[0.0, 10.0, 10.0, 0.0], f64[0.0, 0.0, 10.0, 10.0]);
    mut b = clipper::paths_d();
    clipper::push_path_d(&mut b, f64[2.0, 2.0, 8.0, 8.0], f64[2.0, 8.0, 8.0, 2.0]);
    tree := clipper::boolean_op_tree_d(clipper::CLIP_DIFFERENCE, clipper::FILL_NON_ZERO, a, b, 8);
    println(clipper::tree_count(&tree));

    // Offset an open path into a band with round joins and ends.
    mut line = clipper::paths_d();
    clipper::push_path_d(&mut line, f64[0.0, 20.0, 40.0], f64[0.0, 3.5, -2.0]);
    band := clipper::inflate_paths_d(line, 2.5, clipper::JOIN_ROUND, clipper::END_ROUND,
        clipper::OffsetOptions { miter_limit: 2.0, precision: 3, arc_tolerance: 0.05 });
    println(band.len());
}
```

## Verification

```sh
grail test
```

runs 248 reference cases generated from the C++ library (see `tests/reference/README.md`) and
fails on any difference. Notes on exactness:

- Clipper2 sorts intersections with the unstable `std::sort`; the port reproduces libstdc++'s
  introsort, so results match a libstdc++ (Linux) build of Clipper2.
- Round joins use `sin`, `cos`, `acos` and `atan2` from Grail's math library, which agree with
  glibc to within an ulp; every reference case matches bit for bit, but an output point that
  lands exactly on a rounding boundary could still differ from glibc in rare cases.

## License

MIT (see `LICENSE`); the upstream Boost Software License notice is retained in `NOTICE`.
