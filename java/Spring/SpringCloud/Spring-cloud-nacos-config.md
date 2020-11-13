## Spring Cloud Nacos Config中
### dataId的完整格式如下：
```
${prefix}-${spring.profile.active}-${file-extension}
```
**prefix**: 默认为spring.application.name的值

**spring.profile.active**: 对应当前的环境

**file-exetension**: 配置内容的数据格式，目前只支持yaml和properties

**注意**
优先使用：
prefix-spring.profile.active
如果不存在profile就加载prefix

### Group
默认是没有组的，config都是放在DEFAULT_GROUP

### shared-configs
共享配置文件

---
## Nacos Config的配置文件：
application.yml -> **bootstrap.yml**

---
## 路由网关：Spring Gateway
Spring Gateway默认是使用**webflux**（底层是netty），如果maven中有**spring-web**的依赖会产生冲突
