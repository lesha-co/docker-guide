https://hub.docker.com/_/nginx/

# basic docker

```sh
docker run --name web01 -d -p 8080:80 nginx
docker run --name web02 -d -p 8081:80 nginx
```

edit `/usr/share/nginx/html/index.html`

```sh
ubuntu@ubuntu-standard-2-8-20gb:~$ docker ps
CONTAINER ID   IMAGE                                    COMMAND                  CREATED         STATUS         PORTS                                                                                  NAMES
6931e499c1b7   nginx                                    "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8081->80/tcp, :::8081->80/tcp                                                  web02
21dec45bd67f   nginx                                    "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp                                                  web01
```

# Docker, but with volume mount

```
docker run --name web03 -d -p 8082:80 -v /home/ubuntu/dev/html:/usr/share/nginx/html nginx
                              host:container              host:container
```

# Docker, but let's build a container ourselves

```
docker build -t some-content-nginx .
docker run --name some-nginx -d -p 8083:80 some-content-nginx 
```


# Docker Network
```
ubuntu@ubuntu-standard-2-8-20gb:~$ docker container inspect web01

"NetworkSettings": {
    "Bridge": "",
    "SandboxID": "e2cb949868641a1b12893332679704a1611e2fd16404e5e20981a4563f1a18fd",
    "HairpinMode": false,
    "LinkLocalIPv6Address": "",
    "LinkLocalIPv6PrefixLen": 0,
    "Ports": {
        "80/tcp": [
            {
                "HostIp": "0.0.0.0",
                "HostPort": "8080"
            },
            {
                "HostIp": "::",
                "HostPort": "8080"
            }
        ]
    },
    
    "IPAddress": "172.17.0.2",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "MacAddress": "02:42:ac:11:00:02",
    "Networks": {
        "bridge": {
            "IPAMConfig": null,
            "Links": null,
            "Aliases": null,
            "NetworkID": "139b462654ef8a1c419ca379c341796c07a7b2c860c09506a79002649bde1511",
            "EndpointID": "d9c4ebd0b5b738b88c43496c729352b7e3324a1b64bebe32034a1f40c279f1dd",
            "Gateway": "172.17.0.1",
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "MacAddress": "02:42:ac:11:00:02",
            "DriverOpts": null
        }
    }
}
```
```
ubuntu@ubuntu-standard-2-8-20gb:~$ docker network ls
NETWORK ID     NAME          DRIVER    SCOPE
139b462654ef   bridge        bridge    local
e18ec2a9cd9e   dev_default   bridge    local
f3de98aa54ca   host          host      local
a6a23948dcf1   none          null      local
```

```
ubuntu@ubuntu-standard-2-8-20gb:~$ docker network inspect 139b462654ef
[
    {
        "Name": "bridge",
        "Id": "139b462654ef8a1c419ca379c341796c07a7b2c860c09506a79002649bde1511",
        "Created": "2022-08-22T11:19:36.577696819Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "21dec45bd67fc114ea550c8fb8ff0d8f7fc66c1c8a5f94c5d7fc3b80959328a2": {
                "Name": "web01",
                "EndpointID": "d9c4ebd0b5b738b88c43496c729352b7e3324a1b64bebe32034a1f40c279f1dd",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "6931e499c1b737dfdb09a425eeb8749f414f255386901765b7f2f3fc7c6d77bf": {
                "Name": "web02",
                "EndpointID": "4427306b02a1ffdbba593f9928d0a64724aa0c3f0f84382f1e84ef64b680bffc",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

```sh
ubuntu@ubuntu-standard-2-8-20gb:~$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         254.mcs.mail.ru 0.0.0.0         UG    0      0        0 ens3
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-e18ec2a9cd9e
192.168.0.0     0.0.0.0         255.255.0.0     U     0      0        0 wg0
213.219.212.0   0.0.0.0         255.255.252.0   U     0      0        0 ens3
```


| network        | netmask           | CIDR notation   |
| -------------- | ----------------- | --------------- |
| `172.0.0.0`    | `255.0.0.0`       | `172.0.0.0/8`   |
| `172.17.0.0`   | `255.255.0.0`     | `172.17.0.0/16` |
| `172.17.123.0` | `255.255.255.0`   | `172.17.0.0/24` |
| `172.17.0.2`   | `255.255.255.255` | `172.17.0.2/32` |
