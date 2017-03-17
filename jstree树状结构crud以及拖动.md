### JsTree树状结构CRUD以及拖动

##### 需求

* 树状的组织架构，可进行CRUD和拖拽

* 符合bootstrap样式

##### 方法

* 引入[jsTree](https://www.jstree.com/)
>jsTree是一个 基于jQuery的Tree控件。支持XML，JSON，Html三种数据源。提供创建，重命名，移动，删除，拖"放节点操作。可以自己自定义创建，删 除，嵌套，重命名，选择节点的规则。在这些操作上可以添加多种监听事件。

* 使用[jstree-bootstrap-theme](https://rails-assets.org/#/components/bootstrap-treeview)覆盖起原有的样式


##### 解决

* 后端 

```
# rails代码
organization = CopyOrganization.find_by_id params[:id].to_i if (params[:id] != 0)
    return render json: {flag: 3} if current_user.organization.copy_organization.root.subtree.find_by(name: params[:text])
    case params[:type]
      when 'delete'
        //delete
      when 'create'
      #jstree在你添加一个子节点的时候就会产生一个事件，会生成一个newNode，这个时候我们并不保存到数据库。返回一个{id：0}
        render json: {id: 0}
      when 'rename'
        //update 
        #新建的字节点如果传送过来的id是0 则create  
      when 'move'
       // 移动

    end
```

* 前端

```
	$('#container')
    .jstree({
      "core" : {
        'themes': {
          'name': 'proton',
          'responsive': true
        },
        "animation" : 0,
        "check_callback" : true,
        'data' : {
          'url' : function (node) {
            return '<%= get_organization_tree_organizations_path %>'
          },
          'data' : function (node) {
            return { 'id' : node.id };
          },
          type: 'post'
        }
      },
      "types" : {
        "#" : {
          "max_children" : 1,
          "max_depth" : 4,
          "valid_children" : ["root"]
        },
        "root" : {
          "icon" : false,
          "valid_children" : ["default"]
        },
        "default" : {
          "icon" : false,
          "valid_children" : ["default","file"]
        },
        "file" : {
          "icon" : false,
          "valid_children" : []
        }
      },
      "contextmenu":{
      //节点的可选错操作，这里对每一个节点进行了个性化的可选操作处理
        "items": function($node) {
          var tree = $("#container").jstree(true);
          var hash = {}
          var can_create = false;
          var can_rename = false;
          var can_remove = false;
          if($node.original.create == undefined || $node.original.create){
            hash["Create"] = {
              "separator_before": false,
              "separator_after": false,
              "label": "添加子节点",
              "action": function (obj) {
                $node = tree.create_node($node);
                tree.edit($node);
              }
            }

          }
          if($node.original.rename == undefined || $node.original.rename){
            hash["Rename"] = {
              "separator_before": false,
              "separator_after": false,
              "label": "编辑",
              "action": function (obj) {
                tree.edit($node);
              }
            }
          }
          if($node.original.remove == undefined || $node.original.remove){
            hash["Remove"] =  {
              "separator_before": false,
              "separator_after": false,
              "label": "删除",
              "action": function (obj) {
                tree.delete_node($node);
              }
            }
          }
          return hash;
        }
      },
      "plugins" : [
        "contextmenu", "dnd", "state"
      ]
    })
    .on('delete_node.jstree', function (e, data) {
      $.post('<%= operation_organizations_path %>', { 'id' : data.node.id, type: 'delete' })
          .done(function(d){
            if(d.flag == '2'){
              show_flash('failed', '该节点不可删除');
            }else{
              show_flash('success', '删除成功')
            }
            data.instance.refresh();
          })
          .fail(function () {
            show_flash('failed', '删除失败')
            data.instance.refresh();
          });
    })
    .on('create_node.jstree', function (e, data) {
      var text =  data.node.text;
      $.post('<%= operation_organizations_path %>', {'id' : 0, type: 'create', 'text' : data.text })
          .done(function (d) {
            data.instance.set_id(data.node, d.id);
          })
          .fail(function () {
            show_flash('failed', '添加失败')
            data.instance.refresh();
          });
    })
    .on('rename_node.jstree', function (e, data) {
      $.post('<%= operation_organizations_path %>', { 'id' : data.node.id, 'text' : data.text, type: 'rename', 'parent_id' : data.node.parent })
          .done(function(d){
            if(d.flag == '2'){
              show_flash('failed', '修改失败')
            }else if(d.flag == '3'){
              show_flash('failed', data.node.text + '已经存在');
            }else{
              show_flash('success', '修改成功')
            }
            data.instance.refresh();
          })
          .fail(function () {
            data.instance.refresh();
            show_flash('failed', '修改失败')
          });
    })
    .on('move_node.jstree', function (e, data) {
      $.post('<%= operation_organizations_path %>', { 'id' : data.node.id, 'parent' : data.parent, 'position' : data.position, type: 'move' })
          .done(function(d){
            if(d.flag == '2'){
              show_flash('failed', '移动失败')
            }else{
              show_flash('success', '移动成功')
            }
            data.instance.refresh();
          })
          .fail(function () {
            data.instance.refresh();
            show_flash('failed', '移动失败')
          });
    });
```



