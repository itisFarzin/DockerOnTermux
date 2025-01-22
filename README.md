# Docker On Termux
_This note is more or less a personal note_

You have to follow [FreddieOliveira's guide](https://gist.github.com/FreddieOliveira/efe850df7ff3951cb62d74bd770dce27) to set up Docker on Termux, This includes:
- Run [check-config.sh](https://raw.githubusercontent.com/moby/moby/master/contrib/check-config.sh) to identify missing flags and enable them in your device Kernel defconfig
- [Patching the Kernel](https://gist.github.com/FreddieOliveira/efe850df7ff3951cb62d74bd770dce27#41-kernel-patches)

After that, you will hopefully be able to run Docker on Termux, but (for me) there was a problem: *network driver*. The bridge network driver crashed my phone, and I struggled to figure out what was happening. After switching to the host network driver, Docker worked successfully, and I was able to run some containers.

So I wrote a simple wrapper for Docker to use host network driver for containers and run Docker commands with sudo:

```bash
function docker {
    if [ "$1" = "run" ]; then
        shift
        command sudo docker run --network=host "$@"
    elif [ "$1" = "create" ]; then
        shift
        command sudo docker create --network=host "$@"
    elif [ "$1" = "build" ]; then
        shift
        command sudo docker build --network=host "$@"
    elif [ "$1" = "compose" ]; then
        shift
        command sudo DOCKER_HOST="unix://$PREFIX/var/run/docker.sock" docker compose "$@"
    else
        command sudo docker "$@"
    fi
}
```

But something isn't right. For example, after Docker has been running if I restart the phone, running Docker again still causes Android to crash, it seems like it wants to start/create the bridge interface.
After some investigation I found out if I remove the `$PREFIX/lib/docker/network` folder before running Docker, it fixes the issue.

So I added this line to the `$PREFIX/var/service/dockerd/run` file:

```bash
rm -rf /data/data/com.termux/files/usr/lib/docker/network
```

I hope you find this note helpful.