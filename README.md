# Introduction

[![Build Status](https://travis-ci.org/spance/suft.svg)](https://travis-ci.org/spance/suft)
[![GoDoc](https://godoc.org/github.com/spance/suft/protocol?status.svg)](https://godoc.org/github.com/spance/suft/protocol)

The SUFT (Small-scale UDP Fast Transmission) Protocol is an application layer transmission protocol based on UDP and implemented in Golang. It has lower latency than TCP and provides reliable and ordered delivery of a stream of octets under plain congestion control.

The protocol seeks for maximized throughput and minimizes impact of lost packets on throughput. It is only for small/medium-scale communication or some situations where TCP is not applicable.

# Goals & Features

- Transmitting model has predictable performance.
- Fast retransmission mode does better on lossy link.
- Minimum retransmission mode doesn't waste traffic.
- No resource consumption while the connection is idle.
- Special modes for certain situations.

# Protocol APIs

SUFT implements the Golang: [net.Conn](https://golang.org/pkg/net/#Conn) and [net.Listener](https://golang.org/pkg/net/#Listener) interfaces completely.

```go
import "github.com/spance/suft/protocol"

e, err := suft.NewEndpoint(p *suft.Params)
// for server
conn := e.Listen() // or e.Accept()
// for client
conn, err := e.Dial(rAddr string)
```

# Basic Theories

Sent-count(aka, the count of unique data packets) and lost-count

```
    lscnt
-------------- = Lose Rate
 scnt + lscnt

    lscnt
-------------- = Retransmit Rate
    scnt
```

latency, window and traffic speed

```
  1000
--------- * mss * win = Speed
 latency
```

# License

GPL version 3 or any later version

    SUFT Protocol
    Copyright (C) <2016>  <spance, l2dy>

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.

# Tool Usage

Main package include a tool for testing, similar to netcat (nc).

Build with `go get -u -v github.com/spance/suft/suft-nc`

```
./suft-nc [-l addr:port] [-r addr:port] [-s] [-b 10] [-fr] < [send_file] > [recv_file]

-l:  local bind address, e.g. localhost:9090 or :8080
-r:  remote address (for client), e.g. 8.8.8.8:9090 or examples.com:8080
-b:  max bandwidth of sending in mbps (be careful, see Notes#2)
-s:  for server
-fr: enable fast retransmission (useful for lossy link)
-sr: don't shrink window when a lot of packets were lost
-ft: flat traffic (slow down bursty traffic, useful when sender has more bandwidth than receiver)
```

Examples:

```
// send my_file to remote in 10mbps
remote# ./suft-nc -l :9090 -s > recv_file
local# ./suft-nc -l :1234 -r remote:9090 -b 10 -fr < my_file
```

```
// recv my_file from remote in 20mbps
remote# ./suft-nc -l :9090 -s -b 20 -fr < my_file
local# ./suft-nc -l :1234 -r remote:9090 > recv_file
```

```
// simple chat room
remote# ./suft-nc -l :9090 -s
local# ./suft-nc -l :1234 -r remote:9090
```

Notes:

1. The target to be connected shouldn't be behind NAT (or should use port mapping).
2. Use improper bandwidth(-b) may waste huge bandwidth and may be suspected of carrying out DoS flood.

# Known Issues

1. Numbers

   seq, ack... use uint32, that means one connection cannot transmit more than 4G packets (bytes ∈ [4GB, 5.6TB]).

2. Detecting channel capacity

   To do or NOT ?

3. Test

   Needs a lot of testing under real-world scenes. Welcome to share your test results.
