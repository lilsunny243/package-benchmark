[![](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2Fordo-one%2Fpackage-benchmark%2Fbadge%3Ftype%3Dswift-versions)](https://swiftpackageindex.com/ordo-one/package-benchmark)
[![](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2Fordo-one%2Fpackage-benchmark%2Fbadge%3Ftype%3Dplatforms)](https://swiftpackageindex.com/ordo-one/package-benchmark)
[![Swift Linux build](https://github.com/ordo-one/package-benchmark/actions/workflows/swift-linux-build.yml/badge.svg)](https://github.com/ordo-one/package-benchmark/actions/workflows/swift-linux-build.yml)
[![Swift macOS build](https://github.com/ordo-one/package-benchmark/actions/workflows/swift-macos-build.yml/badge.svg)](https://github.com/ordo-one/package-benchmark/actions/workflows/swift-macos-build.yml)
[![Swift address sanitizer](https://github.com/ordo-one/package-benchmark/actions/workflows/swift-sanitizer-address.yml/badge.svg)](https://github.com/ordo-one/package-benchmark/actions/workflows/swift-sanitizer-address.yml)
[![Swift thread sanitizer](https://github.com/ordo-one/package-benchmark/actions/workflows/swift-sanitizer-thread.yml/badge.svg)](https://github.com/ordo-one/package-benchmark/actions/workflows/swift-sanitizer-thread.yml)
[![codecov](https://codecov.io/gh/ordo-one/package-benchmark/branch/main/graph/badge.svg?token=hXHmhEG1iF)](https://codecov.io/gh/ordo-one/package-benchmark)

# Benchmark 

Benchmark allows you to easily create sophisticated Swift performance benchmarks

## Overview

Performance is a key feature for many apps and frameworks. Benchmark helps make it easy to measure and track many different metrics that affects performance, such as CPU usage, memory usage and use of operating system resources such as threads and system calls.

Benchmark works on both macOS and Linux and supports several key workflows for performance measurements:

* **Automated Pull Request performance regression checks** by comparing the performance metrics of a pull request with the main branch and having the PR workflow check fail if there is a regression according to absolute or relative thresholds specified per benchmark
* **Manual comparison of multiple performance baselines** for iterative or A/B performance work by an individual developer
* **Export of benchmark results in several formats** for analysis or visualization

Benchmark provides a quick way for validation of performance metrics, while other more specialized tools such as Instruments, DTrace, Heaptrack, Leaks, Sample and more can be used for finding root causes for any deviations found.

Benchmark is suitable for both smaller ad-hoc benchmarks focusing on execution time and more extensive benchmarks that care about several additional metrics such as memory allocations, syscalls, thread usage, context switches, and more. Thanks to the [Histogram foundation](ttps://github.com/ordo-one/package-histogram) it’s especially suitable for capturing latency statistics for large number of samples.

## Documentation

Documentation on how to use Benchmark in your Swift package can be [viewed online](https://swiftpackageindex.com/ordo-one/package-benchmark/main/documentation/benchmark) (hosted by the Swift Package Index, thanks!) or inside Xcode using `Build Documentation`. 
Additionally the command plugin provides help information if you run `swift package benchmark help` from the command line.

## Output

The default text output from Benchmark is oriented around [the five-number summary](https://en.wikipedia.org/wiki/Five-number_summary) percentiles, plus the last decile (`p90`) and the last percentile (`p99`) - it's thus a variation of a [seven-figure summary](https://en.wikipedia.org/wiki/Seven-number_summary) with the focus on the 'bad' end of results (as those are what we typically care about addressing).
We've found that focusing on percentiles rather than average or standard deviations, is more useful for a wider range of benchmark measurements and gives a deeper understanding of the results.
Percentiles allows for a consistent way of expressing benchmark results of both throughput and latency measurements (which typically do **not** have a standardized distribution, being almost always multi-modal in nature).
This multi-modal nature of the latency measurements leads to the common statistical measures of mean and standard deviation being potentially misleading.

## Writing benchmarks
There are [documentation available](https://swiftpackageindex.com/ordo-one/package-benchmark/main/documentation/benchmark/writingbenchmarks) as well as a [a sample project](https://github.com/ordo-one/package-benchmark-samples) using various aspects of this package in practice.

## Sample benchmark code
```swift
import BenchmarkSupport
@main extension BenchmarkRunner {}
@_dynamicReplacement(for: registerBenchmarks)

func benchmarks() {

    Benchmark("Minimal benchmark") { benchmark in
      // measure something here
    }

    Benchmark("All metrics, full concurrency, async",
              metrics: BenchmarkMetric.all,
              maxDuration: .seconds(10)) { benchmark in
        let _ = await withTaskGroup(of: Void.self, returning: Void.self, body: { taskGroup in
            for _ in 0..< 80  {
                taskGroup.addTask {
                    dummyCounter(defaultCounter()*1000)
                }
            }
            for await _ in taskGroup {
            }
        })
    }
}
```

### Running benchmarks
To execute all defined benchmarks, simply run:

```swift package benchmark```

Please see the [documentation](https://swiftpackageindex.com/ordo-one/package-benchmark/main/documentation/benchmark/runningbenchmarks) for more detail on all options.

### Sample output benchmark run
<img width="1005" alt="image" src="https://user-images.githubusercontent.com/8501048/225281508-7c732325-1276-49bf-bff9-031a62afed0a.png">

### Sample output benchmark grouped by metric 
<img width="1089" alt="image" src="https://user-images.githubusercontent.com/8501048/225281786-411530de-25c2-47b5-b99f-0d7bac3209a7.png">

### Sample output delta comparison
<img width="1173" alt="image" src="https://user-images.githubusercontent.com/8501048/225282373-e7ba9fa1-1a2a-4028-b053-9f3aa82361b0.png">

### Sample output threshold deviation check
<img width="956" alt="image" src="https://user-images.githubusercontent.com/8501048/225282982-95c9c641-9455-4df2-81bc-6aee43721223.png">

### Sample usage of Youplot
```bash
swift package benchmark run --filter InternalUTCClock-now --metric wallClock --format histogramPercentiles --path stdout --no-progress | uplot lineplot -H
```
<img width="523" alt="image" src="https://user-images.githubusercontent.com/8501048/225284254-c1349494-2323-4460-b18a-7bc2896b5dc4.png">

### API and file format stability
The API will be deemed stable as of `1.0.0` and follows semantical versioning for future releases. 

The export file formats that are externally defined (e.g. JMH or HDR Histogram formats) will follow the upstream definitions if they change, but have been quite stable for several years. 

The Histogram codable representation is not stable and may change if the Histogram implementation changes.

The benchmark internal baseline representation (stored in `.benchmarkBaselines`) is not stable and is not viewed as public API and may break over time.

For those wanting to save benchmark data over time, it's recommended to export data in e.g. HDR Histogram representations (percentiles, average, stddev etc) or simply post processing the histogramSamples format (which is raw data) to your desired representation.

PR:s for additional standardized formats are welcome, as the export formats are the intended stable interface for saving such data.

### CI build note
The badges above shows that macOS builds are failing on the CI [as GitHub still haven't provided runners for macOS 13 Ventura](https://github.com/actions/runner-images/issues/6426), it works in practice.

