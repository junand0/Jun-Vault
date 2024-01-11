[ref1](https://github.com/saba-ja/vivado_docker)
[ref2](https://github.com/carlesfernandez/docker-petalinux)
[ref3](https://support.xilinx.com/s/question/0D52E00006liykGSAQ/segfaults-building-projects-with-vivado-20211-in-docker?language=en_US)
[ref4](https://github.com/sinitame/xilinx-docker-mac)

### Chat GPT
![](https://i.imgur.com/r6Q8M2P.png)
```
docker pull <image name>:<tag>
```
Replace `<image name>` with the name of the Vivado Docker image and `<tag>` with the version or tag you want to use.

![](https://i.imgur.com/MG9HHSI.png)
```
docker run --name <container name> -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -it <image name>:<tag>
```

![](https://i.imgur.com/zk9xY7L.png)
