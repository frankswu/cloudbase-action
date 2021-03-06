# @cloudbase/database

[![NPM Version](https://img.shields.io/npm/v/@cloudbase/database.svg?style=flat)](https://www.npmjs.com/package/@cloudbase/database)
[![](https://img.shields.io/npm/dt/@cloudbase/database.svg)](https://www.npmjs.com/package/@cloudbase/database)

## 介绍

提供 TCB JS SDK 操作数据库的接口。

## 安装

```
yarn add @cloudbase/database
```

## 使用

使用 `@cloudbase/database` 时，需要提供发送请求的类 `reqClass`。 

```js
// index.js

const database = require('@cloudbase/database/').Db;
const Request = require('./request');

class Tcb {
  ...
  database(dbConfig) {
    database.reqClass = Request;
    return new database(dbConfig);
  }
}
```

实现 `Request` 类，需要提供异步的 `send` 方法。

```js
// request.js

// 进一步处理，鉴权等...
const requestHandler = require('requestHandler');

class Request {
  constructor(config) {
    this.config = config;
  }

  async send(action, data) {
    const params = Object.assign({}, data, {
      action
    });

    const slowQueryWarning = setTimeout(() => {
      console.warn(
        'Database operation is longer than 3s. Please check query performance and your network environment.'
      );
    }, 3000);

    try {
      return await requestHandler({
        timeout: this.config.timeout,
        config: this.config.config,
        params,
        method: 'post',
        headers: {
          'content-type': 'application/json'
        }
      });
    } finally {
      clearTimeout(slowQueryWarning);
    }
  }
}

module.exports = Request;
```

请求数据样例

```json
{
  "url": "https://tcb-admin.tencentcloudapi.com/admin?eventId=1554896261428_92044",
  "method": "post",
  "timeout": 15000,
  "headers": { "user-agent": "tcb-admin-sdk/1.4.6", "content-type": "application/json" },
  "body": {
    "collectionName": "coll-1",
    "action": "database.addCollection",
    "envName": "base-830cab",
    "timestamp": 1554896261428,
    "eventId": "1554896261428_92044",
    "authorization": "q-sign-algorithm=sha1&q-ak=xxx&q-sign-time=1554896260;1554897160&q-key-time=1554896260;1554897160&q-header-list=content-type;user-agent&q-url-param-list=action;collectionname;envname;eventid;timestamp&q-signature=xxxxx",
    "sdk_version": "1.4.6"
  },
  "json": true
}
```