---
title: "DRY"
date: 2012-03-04T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

从刚接触Rails开始就被灌输三种观念--DRY、COC、REST.虽然这三种思想还没完全的领悟透彻,但是,已经感受到了它们的强大之处.

目前印象最深刻的则是DRY.最近在跟着Ruby on Rails Tutorial学写一点Rspec测试.这种感觉又更强烈了.看着下面代码一步步的减少,感慨DRY的思想无处不在啊.这里只是一个Example可能感觉不到什么,但是当Example多了而且几乎都在测试同一样东西的时候,优点不言而喻.

```ruby
describe "Home page" do
    it "should have the h1 'Sample App'" do
    	visit '/static_pages/home'
      page.should have_selector('h1', text: 'Sample App')
    end

    it "should have the title 'Home'" do
      visit '/static_pages/home'
      page.should have_selector('title',
            text: "Ruby on Rails Tutorial Sample App | Home")
    end
  end
```

```ruby
describe "Home page" do
  it "should have the h1 'Sample App'" do
    visit root_path
    page.should have_selector('h1', text: 'Sample App')
  end

  it "should have the title 'Home'" do
    visit root_path
    page.should have_selector('title',
                      text: "Ruby on Rails Tutorial Sample App | Home")
  end
end
```

```ruby
describe "Home page" do
  before { visit root_path }

  it "should have the h1 'Sample App'" do
    page.should have_selector('h1', text: 'Sample App')
  end

  it "should have the title 'Home'" do
    page.should have_selector('title',
                      text: "Ruby on Rails Tutorial Sample App | Home")
  end
end
```

```ruby
  subject { page }

  describe "Home page" do
    before { visit root_path }

    it { should have_selector('h1', text: 'Sample App') }
    it { should have_selector 'title',
                        text: "Ruby on Rails Tutorial Sample App | Home" }
  end
```

```ruby
  subject { page }

  describe "Home page" do
    before { visit root_path }

    it { should have_selector('h1',    text: 'Sample App') }
    it { should have_selector('title', text: full_title('Home')) }
  end
```

```ruby
shared_examples_for "all static pages" do
    it { should have_selector('h1',    text: heading) }
    it { should have_selector('title', text: full_title(page_title)) }
  end

  describe "Home page" do
    before { visit root_path }
    let(:heading)    { 'Sample App' }
    let(:page_title) { 'Home' }

    it_should_behave_like "all static pages"
  end
```

其实,Rspec只是Ruby写的测试框架.但是,目前Rails程序上写测试用的最多的还是Rspec吧.而在Rails中,最直观的运用到这思想的应该是helper、partial了吧.把好几个view的重复代码.整合在一个partial下.代码量的减少不言而喻.减少了出错的几率.修改重构起来方便很多.

当然DRY的思想绝不仅仅在于这一点点.不然国外也没那么名著专门描述它.

昨天刚从一位网友也可以说是Rails学习道友那买了本打印版的《Agile Web Development with Rails,Fourth Edition》.在跟这位道友面交的时候他不断地鼓舞我,即使到最后用的人寥寥无几也要坚持 Rails下去,坚持它的思想.其实,不用他跟我说我也会坚持.不然,我不会在学校开展JAVA课程的时候自己那么寂寞的跑去学Ruby/Rails.只是他的一番话的确对我又有了不少的鼓舞.他不断地说我和年纪就接触这么新的语言框架什么的,其实,我更感慨敬佩他比我大了15~6岁还那么有power学习这些新事物.

OK,扯远了,Over