---
title: "Ruby/Rails.note"
date: 2012-02-27T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

把平常中一些开发出错以及解决方法记录了下来,其实,基本上都是Google或者StackOverflow得到的答案.然后有些都不知道问题的根源,只知道个解决方法

*
```
Issue--
	RVM is not a function, selecting rubies with 'rvm use ...' will not work.
```

```
Solution--
	添加下面这句到  ~/.bashrc
	[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"重启终端
```

*

```
Issue--
	Gem::Installer::ExtensionBuildError: ERROR: Failed to build gem native extension.
	.
	.
	An error occured while installing mysql2 (0.3.11), and Bundler cannot continue.

	Make sure that `gem install mysql2 -v '0.3.11'` succeeds before bundling.
```

```
Solution--
sudo apt-get install libmysql-Ruby libmysqlclient-dev (Ubuntu)
```

*
```
Issue--
Could not find a JavaScript runtime. See https://github.com/sstephenson/execjs for a list of available runtimes.
```

```
Solution--
Just install execjs and the Rubyracer in your gemfile and run bundle after.
gem 'execjs'
gem 'theRubyracer'
```

*
```
Issue--
>Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
Couldn't create database for {"adapter"=>"mysql2", "encoding"=>"utf8", "database"=>"o_p_", "pool"=>5, "username"=>"root", "password"=>nil, "socket"=>"/tmp/mysql.sock"}, charset: , collation:
```

```
Solution--
host:127.0.0.1(diff??localhost)
```

*
```
Issue--
Gem::Installer::ExtensionBuildError: ERROR: Failed to build gem native extension.

/usr/bin/Ruby1.9.1 extconf.rb
checking for pg_config... no
No pg_config... trying anyway. If building fails, please try again with
--with-pg-config=/path/to/pg_config
checking for libpq-fe.h... no
Can't find the 'libpq-fe.h header
* extconf.rb failed *
Could not create Makefile due to some reason, probably lack of
necessary libraries and/or headers. Check the mkmf.log file for more
details. You may need configuration options.


sudo apt-get install libpq-dev

sudo gem install pg
```
```
Solution--
sudo apt-get install libpq-dev
sudo gem install pg

#貌似跟之前那个mysql的问题有点像
```

```
除了系统的Ruby文件可以用相对路径,自己编写的Ruby文件如果要require的话需要用绝对路径.否则会报错no such file to load .
而load则无论什么Ruby文件都可以用相对路径加载
```