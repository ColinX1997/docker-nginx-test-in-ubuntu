1. Prepare your environment

- Prepare your npm (nodejs package manager) environment - Follow https://nodejs.cn/download/
- Use follow cmd to check your env
  npm -v
  node -v
- Open VS Code or any other IDE, create one folder
- set image registry (npm config set registry https://xxxx/repository/npm-group/)
- Install Angular bootstrap, use:
  npm i -g @angular/cli
  Check the result through: ng --version

2. Create your first Angular app

- Init one default angular project, use (ng new project-name)
- run it (ng serve)

3. Create your docker file, ex:

```
# 使用Node.js镜像作为构建环境
FROM xxxx.cn/hub/node:18-alpine AS builder

# 设置工作目录
WORKDIR /app

# 复制package.json和package-lock.json
COPY package.json package-lock.json ./

# 指定npm源
RUN npm config set registry https://xxxx/repository/npm-group/

# 安装依赖
RUN npm install

# 复制项目文件
COPY . .

# 构建生产环境
RUN npm run build --production

# 使用Nginx作为服务运行环境
FROM harbor-ali.xxxxxx.cn/hub/nginx:alpine

RUN mkdir -p /dist && chmod 0777 /dist

# 复制自定义的 nginx 配置文件
COPY nginx.conf /etc/nginx/nginx.conf
RUN rm -f /etc/nginx/conf.d/default.conf

# 复制构建后的文件到Nginx目录
COPY --from=builder /app/dist/testxy /dist/nginx/html

# 曝露80端口
EXPOSE 80

# 启动Nginx
CMD ["nginx", "-g", "daemon off;"]

```

Then create new nginx file: nginx.conf

```
worker_processes  1;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    keepalive_timeout  65;

    server {
    listen 80;
    server_name localhost;

    location / {
        root   /dist/nginx/html;
        index  index.html;
        try_files $uri $uri/ /index.html;
    }
}

    include /etc/nginx/conf.d/*.conf;

}
```

in angular.json, pay attention to build params:

```
"build": {
          "builder": "@angular-devkit/build-angular:browser", #browser type is differnt from application type
          "options": {
            "aot": true,
            "outputPath": "dist/testxy",  #build output path, following WORKDIR in dockerfile
            "index": "src/index.html",    #entry point
            "main": "src/main.ts",
            "polyfills": ["zone.js"],
            "tsConfig": "tsconfig.app.json",
            "inlineStyleLanguage": "scss",
            "assets": [
              {
                "glob": "**/*",
                "input": "public"
              }
            ],
            "styles": ["src/styles.scss"],
            "scripts": []
          }
}
```

4. Try to build image in ubuntu env

- install ubuntu WSL, refer to: https://learn.microsoft.com/zh-cn/windows/wsl/install-manual
- after installation finished, login to your virtual env
- use apt cli to install docker service in ubuntu, refer to: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04
- start docker service， use cmd:  
  `sudo service docker restart`
- cd to local project, usually in mnt folder, for ex:/mnt/c_workspace/dev/testxy
- build the image use cmd:
  `docker build -t testxy`
- run the image directly to check the build result, use cmd:
  ` docker run -p 10086:80 testxy`

5. Frequently used docker cmds

- docker build -t project-name .
- docker run -p 10086:10086 project-name #指的是将容器暴露的端口映射到宿主机的某个端口（宿主机：容器）
- docker images
- docker rmi image-id
- docker rm -f container-id
- docker pull image-address
- docker exec -it 45029881c98b /bin/sh #enter into one container, used to check the details
