---
title: selenium简单使用
date: 2023-03-13 19:33:00
description: 使用selenium进行简单的UI测试。
categories:
- 测试
tags:
- 测试
- selenium
---

测试环节同开发一样重要，良好的测试用例可以帮助快速定位问题，代码重构，使开发的软件更加健壮。在web开发中，UI测试不得不提[Selenium](https://www.selenium.dev/)，引用官网的话如下：

> Primarily it is for automating web applications for testing purposes, but is certainly not limited to just that. 

> Boring web-based administration tasks can (and should) also be automated as well.

目前支持众多开发语言：Java、Python、CSharp、Ruby、JavaScript、Kotlin。下面以JavaScript为例。原文来自于[Write your first Selenium script](https://www.selenium.dev/documentation/webdriver/getting_started/first_script/)

Selenium所做的一切就是发送浏览器命令来执行某些操作或发送获取信息的请求。使用Selenium执行大多数操作基本上由以下基础指令组成。

### 基础指令

1. 开启会话
2. 浏览器上执行操作
3. 获取浏览器信息
4. 设置页面等待策略
5. 查找页面元素
6. 页面元素执行操作
7. 请求元素信息
8. 结束会话

完整示例如下：

```
const {By, Builder, Browser} = require('selenium-webdriver');
const {suite} = require('selenium-webdriver/testing');
const assert = require("assert");

suite(function (env) {
  describe('First script', function () {
    let driver;

    // 1. 开启会话
    before(async function () {
      driver = await new Builder().forBrowser('chrome').build();
    });

    // 8. 结束会话
    after(async () => await driver.quit());

    it('First Selenium script', async function () {
      // 2. 浏览器上执行操作
      await driver.get('https://www.selenium.dev/selenium/web/web-form.html'); // 打开页面
			
      // 3. 获取浏览器信息
      let title = await driver.getTitle();
      assert.equal("Web form", title);

      // 4. 设置页面等待策略
      await driver.manage().setTimeouts({implicit: 500}); // 隐式等待500ms

      // 5. 查找页面元素
      let textBox = await driver.findElement(By.name('my-text'));
      
      let submitButton = await driver.findElement(By.css('button'));

      // 6. 页面元素执行操作
      await textBox.sendKeys('Selenium');
      await submitButton.click();

      let message = await driver.findElement(By.id('message'));

      // 7. 请求元素信息
      let value = await message.getText();
      assert.equal("Received!", value);
    });
  });
}, { browsers: [Browser.CHROME, Browser.FIREFOX]});
```

### 运行上述脚本

```
// 1. 新建Node项目，当前Node.js版本v19.2.0
// 2. 添加Selenium依赖 
npm add selenium-webdriver
// 3. 添加测试框架mocha，更多请查看 https://mochajs.org
npm add mocha
// 4. 配置npm脚本
// 将 "test": "npx mocha test/*.js --timeout 60000" 放入package.json的scripts节点中
// 将上述示例代码放入test/first-script.js中
// 5. 运行脚本
npm run test
```

更多请参考：

- [Mocha](https://mochajs.org)
- [Selenium](https://www.selenium.dev/)