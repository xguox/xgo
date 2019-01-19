---
title: "Rails 5 belongs_to 默认 required"
date: 2016-09-24T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

```ruby
class Team < ApplicationRecord
  has_many :projects
end

class Project < ApplicationRecord
  belongs_to :team
end

[1] pry(main)> Project.create(name: 'project_1')
   (0.2ms)  BEGIN
   (0.2ms)  ROLLBACK
=> #<Project:0x007fb8462ddf10 id: nil, team_id: nil, name: "project_1", created_at: nil, updated_at: nil>
[2] pry(main)> _.errors
=> #<ActiveModel::Errors:0x007fb84634c500
 @base=#<Project:0x007fb8462ddf10 id: nil, team_id: nil, name: "project_1", created_at: nil, updated_at: nil>,
 @details={:team=>[{:error=>:blank}]},
 @messages={:team=>["must exist"]}>
```

在 Rails 5 以前是可以成功 Save 的, 但是, 从 Rails 5 开始, belongs_to 的关联都默认必须有值, 除非这么写:

```ruby
class Project < ApplicationRecord
  belongs_to :team, optional: true
end

[3] pry(main)> Project.create(name: 'project_1')
   (0.2ms)  BEGIN
  SQL (5.4ms)  INSERT INTO "projects" ("name", "created_at", "updated_at") VALUES ($1, $2, $3) RETURNING "id"  [["name", "project_1"], ["created_at", 2016-09-28 04:05:41 UTC], ["updated_at", 2016-09-28 04:05:41 UTC]]
   (1.0ms)  COMMIT
=> #<Project:0x007fff3b2bc388 id: "05051810-fea1-4ae9-b138-05a92e0bb3e9", team_id: nil, name: "project_1", created_at: Wed, 28 Sep 2016 04:05:41 UTC +00:00, updated_at: Wed, 28 Sep 2016 04:05:41 UTC +00:00>
```

要使用像原本那样默认可选的话, 在 `config/initializers/new_framework_defaults.rb` 改一下

```ruby
Rails.application.config.active_record.belongs_to_required_by_default = false
```
