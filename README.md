## UnixDump

[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

UnixDump is a small eBPF powered utility that can be used to dump unix socket traffic.

### System requirements

This project was developed on a Ubuntu Hirsute machine (Linux Kernel 5.11).

- golang 1.16+
- Kernel headers are expected to be installed in `lib/modules/$(uname -r)`, update the `Makefile` with their location otherwise.
- clang & llvm 11.0.1+

### Build

1) If you need to rebuild the eBPF programs, use the following command:

```shell script
# ~ make build-ebpf
```

2) To build UnixDump, run:

```shell script
# ~ make build
```

3) To install UnixDump (copy to /usr/bin/unixdump) run:
```shell script
# ~ make install
```

### Getting started

UnixDump needs to run as root. Run `sudo unixdump -h` to get help.

```shell script
# ~ unixdump -h
Usage:
  unixdump [flags]

Flags:
  -c, --comm stringArray     list of filtered process comms, leave empty to capture everything
  -h, --help                 help for unixdump
  -l, --log-level string     log level, options: panic, fatal, error, warn, info, debug or trace (default "info")
      --pcap                 when set, UnixDump will export the captured data in a pcap file
  -p, --pid int              pid filter, leave empty to capture everything
      --socket stringArray   list of unix sockets you want to listen on, leave empty to capture everything
```

### Importing UnixDump in your project

You can import UnixDump in your project and provide a callback that will be called on each captured `UnixEvent`. See the sample code below:

```golang
package main

import (
	"fmt"
	"os"
	"os/signal"

	"github.com/Nirusu/unixdump/pkg/unixdump"
)

func main() {
	dump, err := unixdump.NewUnixDump(unixdump.Options{
		EventHandler: handleEvent,
	})
	if err != nil {
		fmt.Println(err)
		return
	}
	if err = dump.Start(); err != nil {
		fmt.Println(err)
		return
	}

	wait()

	_ = dump.Stop()
	return
}

func handleEvent(evt unixdump.UnixEvent) {
	fmt.Println(evt)
}

func wait() {
	sig := make(chan os.Signal, 1)
	signal.Notify(sig, os.Interrupt, os.Kill)
	<-sig
	fmt.Println()
}
```

## License

- The golang code is under Apache 2.0 License.
- The eBPF programs are under the GPL v2 License.