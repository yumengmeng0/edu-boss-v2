# edu-boss

!alt[./images/prview.png]

# SSM 单体项目：拉勾教育后台管理系统

对应的后端：SSM 单体项目：拉勾教育后台管理系统
https://github.com/yumengmeng92/lagou_edu_home_parent

```shell
## Project setup
$ npm install
### Compiles and hot-reloads for development
$ npm run serve
### Compiles and minifies for production
$ npm run build
### Lints and fixes files
$ npm run lint
### E2E and Unit tests
$ npm run test

### Customize configuration
# See [Configuration Reference](https://cli.vuejs.org/config/).
```


# 模版
```js
return axios
	.get("")
	.then((response) => {})
	.catch((error) => {
		this.$message.error("失败！");
	});
```

# Promise 返回的问题

如何将 Promise.then 中的值直接 return 出来：https://cloud.tencent.com/developer/article/1839124  
使用 async….await，async 声明发放为异步方法，await 等待异步操作执行完毕。

```javascript

    // 获取字符串
    getSpaceName(spaceId) {
        console.log(spaceId);
        let spaceName = "";
        axios
            .get("/promotionSpace/findPromotionSpaceById", {
                params: {
                    id: spaceId
                }
            })
            .then(resp => {
                //    不能在这里直接返回
                // return resp.data.content.name
                // 不能在异步Promise内部返回
                spaceName = resp.data.content.name;
            })
            .catch(error => {
                this.$message.error("查询广告位置失败！");
            });
        return spaceName;
    },


    async getSpaceName(spaceId) {
        console.log(spaceId);
        let spaceName = "";
        let result = await axios
            .get("/promotionSpace/findPromotionSpaceById", {
                params: {
                    id: spaceId
                }
            })
            .then(resp =>
                //    不能在这里直接返回
                // return resp.data.content.name
                // 不能在异步Promise内部返回
               resp.data.content.name;
            )
            .catch(error => {
                this.$message.error("查询广告位置失败！");
            });
            console.log(result);
        return spaceName; // 这里返回的是Promise对象
    },
```
