# 摸鱼倒计时

> 一个谷歌浏览器倒计时插件，灵感来源于[sbjim/go-home](https://github.com/sbjim/go-home)，原项目是用 JQuery 写的，并且作者大大好久没更新了，所以我写了一个 Vue 版本，并添加了一些额外的功能。

## 安装步骤

在谷歌浏览器中打开链接 chrome://extensions/

打开开发者模式，加载已解压的扩展程序

选中根目录底下打包生成的 dist 文件夹就可以了

如果需要自己打包，步骤如下：

注意：node 版本要求 14.x

```bash
# 安装依赖
npm install

# 打包
npm run build
```

## 插件默认值

1. 考勤时间默认为 9:00-18:00
2. 发薪日默认为每月 10 号
3. 工作制默认为双休
4. 点餐提醒默认值为空，也就是不提醒
5. 法定节假日信息需要在 src/holiday.js 文件中配置

## 开发进度

- [x] 下班时间，周末，发薪日，法定节假日倒计时
- [x] 考勤时间可配置
- [x] 发薪日可配置
- [x] 法定节假日可配置
- [x] 点餐提醒可配置
- [x] 下班时，桌面弹窗提醒
- [x] 支持双休，大小周，单休工作制配置

所有开发计划已完成，目前暂无开发计划，如果有好的建议或者 BUG 可以通过 issues 反馈

## 插件效果展示

### 页面

![demo1.png](https://gitee.com/GaoWeiQiang1996/go-home-vue/raw/master/src/assets/demo1.png)

### 设置

![demo2.png](https://gitee.com/GaoWeiQiang1996/go-home-vue/raw/master/src/assets/demo2.png)

### 通知

![demo3.png](https://gitee.com/GaoWeiQiang1996/go-home-vue/raw/master/src/assets/demo3.png)

### 喜欢的话给个免费的 star 吧！！！
