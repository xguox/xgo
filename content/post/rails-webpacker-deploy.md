---
title: "Rails 5 webpacker 部署时候报错"
date: 2017-07-26T16:01:23+08:00
draft: false
tags: ["JavaScript", "React", "Ruby"]
categories: ["JavaScript", "React", "Ruby"]
---

---
layout: post
title: "Rails 5 webpacker 部署时候报错"
date: 2017-07-26 13:25:20
categories: [JavaScript, React, Ruby]
tags: [JavaScript, React, Ruby]
---
<!--more-->

```ruby
SSHKit::Command::Failed: rake exit status: 2
rake stdout: yarn install v0.21.3
[1/4] Resolving packages...
[2/4] Fetching packages...
info "fsevents@1.1.2" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 152.92s.
Webpacker is installed 🎉 🍰
Using /rails_apps/supply_chain/releases/20170725080931/config/webpack/paths.yml file for setting up webpack paths
Compiling webpacker assets 🎉
rake stderr: WARNING: Use strings for Figaro configuration. false was converted to "false".
WARNING: Use strings for Figaro configuration. true was converted to "true".
warning No license field
warning fsevents@1.1.2: The platform "linux" is incompatible with this module.
```

到服务器上手工执行
```ruby
RAILS_ENV=staging bundle exec rake assets:precompile --trace
```

结果报了

```ruby
No PostCSS Config found  一堆。。。。。。。。。。。。。
```

改一下 `config/webpack/loaders/sass.js`

```javascript
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const path = require('path')

const { env } = require('../configuration.js')
const postcssConfig = path.resolve(process.cwd(), '.postcssrc.yml')

module.exports = {
  test: /\.(scss|sass|css)$/i,
  use: ExtractTextPlugin.extract({
    fallback: 'style-loader',
    use: [
      { loader: 'css-loader', options: { minimize: env.NODE_ENV === 'production',  } },
      { loader: 'postcss-loader', options: { sourceMap: true, config: { path: postcssConfig } } },
      'resolve-url-loader',
      { loader: 'sass-loader', options: { sourceMap: true } }
    ]
  })
}
```

完事, 收工.