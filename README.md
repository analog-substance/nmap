# nmap

<p align="center">
    <img width="350" src="img/logo.png"/>
<p>

<p align="center">
    <a href="LICENSE">
        <img src="https://img.shields.io/badge/license-MIT-blue.svg?style=flat" />
    </a>
    <a href="https://pkg.go.dev/github.com/analog-substance/nmap/v3"><img src="https://pkg.go.dev/badge/github.com/analog-substance/nmap/v3" alt="PkgGoDev github.com/analog-substance/nmap/v3"></a>
    <a href="https://goreportcard.com/report/github.com/analog-substance/nmap/v3">
        <img src="https://goreportcard.com/badge/github.com/analog-substance/nmap/v3">
    </a>
    <a href="https://github.com/analog-substance/nmap/actions/workflows/build.yml">
        <img src="https://github.com/analog-substance/nmap/actions/workflows/build.yml/badge.svg">
    </a>
    <a href="https://github.com/analog-substance/nmap/actions/workflows/test.yml">
        <img src="https://github.com/analog-substance/nmap/actions/workflows/test.yml/badge.svg">
    </a>
    <a href='https://coveralls.io/github/analog-substance/nmap'>
        <img src='https://coveralls.io/repos/github/analog-substance/nmap/badge.svg' alt='Coverage Status' />
    </a>

<p>

This library aims at providing idiomatic `nmap` bindings for go developers, in order to make it easier to write security audit tools using golang.

## What is nmap

Nmap (Network Mapper) is a free and open-source network scanner created by [Gordon Lyon](https://en.wikipedia.org/wiki/Gordon_Lyon). Nmap is used to discover hosts and services on a computer network by sending packets and analyzing the responses.

Nmap provides a number of features for probing computer networks, including host discovery and service and operating system detection. These features are extensible by scripts that provide more advanced service detection, vulnerability detection, and other features. Nmap can adapt to network conditions including latency and congestion during a scan.

## Why use go for penetration testing

Most pentest tools are currently written using Python and not Go, because it is easy to quickly write scripts, lots of libraries are available, and it's a simple language to use. However, for writing robust and reliable applications, Go is the better tool. It is statically compiled, has a static type system, much better performance, it is also a very simple language to use and goroutines are awesome... But I might be slighly biased, so feel free to disagree.

## Supported features

- [x] All of `nmap`'s native options.
- [x] Additional [idiomatic go filters](examples/service_detection/main.go#L19) for filtering hosts and ports.
- [x] Helpful enums for nmap commands. (time templates, os families, port states, etc.)
- [x] Complete documentation of each option, mostly insipred from nmap's documentation.
- [x] Run a nmap scan asynchronously.
- [x] Scan progress can be piped through a channel.
- [x] Write the nmap output to a given file while also parsing it to the struct.
- [x] Stream the nmap output to an `io.Writer` interface while also parsing it to the struct.
- [x] Functionality to show local interfaces and routes.

## Simple example

```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "github.com/analog-substance/nmap/v3"
)

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Minute)
	defer cancel()

	// Equivalent to `/usr/local/bin/nmap -p 80,443,843 google.com facebook.com youtube.com`,
	// with a 5-minute timeout.
	scanner, err := nmap.NewScanner(
		ctx,
		nmap.WithTargets("google.com", "facebook.com", "youtube.com"),
		nmap.WithPorts("80,443,843"),
	)
	if err != nil {
		log.Fatalf("unable to create nmap scanner: %v", err)
	}

	result, warnings, err := scanner.Run()
	if len(*warnings) > 0 {
		log.Printf("run finished with warnings: %s\n", *warnings) // Warnings are non-critical errors from nmap.
	}
	if err != nil {
		log.Fatalf("unable to run nmap scan: %v", err)
	}

	// Use the results to print an example output
	for _, host := range result.Hosts {
		if len(host.Ports) == 0 || len(host.Addresses) == 0 {
			continue
		}

		fmt.Printf("Host %q:\n", host.Addresses[0])

		for _, port := range host.Ports {
			fmt.Printf("\tPort %d/%s %s %s\n", port.ID, port.Protocol, port.State, port.Service.Name)
		}
	}

	fmt.Printf("Nmap done: %d hosts up scanned in %.2f seconds\n", len(result.Hosts), result.Stats.Finished.Elapsed)
}
```

The program above outputs:

```bash
Host "172.217.16.46":
    Port 80/tcp open http
    Port 443/tcp open https
    Port 843/tcp filtered unknown
Host "31.13.81.36":
    Port 80/tcp open http
    Port 443/tcp open https
    Port 843/tcp open unknown
Host "216.58.215.110":
    Port 80/tcp open http
    Port 443/tcp open https
    Port 843/tcp filtered unknown
Nmap done: 3 hosts up scanned in 1.29 seconds
```

## Advanced example

[Cameradar](https://github.com/Ullaakut/cameradar) already uses this library at its core to communicate with nmap, discover RTSP streams and access them remotely.

<p align="center">
   <img src="https://raw.githubusercontent.com/Ullaakut/cameradar/master/images/Cameradar.gif" width="100%"/>
</p>

More examples:

- [Basic scan](examples/basic_scan/main.go)
- [Basic scan but asynchronously](examples/basic_scan_async/main.go)
- [Basic scan with nmap progress piped through](examples/basic_scan_progress/main.go)
- [Basic scan with output to a streamer](examples/basic_scan_streamer_interface/main.go)
- [Count hosts for each operating system on a network](examples/count_hosts_by_os/main.go)
- [Service detection](examples/service_detection/main.go)
- [IP address spoofing and decoys](examples/spoof_and_decoys/main.go)
- [List local interfaces](examples/list_interfaces/main.go)

## External resources

- [Official nmap documentation](https://nmap.org/docs.html)
- [Nmap reference guide](https://nmap.org/book/man.html)
