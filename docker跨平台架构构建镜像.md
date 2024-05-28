#### docker跨平台架构构建镜像

###### 官方文档

[Multi-platform images | Docker Docs](https://docs.docker.com/build/building/multi-platform/)

[docker buildx build | Docker Documentation](https://docs.docker.com/engine/reference/commandline/buildx_build/)

###### docker buildx 构建本地镜像时，多个本地镜像并不能合成一个，也就是本地镜像名字不能相同

我们构建一个名为nicaicai的镜像，目标平台架构linux/aarch64

构建完发现镜像ID为：12e4f0289d3e ，镜像tag为：nicaicai:latest

然后我们再构建一个名为nicaicai的镜像，这次目标平台架构改为linux/amd64

构建完成发现镜像ID为：12e4f0289d3e ，镜像tag为：nicaicai:latest

再看之前的nicaicai:latest，tag已经被抢占了，已经为none了

```
root@192-168-200-252:/data/debian-based-images# docker buildx build -t nicaicai --platform=linux/aarch64 -o type=docker -f debian10.Dockerfile . 
[+] Building 25.4s (13/13) FINISHED                                                                                                                                                                           
 => [internal] load .dockerignore                                                                                                                                                                        0.3s
 => => transferring context: 2B                                                                                                                                                                          0.0s
 => [internal] load build definition from debian10.Dockerfile                                                                                                                                            0.3s
 => => transferring dockerfile: 1.17kB                                                                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/debian:10                                                                                                                                             3.9s
 => [1/7] FROM docker.io/library/debian:10@sha256:7cd85d3d51a435062010581f14c5e3f9428388ac7234cc9a1e23dd14d7e4e914                                                                                       0.1s
 => => resolve docker.io/library/debian:10@sha256:7cd85d3d51a435062010581f14c5e3f9428388ac7234cc9a1e23dd14d7e4e914                                                                                       0.1s
 => [internal] load build context                                                                                                                                                                        0.0s
 => => transferring context: 38B                                                                                                                                                                         0.0s
 => CACHED [2/7] WORKDIR /opt/odoo                                                                                                                                                                       0.0s
 => CACHED [3/7] COPY ./requirements.txt ./requirements.txt                                                                                                                                              0.0s
 => CACHED [4/7] RUN sed -i  -e 's/deb.debian.org/mirrors.ustc.edu.cn/g'  -e 's/security.debian.org/mirrors.ustc.edu.cn/g'  /etc/apt/sources.list                                                        0.0s
 => CACHED [5/7] RUN apt-get update -y && apt-get install -y tzdata  && ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  && echo Asia/Shanghai > /etc/timezone  && dpkg-reconfigure --frontend   0.0s
 => CACHED [6/7] RUN apt-get update -y && apt-get install -y python3-pip python-dev  libxml2-dev libxslt1-dev libpq-dev libsasl2-dev  libldap2-dev libssl-dev libffi-dev libjpeg-dev  libz-dev libreadl  0.0s
 => CACHED [7/7] RUN pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple  && pip3 install pip==22.0.4  && pip3 install setuptools==57.5.0  && pip3 --default-timeout=100 install   0.0s
 => exporting to docker image format                                                                                                                                                                    20.9s
 => => exporting layers                                                                                                                                                                                  0.0s
 => => exporting manifest sha256:68b5a8d87dd66ed2a263008b73c12c1c69e9c58ca9813058f485225bf34be0ac                                                                                                        0.0s
 => => exporting config sha256:12e4f0289d3e92dd2a9f2a10101f188fee7ba55d6fbaa29f9f7c29747f9fa144                                                                                                          0.0s
 => => sending tarball                                                                                                                                                                                  20.9s
 => importing to docker                                                                                                                                                                                  0.1s
root@192-168-200-252:/data/debian-based-images# docker images 
REPOSITORY                                  TAG                IMAGE ID       CREATED         SIZE
zhangdeshuai                                latest             423773fe9b42   18 hours ago    1.45GB
harbor.insuite.cn/library/insuite_runtime   ubuntu18.aarch64   9cce33952d80   20 hours ago    1.3GB
ubuntu1804                                  python3.7          26b776a9674c   20 hours ago    258MB
eipwork/kuboard                             v3                 a1bfde5ba76f   47 hours ago    458MB
eipwork/kuboard                             v3.5.2.5           a1bfde5ba76f   47 hours ago    458MB
harbor.insuite.cn/kuboard/kuboard           v3                 a1bfde5ba76f   47 hours ago    458MB
harbor.insuite.cn/kuboard/kuboard           v3.5.2.5           a1bfde5ba76f   47 hours ago    458MB
nicaicai                                    latest             12e4f0289d3e   3 days ago      1.35GB
ubuntu                                      18.04              f9a80a55f492   2 months ago    63.2MB
moby/buildkit                               buildx-stable-1    152ba3e57b49   3 months ago    168MB
eipwork/kuboard-agent                       v3                 02a2e9bda11a   7 months ago    101MB
harbor.insuite.cn/kuboard/kuboard-agent     v3                 02a2e9bda11a   7 months ago    101MB
eipwork/etcd-host                           3.4.16-2           d6066d124f66   14 months ago   119MB
harbor.insuite.cn/kuboard/etcd-host         3.4.16-2           d6066d124f66   14 months ago   119MB
questdb/questdb                             6.0.5              832e7fbaab9b   23 months ago   96.1MB
harbor.insuite.cn/kuboard/questdb           6.0.5              832e7fbaab9b   23 months ago   96.1MB
questdb/questdb                             6.0.4              2fbb6abb72a9   23 months ago   96MB
harbor.insuite.cn/kuboard/questdb           6.0.4              2fbb6abb72a9   23 months ago   96MB
eipwork/etcd-host                           3.4.16-1           28ad0cc2f0f9   2 years ago     119MB
harbor.insuite.cn/kuboard/etcd-host         3.4.16-1           28ad0cc2f0f9   2 years ago     119MB
root@192-168-200-252:/data/debian-based-images# docker buildx build -t nicaicai --platform=linux/amd64 -o type=docker -f debian10.Dockerfile . 
[+] Building 18.1s (13/13) FINISHED                                                                                                                                                                           
 => [internal] load build definition from debian10.Dockerfile                                                                                                                                            0.0s
 => => transferring dockerfile: 1.17kB                                                                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                                                                        0.0s
 => => transferring context: 2B                                                                                                                                                                          0.0s
 => [internal] load metadata for docker.io/library/debian:10                                                                                                                                             1.9s
 => [1/7] FROM docker.io/library/debian:10@sha256:7cd85d3d51a435062010581f14c5e3f9428388ac7234cc9a1e23dd14d7e4e914                                                                                       0.0s
 => => resolve docker.io/library/debian:10@sha256:7cd85d3d51a435062010581f14c5e3f9428388ac7234cc9a1e23dd14d7e4e914                                                                                       0.0s
 => [internal] load build context                                                                                                                                                                        0.0s
 => => transferring context: 38B                                                                                                                                                                         0.0s
 => CACHED [2/7] WORKDIR /opt/odoo                                                                                                                                                                       0.0s
 => CACHED [3/7] COPY ./requirements.txt ./requirements.txt                                                                                                                                              0.0s
 => CACHED [4/7] RUN sed -i  -e 's/deb.debian.org/mirrors.ustc.edu.cn/g'  -e 's/security.debian.org/mirrors.ustc.edu.cn/g'  /etc/apt/sources.list                                                        0.0s
 => CACHED [5/7] RUN apt-get update -y && apt-get install -y tzdata  && ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  && echo Asia/Shanghai > /etc/timezone  && dpkg-reconfigure --frontend   0.0s
 => CACHED [6/7] RUN apt-get update -y && apt-get install -y python3-pip python-dev  libxml2-dev libxslt1-dev libpq-dev libsasl2-dev  libldap2-dev libssl-dev libffi-dev libjpeg-dev  libz-dev libreadl  0.0s
 => CACHED [7/7] RUN pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple  && pip3 install pip==22.0.4  && pip3 install setuptools==57.5.0  && pip3 --default-timeout=100 install   0.0s
 => exporting to docker image format                                                                                                                                                                    15.9s
 => => exporting layers                                                                                                                                                                                  0.0s
 => => exporting manifest sha256:eed4a3a3b83601af4964a5d00ea8a5e7c5e8b891976440dafff644c6071ac0cf                                                                                                        0.0s
 => => exporting config sha256:423773fe9b42bf50db5f575969b8f39c978cf1ba9d0db55749f2b7123c4f43ed                                                                                                          0.0s
 => => sending tarball                                                                                                                                                                                  15.9s
 => importing to docker                                                                                                                                                                                  0.1s
root@192-168-200-252:/data/debian-based-images# docker images 
REPOSITORY                                  TAG                IMAGE ID       CREATED         SIZE
nicaicai                                    latest             423773fe9b42   18 hours ago    1.45GB
zhangdeshuai                                latest             423773fe9b42   18 hours ago    1.45GB
harbor.insuite.cn/library/insuite_runtime   ubuntu18.aarch64   9cce33952d80   20 hours ago    1.3GB
ubuntu1804                                  python3.7          26b776a9674c   20 hours ago    258MB
eipwork/kuboard                             v3                 a1bfde5ba76f   47 hours ago    458MB
eipwork/kuboard                             v3.5.2.5           a1bfde5ba76f   47 hours ago    458MB
harbor.insuite.cn/kuboard/kuboard           v3                 a1bfde5ba76f   47 hours ago    458MB
harbor.insuite.cn/kuboard/kuboard           v3.5.2.5           a1bfde5ba76f   47 hours ago    458MB
<none>                                      <none>             12e4f0289d3e   3 days ago      1.35GB
ubuntu                                      18.04              f9a80a55f492   2 months ago    63.2MB
moby/buildkit                               buildx-stable-1    152ba3e57b49   3 months ago    168MB
eipwork/kuboard-agent                       v3                 02a2e9bda11a   7 months ago    101MB
harbor.insuite.cn/kuboard/kuboard-agent     v3                 02a2e9bda11a   7 months ago    101MB
eipwork/etcd-host                           3.4.16-2           d6066d124f66   14 months ago   119MB
harbor.insuite.cn/kuboard/etcd-host         3.4.16-2           d6066d124f66   14 months ago   119MB
questdb/questdb                             6.0.5              832e7fbaab9b   23 months ago   96.1MB
harbor.insuite.cn/kuboard/questdb           6.0.5              832e7fbaab9b   23 months ago   96.1MB
harbor.insuite.cn/kuboard/questdb           6.0.4              2fbb6abb72a9   23 months ago   96MB
questdb/questdb                             6.0.4              2fbb6abb72a9   23 months ago   96MB
eipwork/etcd-host                           3.4.16-1           28ad0cc2f0f9   2 years ago     119MB
harbor.insuite.cn/kuboard/etcd-host         3.4.16-1           28ad0cc2f0f9   2 years ago     119MB
root@192-168-200-252:/data/debian-based-images# 
    
```

###### docker buildx 构建本地镜像时，无法一次构建多个平台架构的镜像，但是构建远程镜像是（也就是不指定-o type=docker）时，是可以一次构建多个平台架构的镜像的

```
root@192-168-200-252:/data/debian-based-images# docker buildx build -t nicaicai --platform=linux/aarch64,linux/amd64 -o type=docker -f debian10.Dockerfile . 
[+] Building 0.0s (0/0)                                                                                                                                                                                       
error: docker exporter does not currently support exporting manifest lists
```

```
root@192-168-200-252:/data/python3-based-images# docker buildx build -t remoteimage --platform=linux/aarch64,linux/amd64 . 
WARN[0000] No output specified for docker-container driver. Build result will only remain in the build cache. To push result image into registry use --push or to load image into docker use --load 
[+] Building 3.5s (6/10)                                                                                                                                                                                      
 => [internal] load .dockerignore                                                                                                                                                                        0.0s
 => => transferring context: 2B                                                                                                                                                                          0.0s
 => [internal] load build definition from Dockerfile                                                                                                                                                     0.0s
 => => transferring dockerfile: 414B                                                                                                                                                                     0.0s
 => [linux/amd64 internal] load metadata for docker.io/library/ubuntu:18.04                                                                                                                              3.3s
 => [linux/arm64 internal] load metadata for docker.io/library/ubuntu:18.04                                                                                                                              3.3s
 => [linux/amd64 1/3] FROM docker.io/library/ubuntu:18.04@sha256:152dc042452c496007f07ca9127571cb9c29697f42acbfad72324b2bb2e43c98                                                                        0.1s
 => => resolve docker.io/library/ubuntu:18.04@sha256:152dc042452c496007f07ca9127571cb9c29697f42acbfad72324b2bb2e43c98                                                                                    0.0s
 => [linux/arm64 1/3] FROM docker.io/library/ubuntu:18.04@sha256:152dc042452c496007f07ca9127571cb9c29697f42acbfad72324b2bb2e43c98                                                                        0.1s
 => => resolve docker.io/library/ubuntu:18.04@sha256:152dc042452c496007f07ca9127571cb9c29697f42acbfad72324b2bb2e43c98                                                                                    0.0s
```

###### docker buildx 构建本地镜像时，处理python的依赖文件requirements的依赖包版本时，由于amd64架构里的依赖包版本不一定有对应的aarch64架构版本，所以需要调整requirements中的依赖包版本，或者在requirements里添加架构条件，以便兼容多个架构

```
root@88129905d7df:/opt/odoo# grep aarch64 requirements.txt 
pandas==1.1.5; platform_system == "Linux" and platform_machine == "aarch64"
PyMuPDF==1.19.0 ; python_version < '3.7' and platform_system == "Linux" and platform_machine == "aarch64"
```

###### docker buildx 一次构建多平台镜像，且推送到私有仓库，而不是推送到docker hub仓库

关键点：-t的时候就指定私有仓库地址；--platform的时候指定多个目标架构，中间用逗号分开；-o type为registry 而不是之前的docker，docker会导出成本地镜像，而registry会推送到远程仓库，当然需要有推送的权限。

```
root@192-168-200-252:/data/debian-based-images# docker buildx build -t harbor.insuite.cn/library/insuite_runtime:v3.1 --platform=linux/aarch64,linux/amd64 -o type=registry -f debian10.Dockerfile . [+] Building 4.6s (21/21) FINISHED                                                                                                                                                                            
 => [internal] load build definition from debian10.Dockerfile                                                                                                                                            0.0s
 => => transferring dockerfile: 1.17kB                                                                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                                                                        0.0s
 => => transferring context: 2B                                                                                                                                                                          0.0s
 => [linux/arm64 internal] load metadata for docker.io/library/debian:10                                                                                                                                 3.0s
 => [linux/amd64 internal] load metadata for docker.io/library/debian:10                                                                                                                                 3.2s
 => [internal] load build context                                                                                                                                                                        0.0s
 => => transferring context: 38B                                                                                                                                                                         0.0s
 => [linux/amd64 1/7] FROM docker.io/library/debian:10@sha256:7cd85d3d51a435062010581f14c5e3f9428388ac7234cc9a1e23dd14d7e4e914                                                                           0.1s
 => => resolve docker.io/library/debian:10@sha256:7cd85d3d51a435062010581f14c5e3f9428388ac7234cc9a1e23dd14d7e4e914                                                                                       0.1s
 => [linux/arm64 1/7] FROM docker.io/library/debian:10@sha256:7cd85d3d51a435062010581f14c5e3f9428388ac7234cc9a1e23dd14d7e4e914                                                                           0.1s
 => => resolve docker.io/library/debian:10@sha256:7cd85d3d51a435062010581f14c5e3f9428388ac7234cc9a1e23dd14d7e4e914                                                                                       0.1s
 => CACHED [linux/amd64 2/7] WORKDIR /opt/odoo                                                                                                                                                           0.0s
 => CACHED [linux/amd64 3/7] COPY ./requirements.txt ./requirements.txt                                                                                                                                  0.0s
 => CACHED [linux/amd64 4/7] RUN sed -i  -e 's/deb.debian.org/mirrors.ustc.edu.cn/g'  -e 's/security.debian.org/mirrors.ustc.edu.cn/g'  /etc/apt/sources.list                                            0.0s
 => CACHED [linux/amd64 5/7] RUN apt-get update -y && apt-get install -y tzdata  && ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  && echo Asia/Shanghai > /etc/timezone  && dpkg-reconfigure  0.0s
 => CACHED [linux/amd64 6/7] RUN apt-get update -y && apt-get install -y python3-pip python-dev  libxml2-dev libxslt1-dev libpq-dev libsasl2-dev  libldap2-dev libssl-dev libffi-dev libjpeg-dev  libz-  0.0s
 => CACHED [linux/amd64 7/7] RUN pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple  && pip3 install pip==22.0.4  && pip3 install setuptools==57.5.0  && pip3 --default-timeout=  0.0s
 => CACHED [linux/arm64 2/7] WORKDIR /opt/odoo                                                                                                                                                           0.0s
 => CACHED [linux/arm64 3/7] COPY ./requirements.txt ./requirements.txt                                                                                                                                  0.0s
 => CACHED [linux/arm64 4/7] RUN sed -i  -e 's/deb.debian.org/mirrors.ustc.edu.cn/g'  -e 's/security.debian.org/mirrors.ustc.edu.cn/g'  /etc/apt/sources.list                                            0.0s
 => CACHED [linux/arm64 5/7] RUN apt-get update -y && apt-get install -y tzdata  && ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  && echo Asia/Shanghai > /etc/timezone  && dpkg-reconfigure  0.0s
 => CACHED [linux/arm64 6/7] RUN apt-get update -y && apt-get install -y python3-pip python-dev  libxml2-dev libxslt1-dev libpq-dev libsasl2-dev  libldap2-dev libssl-dev libffi-dev libjpeg-dev  libz-  0.0s
 => CACHED [linux/arm64 7/7] RUN pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple  && pip3 install pip==22.0.4  && pip3 install setuptools==57.5.0  && pip3 --default-timeout=  0.0s
 => exporting to image                                                                                                                                                                                   1.1s
 => => exporting layers                                                                                                                                                                                  0.0s
 => => exporting manifest sha256:68b5a8d87dd66ed2a263008b73c12c1c69e9c58ca9813058f485225bf34be0ac                                                                                                        0.0s
 => => exporting config sha256:12e4f0289d3e92dd2a9f2a10101f188fee7ba55d6fbaa29f9f7c29747f9fa144                                                                                                          0.0s
 => => exporting manifest sha256:eed4a3a3b83601af4964a5d00ea8a5e7c5e8b891976440dafff644c6071ac0cf                                                                                                        0.0s
 => => exporting config sha256:423773fe9b42bf50db5f575969b8f39c978cf1ba9d0db55749f2b7123c4f43ed                                                                                                          0.0s
 => => exporting manifest list sha256:b449f83a198058c1cd1ba01f81ee55c99b961e573bfe9071bdb0cbfe616bc95d                                                                                                   0.0s
 => => pushing layers                                                                                                                                                                                    0.7s
 => => pushing manifest for harbor.insuite.cn/library/insuite_runtime:v3.1@sha256:b449f83a198058c1cd1ba01f81ee55c99b961e573bfe9071bdb0cbfe616bc95d                                                       0.3s
 => [auth] library/insuite_runtime:pull,push token for harbor.insuite.cn                                                                                                                                 0.0s
You have new mail in /var/mail/root
root@192-168-200-252:/data/debian-based-images# docker images 
```
