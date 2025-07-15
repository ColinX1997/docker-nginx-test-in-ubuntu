# 使用Node.js镜像作为构建环境
FROM xxxxx/hub/node:18-alpine AS builder

# 设置工作目录
WORKDIR /app

# 复制package.json和package-lock.json
COPY package.json package-lock.json ./

# 指定npm源
RUN npm config set registry xxxx

# 安装依赖
RUN npm install

# 复制项目文件
COPY . .

# 构建生产环境
RUN npm run build --production

# 使用Nginx作为服务运行环境
FROM xxxxx/hub/nginx:alpine

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
