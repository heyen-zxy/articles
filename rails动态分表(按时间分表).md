## 动态分表

#### 问题

目前遇到的问题是，随着时间的推移产生的数据量越来越多，类似于话单数据。


#### 逻辑
* 解决数据库中动态的去创建表，时间到了自动创建一个和主表相同数据结构的表

* 在rails项目中，动态的去定义与表对应的active model，并调用

* 解决查询跨表的问题

#### 解决(下面采用的按月分表)

* 时间到了自动创建表，最先想到的是[whenever](https://github.com/javan/whenever)定时任务,引入gemfile`gem 'whenever', :require => false`

	* 初始化，在项目目录执行
	
``` test-MacBook-Pro:test_messages zxypow$ wheneverize .
[add] writing `./config/schedule.rb'
[done] wheneverized! ```
			
	* 编辑`schedule.rb`
			
			#schedule.rb
			#每天(根据需求设定)检测创建一次当前月的表和下个月的表
			every 1.days, :at => '0:00 am' do
  				runner 'User.create_table'
  				runner 'User.create_table Date.today.next_month'
			end

* 动态定义active model，利用ruby元编程实现
	
	* 为了方便适用于不同的model，在app/model/concern文件夹下创建dynamic_table.rb
		
			#dynamic_table.rb
			module DynamicTable
			  extend ActiveSupport::Concern
			
			  def self.included(base)
			    base.extend ClassMethods
			  end
			
			  module ClassMethods
			    def get_dynamic_class(date = Date.today)
			      class_name = set_class_name(date)
			      class_name.constantize
			    rescue NameError
			      define_dynamic_class(date)
			    end
			
			
			    def create_table date=Date.today
			      new_table_name = set_table_name date
			      conn = ActiveRecord::Base.connection
			      #将原始的表复制一份到新的动态生成的表
			      conn.execute "CREATE TABLE `#{new_table_name}` SELECT * FROM `#{table_name}` WHERE 1=2 " unless conn.table_exists? new_table_name
			    end
			
			
			    private
			
			    def set_table_name date = Date.today
			      "#{table_name}_#{date_suffix date}"
			    end
			
			    def set_class_name(date = Date.today)
			      "#{name}_#{date_suffix date}".classify
			    end
			
			    def date_suffix(date = Date.today)
			      date.strftime '%Y_%m'
			    end
			
			    #创建动态的class
			    def define_dynamic_class(date = Date.today)
			      class_name = set_class_name date
			      table_name = set_table_name date
			      Object.const_set class_name, Class.new(ActiveRecord::Base) { self.table_name = "#{table_name}" }
			      class_name.constantize
			    end
			  end
			
			
			end
		
	* 在model 中添加
			
			#user.rb
			
			include DynamicTable
		
* 解决数据查询的问题
	
	* 基础的CRUD只需要`model.get_dynamic_class date`即可获得数据库对应的model
	
	* 跨表查询，因为我们的表结构是一样的，最先想到的是数据库的union方法，完全满足我们的需求，并且已经有人写了gem [active_record_union](https://github.com/brianhempel/active_record_union)，我们只需要使用gem即可，gemfile中引入`gem 'active_record_union'`
			
			#查询
			>>User.all.union(User.get_dynamic_class.all).where(created_at: [Date.yesterday.beginning_of_day, Date.tomorrow.end_of_day])
			#<ActiveRecord::Relation []>
			User Load (3.8ms)  SELECT `users`.* FROM ( (SELECT `users`.* FROM `users`) UNION (SELECT `users_2016_11`.* FROM `users_2016_11`) ) `users` WHERE `users`.`created_at` IN ('2016-11-21 00:00:00.000000', '2016-11-23 23:59:59.999999')
			
	* 通常我们查询的时候都是一个时间区间，那么union的model也是动态的,在dynamic_table.rb中添加一个方法
			
			# dynamic_table.rb	
			#起始时间和结束时间添加上默认值	
			def union_table begin_date=Date.new(2016, 1, 1), end_date = Date.today
			  #每个月截取一天作为动态创建model的参数
		      dates = (begin_date..end_date).to_a.uniq{|x| x.strftime('%Y%m') }
		      result = all
		      dates.each do |date|
		        result = result.union(get_dynamic_class(date).all) if ActiveRecord::Base.connection.table_exists? set_table_name(date)
		      end
		      result
		    end
		    
	* 跨表查询
	
			>>User.union_table(Date.new(2016, 2, 3), Date.today).where(created_at: Date.new(2016, 2, 3)..Date.today)
			#<ActiveRecord::Relation []>		
			User Load (6.4ms)  SELECT `users`.* FROM ( (SELECT `users`.* FROM ( (SELECT `users`.* FROM ( (SELECT `users`.* FROM ( (SELECT `users`.* FROM `users`) UNION (SELECT `users_2016_08`.* FROM `users_2016_08`) ) `users`) UNION (SELECT `users_2016_09`.* FROM `users_2016_09`) ) `users`) UNION (SELECT `users_2016_10`.* FROM `users_2016_10`) ) `users`) UNION (SELECT `users_2016_11`.* FROM `users_2016_11`) ) `users` WHERE (`users`.`created_at` BETWEEN '2016-02-03' AND '2016-11-22')
			
			
		

		



