[![Build Status](https://travis-ci.com/aouyang1/go-matrixprofile.svg?branch=master)](https://travis-ci.com/aouyang1/go-matrixprofile)
[![codecov](https://codecov.io/gh/aouyang1/go-matrixprofile/branch/master/graph/badge.svg)](https://codecov.io/gh/aouyang1/go-matrixprofile)
[![Go Report Card](https://goreportcard.com/badge/github.com/aouyang1/go-matrixprofile)](https://goreportcard.com/report/github.com/aouyang1/go-matrixprofile)
[![GoDoc](https://godoc.org/github.com/aouyang1/go-matrixprofile?status.svg)](https://godoc.org/github.com/aouyang1/go-matrixprofile)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

# go-matrixprofile

Golang library for computing a matrix profiles and matrix profile indexes. Features also include time series discords, time series segmentation, and motif discovery after computing the matrix profile. Currently implements STMP, STAMP, STAMPI, and STOMP. STAMP and STOMP can run with multiple go routines for increased parallelization.

## Contents
- [Installation](#installation)
- [Quick start](#quick-start)
- [Case Study](#case-study)
- [Benchmarks](#benchmarks)
- [Contributing](#contributing)
- [Testing](#testing)
- [Other Libraries](#other-libraries)
- [Contact](#contact)
- [License](#license)
- [Citations](#citations)

## Installation
```sh
$ go get -u github.com/aouyang1/go-matrixprofile/matrixprofile
$ cd $GOPATH/src/github.com/aouyang1/go-matrixprofile
$ make setup
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

	if err = mp.Stomp(1); err != nil {
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
Going through a completely synthetic scenario, we'll cover what features to look for in a matrix profile, and what the additional Discords, TopKMotifs, and Segment tell us. We'll first be generating a fake signal that is composed of sine waves, noise, and sawtooth waves. We then run STOMP on the signal to calculte the matrix profile and matrix profile indexes.

![mpsin](https://github.com/aouyang1/go-matrixprofile/blob/master/mp_sine.png)
subsequence length: 32

* signal: This shows our raw data. Theres several oddities and patterns that can be seen here. 
* matrix profile: generated by running STOMP on this signal which generates both the matrix profile and the matrix profile index. In the matrix profile we see several spikes which indicate that these may be time series discords or anomalies in the time series.
* corrected arc curve: This shows the segmentation of the time series. The two lowest dips around index 420 and 760 indicate potential state changes in the time series. At 420 we see the sinusoidal wave move into a more pulsed pattern. At 760 we see the pulsed pattern move into a sawtooth pattern.
* discords: The discords graph shows the top 3 potential discords of the defined subsequence length, m, based on the 3 highest peaks in the matrix profile. This is mostly composed of noise.
* motifs: These represent the top 6 motifs found from the time series. The first being the initial sine wave pattern. The second is the sinusoidal pulse. The third is during the pulsed sequence on a fall of the pulse to the noise. The fourth is during the pulsed sequence on the rise from the noise to the pulse. The fifth is the sawtooth pattern. 

The code to generate the graph can be found in [this example](https://github.com/aouyang1/go-matrixprofile/blob/master/matrixprofile/example_caseStudy_test.go#L121). The plot can be generated by running
```sh
$ make example
=== RUN   Example
--- PASS: Example (0.16s)
=== RUN   ExampleMatrixProfile_Segment
--- PASS: ExampleMatrixProfile_Segment (0.01s)
=== RUN   ExampleMatrixProfile_TopKMotifs
--- PASS: ExampleMatrixProfile_TopKMotifs (0.01s)
```
A png file will be saved in the top level directory of the repository as `mp_sine.png`

## Benchmarks
Benchmark name                      | NumReps |    Time/Rep    |  Memory/Rep  |    Alloc/Rep  |
-----------------------------------:|--------:|---------------:|-------------:|--------------:|
BenchmarkZNormalize-4               | 10000000|       192 ns/op|      256 B/op|    1 allocs/op|
BenchmarkMovmeanstd-4               |    50000|     26831 ns/op|    65537 B/op|    4 allocs/op|
BenchmarkCrossCorrelate-4           |    10000|    145788 ns/op|    49179 B/op|    3 allocs/op|
BenchmarkMass-4                     |    10000|    153662 ns/op|    49437 B/op|    4 allocs/op|
BenchmarkDistanceProfile-4          |    10000|    152102 ns/op|    49437 B/op|    4 allocs/op|
BenchmarkCalculateDistanceProfile-4 |   200000|      8705 ns/op|        1 B/op|    0 allocs/op|
BenchmarkStmp/m32_pts1k-4           |        5| 304670190 ns/op| 97382905 B/op| 7882 allocs/op|
BenchmarkStmp/m128_pts1k-4          |        5| 291483007 ns/op| 94078201 B/op| 7498 allocs/op|
BenchmarkStamp/m32_p2_pts1k-4       |       10| 169431665 ns/op| 97498204 B/op| 7896 allocs/op|
BenchmarkStomp/m32_p1_pts1k-4       |       50|  30177279 ns/op|   251006 B/op|   21 allocs/op|
BenchmarkStomp/m128_p1_pts1k-4      |       50|  30401886 ns/op|   251025 B/op|   21 allocs/op|
BenchmarkStomp/m128_p2_pts1k-4      |      100|  15921867 ns/op|   396616 B/op|   31 allocs/op|
BenchmarkStomp/m128_p2_pts2k-4      |       20|  62799129 ns/op|   812152 B/op|   31 allocs/op|
BenchmarkStomp/m128_p2_pts5k-4      |       10| 391000142 ns/op|  2088350 B/op|   32 allocs/op|
BenchmarkStampUpdate-4              |       20|  76834458 ns/op|  1158439 B/op|   15 allocs/op|

Ran on a 2018 MacBookAir on Dec 31, 2018
```sh
    Processor: 1.6 GHz Intel Core i5
       Memory: 8GB 2133 MHz LPDDR3
           OS: macOS Mojave v10.14.2
 Logical CPUs: 4
Physical CPUs: 2
```
```sh
$ make bench
```

## Contributing
* Fork the repository
* Create a new branch (feature_\* or bug_\*)for the new feature or bug fix
* Run tests
* Commit your changes
* Push code and open a new pull request

## Testing
Run all tests including benchmarks
```sh
$ make all
```
Just run benchmarks
```sh
$ make bench
```
Just run tests
```sh
$ make test
```

## Other libraries
* R: [github.com/franzbischoff/tsmp](https://github.com/franzbischoff/tsmp)
* Python: [github.com/target/matrixprofile-ts](https://github.com/target/matrixprofile-ts)

## Contact
* Austin Ouyang
* aouyang1@gmail.com

## License
The MIT License (MIT). See [LICENSE](https://github.com/aouyang1/go-matrixprofile/blob/master/LICENSE) for more details.

Copyright (c) 2018 Austin Ouyang

## Citations
Chin-Chia Michael Yeh, Yan Zhu, Liudmila Ulanova, Nurjahan Begum, Yifei Ding, Hoang Anh Dau, Diego Furtado Silva, Abdullah Mueen, Eamonn Keogh (2016). [Matrix Profile I: All Pairs Similarity Joins for Time Series: A Unifying View that Includes Motifs, Discords and Shapelets](https://www.cs.ucr.edu/~eamonn/PID4481997_extend_Matrix%20Profile_I.pdf). IEEE ICDM 2016.

Yan Zhu, Zachary Zimmerman, Nader Shakibay Senobari, Chin-Chia Michael Yeh, Gareth Funning, Abdullah Mueen, Philip Berisk and Eamonn Keogh (2016). [Matrix Profile II: Exploiting a Novel Algorithm and GPUs to break the one Hundred Million Barrier for Time Series Motifs and Joins](https://www.cs.ucr.edu/~eamonn/STOMP_GPU_final_submission_camera_ready.pdf). IEEE ICDM 2016.

Hoang Anh Dau and Eamonn Keogh (2017). [Matrix Profile V: A Generic Technique to Incorporate Domain Knowledge into Motif Discovery](https://www.cs.ucr.edu/~eamonn/guided-motif-KDD17-new-format-10-pages-v005.pdf). KDD 2017.
