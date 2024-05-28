#### docker 构建镜像更换国内apt源--Ubuntu和Debian

* Ubuntu
  ports.ubuntu.com是arm架构下apt的源，跨架构docker镜像构建时用得到
  
  ```
  RUN sed -i \
          -e "s/archive.ubuntu.com/mirrors.ustc.edu.cn/g" \
          -e "s/security.ubuntu.com/mirrors.ustc.edu.cn/g" \
          -e "s/ports.ubuntu.com/mirrors.ustc.edu.cn/g" \
          /etc/apt/sources.list
  ```

* Debian
  Debian在x86、amd64、arm64下apt源地址相同
  
  ```
  RUN sed -i \
      -e 's/deb.debian.org/mirrors.ustc.edu.cn/g' \
      -e 's/security.debian.org/mirrors.ustc.edu.cn/g' \
      /etc/apt/sources.list
  ```
