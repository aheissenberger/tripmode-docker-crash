# Docker Compose crash MacOS Network with TripMode

Problem exist only with Docker Compose and a Dockerfile which pulls an image which has not been pulled in the past.

## Tripmode

Version: 3.1.4

```
systemextensionsctl list
1 extension(s)
--- com.apple.system_extension.network_extension
enabled	active	teamID	bundleID (version)	name	[state]
*	*	78UFGP42EU	ch.tripmode.TripMode.FilterExtension (3.1.0/1304)	FilterExtension	[activated enabled]
```

## docker with BuildKit

Docker Engine Setup:
```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true
  }
}
```

docker version: 20.10.17, build 100c701
Docker Compose version: v2.10.2

you can manually activate **buildkit** with `DOCKER_BUILDKIT=1` - should be activated by default in all current versions!

with **BuildKit**:

`DOCKER_BUILDKIT=1 docker compose build --no-cache`

```
docker compose build           
[+] Building 30.7s (3/3) FINISHED                                                                     
 => [internal] load build definition from Dockerfile                                             0.0s
 => => transferring dockerfile: 60B                                                              0.0s
 => [internal] load .dockerignore                                                                0.0s
 => => transferring context: 2B                                                                  0.0s
 => ERROR [internal] load metadata for docker.io/library/hello-world:linux                      30.6s
------
 > [internal] load metadata for docker.io/library/hello-world:linux:
------
failed to solve: rpc error: code = Unknown desc = failed to solve with frontend dockerfile.v0: failed to create LLB definition: failed to authorize: rpc error: code = Unknown desc = failed to fetch anonymous token: Get "https://auth.docker.io/token?scope=repository%3Alibrary%2Fhello-world%3Apull&service=registry.docker.io": dial tcp: lookup auth.docker.io: i/o timeout
```

> The whole system has lost any kind of network access!

To repeat allway clear the cache: `docker compose down -v --rmi all`


with **BuildKit deactivated**:
`DOCKER_BUILDKIT=0 docker compose build --no-cache`

```
Sending build context to Docker daemon  1.567kB
Step 1/1 : FROM hello-world
latest: Pulling from library/hello-world
7050e35b49f5: Pull complete 
Digest: sha256:18a657d0cc1c7d0678a3fbea8b7eb4918bba25968d3e1b0adebfa71caddbc346
Status: Downloaded newer image for hello-world:latest
 ---> 46331d942d63
Successfully built 46331d942d63
Successfully tagged tripmode-docker-crash-app:latest

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
[+] Running 2/0
 ⠿ Network tripmode-docker-crash_default  Created                                                0.0s
 ⠿ Container tripmode-docker-crash-app-1  Created                                                0.0s
Attaching to tripmode-docker-crash-app-1
tripmode-docker-crash-app-1  | 
tripmode-docker-crash-app-1  | Hello from Docker!
tripmode-docker-crash-app-1  | This message shows that your installation appears to be working correctly.
tripmode-docker-crash-app-1  | 
tripmode-docker-crash-app-1  | To generate this message, Docker took the following steps:
tripmode-docker-crash-app-1  |  1. The Docker client contacted the Docker daemon.
tripmode-docker-crash-app-1  |  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
tripmode-docker-crash-app-1  |     (arm64v8)
tripmode-docker-crash-app-1  |  3. The Docker daemon created a new container from that image which runs the
tripmode-docker-crash-app-1  |     executable that produces the output you are currently reading.
tripmode-docker-crash-app-1  |  4. The Docker daemon streamed that output to the Docker client, which sent it
tripmode-docker-crash-app-1  |     to your terminal.
tripmode-docker-crash-app-1  | 
tripmode-docker-crash-app-1  | To try something more ambitious, you can run an Ubuntu container with:
tripmode-docker-crash-app-1  |  $ docker run -it ubuntu bash
tripmode-docker-crash-app-1  | 
tripmode-docker-crash-app-1  | Share images, automate workflows, and more with a free Docker ID:
tripmode-docker-crash-app-1  |  https://hub.docker.com/
tripmode-docker-crash-app-1  | 
tripmode-docker-crash-app-1  | For more examples and ideas, visit:
tripmode-docker-crash-app-1  |  https://docs.docker.com/get-started/
tripmode-docker-crash-app-1  | 
tripmode-docker-crash-app-1 exited with code 0
```

