### Rails5 ActionCable创建一个简易聊天室
本文参考[DHH大神的视频](https://www.youtube.com/watch?v=n0WUjGkDFS0)，[RubyChina](https://ruby-china.org/topics/28480)上已经有人把视频转到[国内railscasts-china](http://railscasts-china.com/episodes/action-cable-rails-5)了

#### 新建一个rails项目 

	rails _5.0.0_ new test_cable #指定rails版本5.0.0
	
#### 创建一个controller action

	rails g controller users talk
	
#### 创建一个 message model

	rails g model message content:text
	rails g db:migrate  #rails5将rake命令整合到了rails
#### 修改route
	root to: 'users#talk'
	
#### 添加页面显示
	#users_controller
	def talk
      @messages = Message.all
    end
    
    
    #users/talk.html.erb
    <h1>room</h1>
	<div id="messages">
  		<%= render @messages %>
    </div>
    
    #messages/_message.html.erb    
    <p><%= message.content %></p>
    
以上我们的项目可以正常启动了
	
--------	
#### 添加ActionCable
	rails g channel room speak
	create  app/channels/room_channel.rb
    identical  app/assets/javascripts/cable.js
    create  app/assets/javascripts/channels/room.coffee
    
重新运行项目我们发现浏览器控制台报错

	WebSocket connection to 'ws://localhost:3001/cable' failed: Error during WebSocket handshake: Unexpected response code: 404

服务器控制台报错

	Request origin not allowed: http://localhost:3001
	Failed to upgrade to WebSocket (REQUEST_METHOD: GET, HTTP_CONNECTION: Upgrade, HTTP_UPGRADE: websocket)
	
修改：

	#在config/environments 下对应的环境中添加 
	# Action Cable endpoint configuration
    config.action_cable.url = '/cable'
    config.action_cable.allowed_request_origins = [ 'http://localhost:3001' ]
    
重新启动服务器

	#控制台显示已经正常启动
	Finished "/cable/" [WebSocket] for ::1 at 2016-10-27 16:20:33 +0800
	Started GET "/cable" for ::1 at 2016-10-27 16:20:33 +0800
	Started GET "/cable/" [WebSocket] for ::1 at 2016-10-27 16:20:33 +0800
	Successfully upgraded to WebSocket (REQUEST_METHOD: GET, HTTP_CONNECTION: Upgrade, 	HTTP_UPGRADE: websocket)
	RoomChannel is transmitting the subscription confirmation

在浏览器控制台输入

	App.room
	> Subscription {consumer: Consumer, identifier: "{"channel":"RoomChannel"}"}
	App.room.speak()
	> true
	
####测试我们的ActionCable是否已经成功

修改app/assets/javascripts/channels/room.coffee

	#此处的speak方法对应 app/channels/room_channel.rb 的speak
	App.room = App.cable.subscriptions.create "RoomChannel",
  		connected: ->
    		# Called when the subscription is ready for use on the server

  		disconnected: ->
    		# Called when the subscription has been terminated by the server

  		received: (data) ->
    		alert data['message']
    		# Called when there's incoming data on the websocket for this channel

  		speak: (message)->
    		@perform 'speak', message: message
    		
修改app/channels/room_channel.rb
    		
    class RoomChannel < ApplicationCable::Channel
  		def subscribed
    		stream_from "room_channel"
  		end

  		def unsubscribed
   			# Any cleanup needed when channel is unsubscribed
  		end

  		def speak(data)
    		ActionCable.server.broadcast 'room_channel', message: data['message']
  		end
	end
	
刷新页面，在页面控制台执行方法

	App.room.speak('hello')
	
看到页面alert hello，ok以上我们已经完成了一次websocket的交互

#### 完善聊天室

在users/talk.html.erb 添加
	
	<%= text_field_tag :speak %>
	//添加一个监听事件
	<script type="text/javascript">
  		$("#speak").on('keypress', function(event){
    		if(event.keyCode == 13){//enter
      			if($(this).val()){
        			App.room.speak($(this).val());
        			$(this).val('');  
      			}
    		}
  		});
	</script>
	
上面我们已经可以将聊天内容提交到服务器了，接下来服务器接收到数据之后应该创建一个message

	#app/channels/room_channel.rb
	def speak(data)
    	Message.create content: data['message']
  	end
 
 添加一个after_create事件，将当message反应到浏览器，
 	#app/model/message.rb
 	after_create_commit {MessageBroadcastJob.perform_later self}
 
 
 这里我们使用ApplicationJob来完成
 	
  	#控制台执行
  	rails g job MessageBroadcast
  	>  invoke  test_unit
    >  create    test/jobs/message_broadcast_job_test.rb
    >  create  app/jobs/message_broadcast_job.rb
    
    #修改  app/jobs/message_broadcast_job.rb，渲染messages/message模版之后船体给前端
    class MessageBroadcastJob < ApplicationJob
  		queue_as :default

  		def perform(message)
    		ActionCable.server.broadcast 'room_channel', message: render_message(message)
    		# Do something later
  		end

  		private

  		def render_message(message)
    		ApplicationController.renderer.render partial: 'messages/message', locals: 		{message: message}
  		end


	end
	
前端接收之后再页面上添加内容

	App.room = App.cable.subscriptions.create "RoomChannel",
  		connected: ->
    		# Called when the subscription is ready for use on the server

  		disconnected: ->
    		# Called when the subscription has been terminated by the server

  		received: (data) ->
    		$("#messages").append data['message']
    		# Called when there's incoming data on the websocket for this channel

  		speak: (message)->
    		@perform 'speak', message: message
    		
    		
    		
    		
#### 疑问
DHH大神的视频中在route.rb里面添加了 `mount ActionCable.server => '/cable'`
然而我并没有添加，开发依然正常    		
    		
    		
   
		
    
    


	


    







