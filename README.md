#Sideproject Dynbolt

**This is just a sideproject!**

The objective of this sideproject is, to build a scalable, (highly) available, 
distributed, DynamoDB API compatible, key-value store.

- Request Coordinator: Custom or sth. like [Go-Micro](https://github.com/micro/go-micro)
- Membership & Failure Detection: [Serf](https://github.com/hashicorp/serf)
- Local Persistence Engine: [Bolt](https://github.com/boltdb/bolt)

##Links / Resources

- [Dynamo Paper](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)
- [Hashring implementation](https://github.com/stathat/consistent)

Hashring sample

```go
package main

import (
	"fmt"
	"log"

	"stathat.com/c/consistent"
)

func fakeuuid(i int) string {
	uuid := fmt.Sprintf("%v-%X-%x", i, i, i)
	return uuid
}

func main() {
	c := consistent.New()
	c.Add("nodeA")
	c.Add("nodeB")

	hashes := []string{}
	for i := 0; i < 400; i++ {
		hashes = append(hashes, fakeuuid(i))
	}

	count := map[string]int{"nodeA": 0, "nodeB": 0}
	for _, u := range hashes {
		node, err := c.Get(u)
		if err != nil {
			log.Fatal(err)
		}
		count[node] += 1
	}
	fmt.Printf("distribution: %v\n", count)

	c.Add("nodeC")
	count = map[string]int{"nodeA": 0, "nodeB": 0, "nodeC": 0}
	for _, u := range hashes {
		node, err := c.Get(u)
		if err != nil {
			log.Fatal(err)
		}
		count[node] += 1
	}
	fmt.Printf("distribution: %v\n", count)

	c.Remove("nodeC")
	count = map[string]int{"nodeA": 0, "nodeB": 0}
	for _, u := range hashes {
		node, err := c.Get(u)
		if err != nil {
			log.Fatal(err)
		}
		count[node] += 1
	}
	fmt.Printf("distribution: %v\n", count)
}
```
