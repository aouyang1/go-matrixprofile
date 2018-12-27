[![Build Status](https://travis-ci.com/aouyang1/go-matrixprofile.svg?branch=master)](https://travis-ci.com/aouyang1/go-matrixprofile)
[![codecov](https://codecov.io/gh/aouyang1/go-matrixprofile/branch/master/graph/badge.svg)](https://codecov.io/gh/aouyang1/go-matrixprofile)
[![Go Report Card](https://goreportcard.com/badge/github.com/aouyang1/go-matrixprofile)](https://goreportcard.com/report/github.com/aouyang1/go-matrixprofile)
[![GoDoc](https://godoc.org/github.com/aouyang1/go-matrixprofile?status.svg)](https://godoc.org/github.com/aouyang1/go-matrixprofile)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

# go-matrixprofile

Golang library for computing a matrix profiles and matrix profile indexes. Features also include time series segmentation and motif discovery after computing the matrix profile.

## Contents
- [Installation](#installation)
- [Quick start](#quick-start)
- [Case Study](#case-study)
- [Benchmarks](#benchmarks)
- [Contributing](#contributing)
- [Testing](#testing)
- [Contact](#contact)
- [License](#license)

## Installation
```sh
$ go get -u github.com/aouyang1/go-matrixprofile/matrixprofile
```

## Quick start
```sh
$ cat example_mp.go
```
```go
package main

import (
	"fmt"

	"github.com/aouyang1/go-matrixprofile/matrixprofile"
)

func main() {
	sig := []float64{0, 0.99, 1, 0, 0, 0.98, 1, 0, 0, 0.96, 1, 0}

	mp, err := matrixprofile.New(sig, nil, 4)
	if err != nil {
		panic(err)
	}

	if err = mp.Stmp(); err != nil {
		panic(err)
	}

	fmt.Printf("Signal:         %.3f\n", sig)
	fmt.Printf("Matrix Profile: %.3f\n", mp.MP)
	fmt.Printf("Profile Index:  %5d\n", mp.Idx)
}
```
```sh
$ go run example_mp.go
Signal:         [0.000 0.990 1.000 0.000 0.000 0.980 1.000 0.000 0.000 0.960 1.000 0.000]
Matrix Profile: [0.014 0.014 0.029 0.029 0.014 0.014 0.029 0.029 0.029]
Profile Index:  [    4     5     6     7     0     1     2     3     4]
```

## Case study
Going through a completely synthetic scenario, we'll go through what features to look for in a matrix profile, and what the additional Discords, TopKMotifs, and Segment tell us. We'll first be generating a fake signal that composed of sine waves, noise, and sawtooth waves. We then run STMP on the signal triggering a self join.

![mpsin](https://github.com/aouyang1/go-matrixprofile/blob/master/mp_sine.png)
subsequence length: 32

* signal: This shows our raw data. Theres several oddities and patterns that can be seen here. 
* matrix profile: generated by running STMP on this signal which generates both the matrix profile and the matrix profile index. In the matrix profile we see several spikes which indicate that these may be time series discords or anomalies in the time series.
* corrected arc curve: This shows the segmentation of the time series. The two lowest dips around index 420 and 760 indicate potential state changes in the time series. At 420 we see the sinusoidal wave move into a more pulsed pattern. At 760 we see the pulsed pattern move into a sawtooth pattern.
* discords: The discords graph shows the top 3 potential discords of the defined subsequence length, m, based on the 3 highest peaks in the matrix profile.
* motifs: These represent the top 6 motifs found from the time series. The first one being the initial sine wave pattern. The second is during the pulsed sequence on a fall of the pulse to noise. The third is during the pulsed sequence on a rise from the noise to the pulse. The fourth is the sawtooth pattern. The remaining motifs don't look particularly tight, thus may not be relevant motifs.

The code to generate the graph can be found in [this example](https://github.com/aouyang1/go-matrixprofile/blob/master/matrixprofile/example_test.go#L120). The plot can be generated by running
```sh
$ go test -v -run=Example
=== RUN   Example
--- PASS: Example (0.16s)
=== RUN   ExampleMatrixProfile_Segment
--- PASS: ExampleMatrixProfile_Segment (0.01s)
=== RUN   ExampleMatrixProfile_TopKMotifs
--- PASS: ExampleMatrixProfile_TopKMotifs (0.01s)
```
A png file will be saved in the top level directory of the repository as `mp_sine.png`

## Benchmarks
Benchmark name               | NumReps |    Time/Rep   |  Memory/Rep  |   Alloc/Rep
-----------------------------|--------:|--------------:|-------------:|--------------:
BenchmarkZNormalize-4        | 10000000|      178 ns/op|      256 B/op|    1 allocs/op
BenchmarkMovstd-4            |   200000|    11908 ns/op|    27136 B/op|    3 allocs/op
BenchmarkCrossCorrelate-4    |    20000|    66093 ns/op|    34053 B/op|    4 allocs/op
BenchmarkMass-4              |    20000|    68604 ns/op|    42501 B/op|    6 allocs/op
BenchmarkDistanceProfile-4   |    20000|    66735 ns/op|    42501 B/op|    6 allocs/op
BenchmarkStmp/m16-4          |       20| 71546986 ns/op| 42202424 B/op| 5958 allocs/op
BenchmarkStmp/m32-4          |       20| 67491284 ns/op| 42202424 B/op| 5958 allocs/op
BenchmarkStmp/m64-4          |       20| 72172709 ns/op| 42202424 B/op| 5958 allocs/op
BenchmarkStmp/m128-4         |       20| 68277992 ns/op| 42202424 B/op| 5958 allocs/op

Ran on a 2018 MacBookAir on Dec 27, 2018
```
Processor: 1.6 GHz Intel Core i5
   Memory: 8GB 2133 MHz LPDDR3   
       OS: macOS Mojave v10.14.2
```

## Contributing
* Fork the repository
* Create a new feature branch for the new feature or bug fix
* Run tests
* Commit your changes
* Push code and open a new pull request

## Testing
Run all tests including benchmarks
```sh
$ go test -v ./... -bench=.
```
Just run tests
```sh
$ go test -v ./...
```
## Contact
* Austin Ouyang
* aouyang1@gmail.com

## License
The MIT License (MIT). See [LICENSE](https://github.com/aouyang1/go-matrixprofile/blob/master/LICENSE) for more details.

Copyright (c) 2018 Austin Ouyang
