---
layout: post
title:  "Unix Sockets in Golang"
date:   2016-03-20 11:00:00
---

Last week I had to build an application in Go that will have a local client (in the same machine) connecting to it and passing data.
Using an unix socket was the easiest way to make it work.


### Server:

```golang
package main

import "net"
import "log"
import "fmt"

func dataHandler(c net.Conn) {

	buf := make([]byte, 512)
	nr, err := c.Read(buf)

	if err != nil {
		return
	}

	data := string(buf[0:nr])
        fmt.Println(data)
}

func main() {
	l, err := net.Listen("unix", "/tmp/example.sock")
	if err != nil {
		log.Fatal(err)
		return
	}

	for {
		fd, err := l.Accept()
		if err != nil {
			log.Fatal(err)
			return
		}
		dataHandler(fd)
		fd.Close()
	}
}

```

### Client:

```golang
package main

import (
	"fmt"
	"net"
	"log"
)

func main() {
	c, err := net.Dial("unix", "/tmp/example.sock")

	if err != nil {
		panic(err)
	}
	_, err = c.Write([]byte(arg))

	if err != nil {
		log.Println(err)
	}
}
```
