## 创建自定义的response format

通常在action中我们会用到一下几个类型的返回数据，那么怎么创建自定义的format呢
	
	respond_to do |foramt|
	  format.html
      format.json { render json: @users }
      format.js 
	end 
	
比如我想返回一个excel文件，在config/initializers/mime_types.rb
	
	#mime_types.rb
	Mime::Type.register "application/xls", :xls

现在有了`format.xls`方法了，我们需要添加对应的view文件`users.xls.erb`
	
	#index.xls.erb
	<?xml version="1.0"?>
		<Workbook xmlns="urn:schemas-microsoft-com:office:spreadsheet"
		xmlns:o="urn:schemas-microsoft-com:office:office"
		xmlns:x="urn:schemas-microsoft-com:office:excel"
		xmlns:ss="urn:schemas-microsoft-com:office:spreadsheet"
		xmlns:html="http://www.w3.org/TR/REC-html40">
		  <Worksheet ss:Name="用户">
		    <Table>
		      <Row>
		      <Cell><Data ss:Type="String">姓名</Data></Cell>
		      <Cell><Data ss:Type="String">邮箱</Data></Cell>
		      <Cell><Data ss:Type="String">QQ</Data></Cell>
		      <Cell><Data ss:Type="String">年龄</Data></Cell>
		      <Cell><Data ss:Type="String">地址</Data></Cell>
		      </Row>
		      <% User.all.each do |record| %>
		          <Row>
		            <Cell><Data ss:Type="String"><%= record.name %></Data></Cell>
		            <Cell><Data ss:Type="String"><%= record.email %></Data></Cell>
		            <Cell><Data ss:Type="String"><%= record.qq %></Data></Cell>
		            <Cell><Data ss:Type="String"><%= record.age %></Data></Cell>
		            <Cell><Data ss:Type="String"><%= record.address %></Data></Cell>
		          </Row>
		      <% end %>
		    </Table>
		    </Worksheet>
		
		  <Worksheet ss:Name="组织架构">
		    <Table>
		      <Row>
		        <Cell><Data ss:Type="String">组织名</Data></Cell>
		      </Row>
		      <% Organize.all.each do |record| %>
		          <Row>
		            <Cell><Data ss:Type="String"><%= record.name %></Data></Cell>
		          </Row>
		      <% end %>
		    </Table>
		  </Worksheet>
		
		</Workbook>
		
调用`format.xls`
	
	respond_to do |format|
      format.html
      format.json { render json: @users }
      format.xls{
        headers["Content-Disposition"] = "attachment; filename=\"用户-#{Date.today}.xls\""
      }
     
    end
    
同样以上方式可以用于其他类型文件的下载
		
