# Day 03
## Introduction
Using cp, curl, jq

## Prerequisites
* Done [Day 2](../Day-02/readme.md)
* Docker

## Challenge

### Challenge 1

Copy the Dockerfile from [Day 2](../Day-02/readme.md) with cp

```sh
cp ../Day-02/Dockerfile .
cp ../Day-02/main.go .
```

### Challenge 2

Change `main.go` so we have a json file to request

```sh
vi main.go
```

Change the function handler to the following content

```golang
func handler(w http.ResponseWriter, r *http.Request) {
    
    json := `[
        {
            "Path": "interfaces/interface[name=Ethernet1]/subinterfaces",
            "values": {
            "interfaces/interface/subinterfaces": {
                "openconfig-interfaces:subinterface": [
                {
                    "index": 0,
                    "state": {
                    "ifindex": 1
                    }
                }
                ]
            }
            }
        },
        {
            "Path": "interfaces/interface[name=Management0]/subinterfaces",
            "values": {
            "interfaces/interface/subinterfaces": {
                "openconfig-interfaces:subinterface": [
                {
                    "index": 0,
                    "openconfig-if-ip:ipv4": {
                    "addresses": {
                        "address": [
                        {
                            "config": {
                            "ip": "172.20.20.3",
                            "prefix-length": 24
                            },
                            "ip": "172.20.20.3",
                            "state": {
                            "ip": "172.20.20.3",
                            "origin": "STATIC",
                            "prefix-length": 24
                            }
                        }
                        ]
                    },
                    "neighbors": {
                        "neighbor": [
                        {
                            "config": {
                            "ip": "172.20.20.1"
                            },
                            "ip": "172.20.20.1",
                            "state": {
                            "ip": "172.20.20.1",
                            "link-layer-address": "02:42:b8:42:ba:16",
                            "origin": "DYNAMIC"
                            }
                        }
                        ]
                    }
                    },
                    "openconfig-if-ip:ipv6": {
                    "addresses": {
                        "address": [
                        {
                            "config": {
                            "ip": "3fff:172:20:20::3",
                            "prefix-length": 64
                            },
                            "ip": "3fff:172:20:20::3",
                            "state": {
                            "ip": "3fff:172:20:20::3",
                            "origin": "STATIC",
                            "prefix-length": 64,
                            "status": "PREFERRED"
                            }
                        },
                        {
                            "config": {
                            "ip": "fe80::42:acff:fe14:1403",
                            "prefix-length": 64
                            },
                            "ip": "fe80::42:acff:fe14:1403",
                            "state": {
                            "ip": "fe80::42:acff:fe14:1403",
                            "origin": "LINK_LAYER",
                            "prefix-length": 64,
                            "type": "LINK_LOCAL_UNICAST"
                            }
                        }
                        ]
                    },
                    "config": {},
                    "neighbors": {
                        "neighbor": [
                        {
                            "config": {
                            "ip": "3fff:172:20:20::1"
                            },
                            "ip": "3fff:172:20:20::1",
                            "state": {
                            "ip": "3fff:172:20:20::1",
                            "link-layer-address": "02:42:b8:42:ba:16",
                            "neighbor-state": "REACHABLE",
                            "origin": "DYNAMIC"
                            }
                        },
                        {
                            "config": {
                            "ip": "fe80::1"
                            },
                            "ip": "fe80::1",
                            "state": {
                            "ip": "fe80::1",
                            "link-layer-address": "02:42:b8:42:ba:16",
                            "neighbor-state": "REACHABLE",
                            "origin": "DYNAMIC"
                            }
                        }
                        ]
                    }
                    }
                }
                ]
            }
            }
        }
    ]`

    fmt.Fprintf(w, "%s", json)

}
```
</br>
Start the Server again

```sh
sudo docker build -t json-server . 
docker run -d -p 8080:8080 json-server 
```

### Challenge 3
Now get the JSON data from the server
```sh
curl localhost:8080
```

Next format the JSON data, so it looks presentable with jq
```sh
curl localhost:8080 | jq
```

### Challenge 4
At last get all of the path from the JSON data or if you want it harder get the interface name from the path.

<!--  curl -s localhost:8080 | jq '.[].Path' -->
<!-- curl -s localhost:8080 | jq -r '.[].Path | match("name=([^,]+)").string | match("[^=]+$").string' -->


## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%203%20of%20the%20NetAuto%20Bootcamp%20on%20Linux!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain