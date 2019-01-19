---
title: "Rails 4 Devise Strong Parameters"
date: 2013-12-20T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

重拾 Rails, 然后用了 devise 这个 Gem ,遂继续笔记之

**Devise** 在 ruby 社区里头几乎都称之为一门**重**炮, 当然了, 重换来的是功能丰富, 用户注册登录相关的一个 Gem 搞定, 只是要把源码搞透彻那必须流弊的人才干得来!

现阶段对我来说, 知道怎么用就 OK 了.

但是, 其实也还算不上完全知道怎么用. 遂笔记 MARK 一发.

参照官方的**Getting started**
添加 Gem

```ruby
gem 'devise'
```
安装 Gem

`bundle`

跑 generator

```ruby
rails g devise:install
```

and then 是一坨的提示，照着撸完，之后

```ruby
rails g devise user
rake db:migrate
rails generate devise:views
```

至于如果需要添加 index  show 等其他一些 action 的话直接再 generate 一个 users controller
不过 在`routes.rb`中需要加一句 (在`  devise_for :users
`之后)

  `resources :users, only: [:index, :show]`

然后是添加其他字段, add username migration 之后

```ruby
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.for(:sign_up) << :username
  end

end
```

### strong parameter

印象中 rails3 下如
`Post.new(params[:post])` 是可以正常运作的,

但是, 在 rails4 当中就报错

```

ActiveModel::ForbiddenAttributesError in PostsController#create

```

在 Rails4 以后要改成

```ruby
Post.new(params.require(:post).permit(:title, :content))
```
 相当于以往的 **attr_accessible**
把工作从 model 转移到 controller 了, 规避了以前的 mass assignment 问题, 又想起以前 github 中招 XD

摘抄官方 guides

```ruby
class PeopleController < ActionController::Base
  # This will raise an ActiveModel::ForbiddenAttributes exception
  # because it's using mass assignment without an explicit permit
  # step.
  def create
    Person.create(params[:person])
  end

  # This will pass with flying colors as long as there's a person key
  # in the parameters, otherwise it'll raise a
  # ActionController::ParameterMissing exception, which will get
  # caught by ActionController::Base and turned into that 400 Bad
  # Request reply.
  def update
    person = current_account.people.find(params[:id])
    person.update_attributes!(person_params)
    redirect_to person
  end

  private
    # Using a private method to encapsulate the permissible parameters
    # is just a good pattern since you'll be able to reuse the same
    # permit list between create and update. Also, you can specialize
    # this method with per-user checking of permissible attributes.
    def person_params
      params.require(:person).permit(:name, :age)
    end
end
```
所以上边要先把 username 在 params 给 permit 了.

暂时是这些先 MARK

