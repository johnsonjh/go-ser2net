# goser2net

[![GoReportCard](https://goreportcard.com/badge/github.com/BAN-AI-Communications/goser2net)](https://goreportcard.com/badge/github.com/BAN-AI-Communications/goser2net)
[![Maintainability](https://api.codeclimate.com/v1/badges/3c38ebecd2c09b539180/maintainability)](https://codeclimate.com/github/BAN-AI-Communications/goser2net/maintainability)

A serial to telnet client and library as replacement for ser2net.

# How to use the library

To spawn a telnet server on port 5555 and redirect to ttyS0 running at 115200N8
use this code example:

```go
import (
	"github.com/BAN-AI-Communications/goser2net/pkg/ser2net"
)

        w, _ := ser2net.NewSerialWorker("/dev/ttyS0")
	// Run serial worker in new routine
        go w.Worker()

	// Start telnet on port 5555 and serve forever
        err := w.StartTelnet("", 5555)
        if nil != err {
                panic(err)
        }

	// Start GoTTY on port 5556 and serve forever
	err = w.StartGoTTY("", 5556, "")
        if nil != err {
                panic(err)
        }
```

To use an io.ReadWriter do:

```go
import (
        "github.com/BAN-AI-Communications/goser2net/pkg/ser2net"
)

	w, _ := ser2net.NewSerialWorker("/dev/ttyS0")
	// Run serial worker in new routine
	go w.Worker()

	// Get a ReadWriteCloser interface
	i, err := w.NewIoReadWriteCloser()
	if nil != err {
		panic(err)
	}
	defer i.Close()

	// Copy serial out to stdout
	go func() {
		p := make([]byte, 1)
		for {
			n, err := i.Read(p)
			if err != nil {
				break
			}
			fmt.Printf("%s", string(p[:n]))
		}
	}()

	// Copy stdin to serial
	reader := bufio.NewReader(os.Stdin)
	p := make([]byte, 1)
	for {
		_, err := reader.Read(p)
		if err != nil {
			break
		}

		_, err = i.Write(p)
		if err != nil {
			break
		}
	}
```
