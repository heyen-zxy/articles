# 常用gem

### config
##### [config](https://github.com/railsconfig/config)根据不同的生运行环境，做不同的yml配置

### annotate
##### [annotate](https://github.com/ctran/annotate_models)非常好用的rails辅助工具，为每个model文件生成attributes注释

### paper_trail
##### [paper_trail]()一款非常强大的插件，用于记录table的数据变化，为我们debug带来了方便


### ajax-datatables-rails
##### [datatables](http://datatables.club/)本身是一个一款非常好用的基于jquey的插件，新版本已经跟bootstrap有了很好的兼容，[ajax-datatables](https://github.com/antillas21/ajax-datatables-rails)为datatables在ror的应用提供了简便的方法。

	gem 'ajax-datatables-rails'

  	source 'https://rails-assets.org' do

  		gem 'rails-assets-bootstrap'
  
  		gem 'rails-assets-font-awesome', '~> 4.3.0'
  
  		gem 'rails-assets-datatables', '~> 1.10.7'
  
  		gem 'rails-assets-datatables-plugins'
  
  		gem 'rails-assets-datatables-responsive'
  
  		gem 'rails-assets-bootstrap-datepicker'
  
  		gem 'rails-assets-bootstrap-daterangepicker', '~> 2.1.19'
    
  		gem 'rails-assets-admin-lte' , '~> 2.0.5'
  
  		gem 'rails-assets-jquery.treegrid'
  
	end ·
	
	
### Rspec
##### [Rspec](https://github.com/rspec/rspec)一款强大的测试插件。Rspec往往和一些插件结合使用。如：

* [factory_girl](https://github.com/thoughtbot/factory_girl)  一款简便创建测试数据的插件

* [faker](https://github.com/stympy/faker)提供一些简单的方法为我们随机创建name email address 等等

* [database_cleaner](https://github.com/DatabaseCleaner/database_cleaner) 


### carrierwave
##### 一款上传文件的插件，相对于[paperclip](https://github.com/thoughtbot/paperclip)更推荐[carrierwave](https://github.com/carrierwaveuploader/carrierwave)


### kaminari
##### 现在比较流行的分页插件[kaminari](https://github.com/amatsuda/kaminari)



###  whenever
##### [whenever](https://github.com/javan/whenever)定时任务插件

### devise + cancancan + rolify 实现 登录/权限/角色
* [devise](https://github.com/plataformatec/devise)提供登录
* [cancancan](https://github.com/CanCanCommunity/cancancan)提供权限管理
* [rolify](https://github.com/RolifyCommunity/rolify)提供角色管理

### rest-client/savon/bindata获取接口数据
* [rest-client](https://github.com/rest-client/rest-client)获取http协议的接口数据的插件。
* [savon](https://github.com/savonrb/savon)获取web_service协议的接口数据插件
* [bindata](https://github.com/dmendel/bindata)本身不提供获取socket形式的接口数据，链接socket使用ruby原生应用TCPSocket,bindata为我们提供了解析socket数据的方法。


### delayed_job/delayed_job_active_record

##### [delayed_job](https://github.com/tobi/delayed_job)/[delayed_job_active_record](https://github.com/collectiveidea/delayed_job)两个异步处理数据的插件

### weixin_rails_middleware

##### [weixin_rails_middleware](https://github.com/lanrion/weixin_rails_middleware)开发微信应用就用它吧,辅助工具[NATAPP](https://natapp.cn/#about)，基于[ngrok](https://www.npmjs.com/package/ngrok)(容易被墙)的内网穿透工具，在外网上访问你的本地应用。

### ancestry
##### [ancestry](https://github.com/stefankroes/ancestry)方便于model生成树状层级结构，多用于做公司组织架构或分类管理


### mina
##### [mina](https://github.com/mina-deploy/mina)部署项目

### active_importer
##### [active_importer](https://github.com/continuum/active_importer)excel上传

### rails-i18nterface
#### [rails-i18nterface](https://github.com/mose/rails-i18nterface)做多语言或其他语言外包项目 页面可视化管理文言，对于翻译人员来说很实用的软件