# Dockerized JeecgBoot

## 启动

1）获取代码

下载 JeecgBoot 代码到 `app` 目录，执行：

    git clone https://github.com/jeecgboot/jeecg-boot app

切换至 `v2.4.6` 版本：

    cd app
    git checkout v2.4.6

2）Maven 仓库设置

在 `_home/.m2` 目录下，参考 `settings.xml.dist` 创建 `settings.xml`（可直接拷贝改名）。

官方说明：[Maven私服设置 · JeecgBoot 开发文档 · 看云](http://doc.jeecg.com/2043876)

3）后端编译和安装

进入 `backend` 容器（OpenJDK + Maven），执行：

    docker-compose run --rm backend bash
    mvn clean install

4）初始化数据库

执行：

    docker-compose up -d mysql
    # 等待 MySQL 完成初始化
    docker exec -i docker-jeecg_mysql_1 mysql -u root -proot < app/jeecg-boot/db/jeecgboot-mysql-5.7.sql

5）调整后端配置

修改 `jeecg-boot-module-system/src/main/resources/application-dev.yml` 文件，

将 `server.datasource.dynamic.datasource.url` 修改为：

    jdbc:mysql://mysql:3306/jeecg-boot?characterEncoding=UTF-8&useUnicode=true&useSSL=false&tinyInt1isBit=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai

将 `server.redis.host` 修改为 `redis`。

将 `server.redis.password` 修改为 `NOT_SAFE`。

6）启动后端

执行：

    docker-compose up -d backend

7）安装前端依赖

在 `app/ant-design-vue-jeecg` 目录里执行：

    docker run --rm -v $(pwd):/project -w /project -u $(id -u):$(id -g) node:16-bullseye-slim npm ci --registry=https://registry.npm.taobao.org

8）调整前端配置

修改 `ant-design-vue-jeecg/.env.development`：

    -VUE_APP_API_BASE_URL=http://localhost:8080/jeecg-boot
    +VUE_APP_API_BASE_URL=http://jeecg-backend.example.com/jeecg-boot

修改 `ant-design-vue-jeecg/vue.config.js`，配置 Webpack DevServer：

```
module.exports = {
  chainWebpack: (config) => {
    // ...

    // 忽略 node_modules 路径下的文件修改
    config.devServer.watchOptions({
      ignored: /node_modules/
    })
  },

  devServer: {
    proxy: {
      '/jeecg-boot': {
        target: 'http://backend:8080', //请求本地 需要jeecg-boot后台项目
      },
    },
    host: '0.0.0.0',
    allowedHosts: ['jeecg-frontend.example.com'],
    public: 'jeecg-frontend.example.com',
  },
}
```

9）启动前端、Redis

执行：

    docker-compose up -d
