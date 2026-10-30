# lowend

**Measure before you guess. Everything else is folklore.**

A CLI that runs your Dart benchmark script many times, throws away the
warmup, and reports the distribution. The script times itself; `lowend`
controls the runs and does the statistics. Zero dependencies, no
telemetry, no published numbers it can't reproduce.

## Why

`package:benchmark_harness` times a function inside one process. That's
the right tool for microbenchmarks. `lowend` answers a different
question: *given N separate runs of this thing, what does the
distribution look like?* That's the number that predicts production, and
nothing in the Dart ecosystem reports it.

## Install

    dart pub global activate --source git https://github.com/thorin-valehart/lowend

## Two ways to use it

### As a library — time individual calls

```dart
import 'package:lowend/lowend.dart';

void main() {
  final r = bench('cache.get hit', () => cache.get('hot-key'),
      iterations: 10000);
  print(r.toJsonLine());
  // {"name":"cache.get hit","p50_us":41,"p99_us":180,"iterations":10000,...}
}
```

### As a protocol

Your benchmark is a plain Dart script with no dependency on `lowend`.
It runs its own loop and prints one JSON line:

```dart
// bench/cache.dart
import 'dart:convert';

void main() {
  final sw = Stopwatch()..start();
  for (var i = 0; i < 10000; i++) {
    cache.get('hot-key');
  }
  sw.stop();
  print(jsonEncode({'micros': sw.elapsedMicroseconds, 'iterations': 10000}));
}
```
