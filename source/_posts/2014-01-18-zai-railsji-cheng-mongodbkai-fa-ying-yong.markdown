---
layout: post
title: "在rails集成mongodb开发应用"
date: 2014-01-18 16:04:48 +0800
comments: true
sharing: true
footer: true
categories: [rails]
---
最近尝试用rails集成mongodb开发我的第一个rails应用，用IDE直接创建project之后，发现工程默认会采用sqlite数据库。于是需要自己手工进行一些数据库配置：
如果你的app已经使用IDE建立好了，需要修改的文件有 config/application.rb 和 config/environments/development.rb

{% codeblock lang:ruby config/application.rb %}
require 'rails/all'
{% endcodeblock %}
替换为 =>
{% codeblock lang:ruby config/application.rb %}
require "action_controller/railtie"
require "action_mailer/railtie"
require "sprockets/railtie"
require "rails/test_unit/railtie"
{% endcodeblock %}

{% codeblock lang:ruby config/environments/development.rb %}
 #config.active_record.migration_error = :page_load
{% endcodeblock %}
<!-- more -->
上面两个文件修改之后就基本把sqlite remove了。当然如果你还没有建立工程直接在命令行中：
{% codeblock %}
rails new appname --skip-active-record
{% endcodeblock %}

到这里我们的app中基本就算是干净的了。下面我们只需要在Gemfile中加入我们所需要的数据库依赖包就可以了，我这里直接加入mongoid
{% codeblock lang:ruby %}
gem 'mongoid', '~> 4.0.0.alpha1', github: 'mongoid/mongoid'
{% endcodeblock %}

修改Gemfile后，我们 bundle install 就会把相关包依赖下载下来，剩下的我们就可以直接使用mongo了。config目录下还有 database.yml 文件，查看内容应该是sqlite的数据库配置文件，我们remove之，然后新建一个属于我们自己的 mongoid.yml. 
{% codeblock config/mongoid.yml %}
development:
  sessions:
    default:
      database: app_development
      hosts:
        - localhost:27017

test:
  sessions:
    default:
      database: app_test
      hosts:
        - localhost:27017

production:
  sessions:
    default:
      database: app_production
      hosts:
        - localhost:27017
{% endcodeblock %}

文件新建完毕，下面我们就需要怎么使app加载这个文件。继续新建文件 config/initializers/mongoid.rb
{% codeblock lang:ruby config/initializers/mongoid.rb %}
Mongoid.load!(Rails.root.to_s+'/config/mongoid.yml')
{% endcodeblock %}

至此，数据库也基本配置完毕。到命令行中 rake 下面的task
{% codeblock Terminal %}
rake db:mongoid:create_indexes
{% endcodeblock %}

开始在model中新建model ～～ rails + mongo 模式开启！
  