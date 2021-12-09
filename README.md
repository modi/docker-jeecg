[Maven私服设置 · JeecgBoot 开发文档 · 看云](http://doc.jeecg.com/2043876)

初始化数据库：

    docker exec -i docker-jeecg_mysql_1 mysql -u root -proot < app/jeecg-boot/db/jeecgboot-mysql-5.7.sql

调整配置文件 `jeecg-boot-module-system/src/main/resources/application-dev.yml`，修改 MySQL、Redis 的设置。

启动后端：

    mvn exec:java -pl jeecg-boot-module-system -Dexec.mainClass="org.jeecg.JeecgSystemApplication

修改 `ant-design-vue-jeecg/.env.development`：

    -VUE_APP_API_BASE_URL=http://localhost:8080/jeecg-boot
    +VUE_APP_API_BASE_URL=http://jeecg-backend.example.com/jeecg-boot

配置 Webpack DevServer：

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
