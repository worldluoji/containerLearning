FROM golang:1.17.2

ENV GO111MODULE=on \
    GOPROXY=https://goproxy.cn,direct \
    GOSUMDB=off

#指定当前目录为/app    
WORKDIR /app

#将本机当前目录文件拷贝到容器/app目录
ADD . /app

RUN go build http_server.go

#可选，查看容器中当前目录是否已经有了打包出来的文件
#RUN ls

#告诉docker容器暴露端口，在容器启动的时候，需要通过-p 做端口映射
EXPOSE 8088

#配置容器启动后，执行什么命令
ENTRYPOINT ["./http_server"]
