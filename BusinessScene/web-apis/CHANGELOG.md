2017-04-06
    登录 todo
    http://127.0.0.1/formroutenew?html=route-neo4j-debug&query=graph-rabc-login
    
    return apoc.util.md5(split('123456',','))   //e10adc3949ba59abbe56e057f20f883e
    return apoc.util.md5(['123456'])              e10adc3949ba59abbe56e057f20f883e

    更改口令为123456
    MATCH (u:User)
    WITH COLLECT(u) AS us
    FOREACH (i IN us |
         SET i.passwd="e10adc3949ba59abbe56e057f20f883e" )
     
    
    -------------------------------------------------------------------------------------------------------------------- 
    展示关联关系的处理，通过其它入口对关系进行了修改，要在页面进行展现。
    
    组织管理 list
        MATCH (group:Group) 		
        OPTIONAL MATCH (group)-[:HASROLE]->(role:Role)  
        OPTIONAL MATCH (group)-[:HASUSER]->(user:User)
        WITH group, collect(DISTINCT role.id) as roles ,collect(DISTINCT role.name) as rolelist ,count(DISTINCT user) as usercnt   
        RETURN  collect({id:group.id,pid:group.pid,name:group.name,createDate:group.createDate,roles:roles,rolelist:rolelist,usercnt:usercnt})
    用户管理 list
        {% if isempty(org) and isempty(role) then %} 	MATCH (n:User) {% else %}  {% if not isempty(org) then %} 	MATCH (g:Group)-[r:HASUSER]->(n:User) where g.id={org}  {% end %}	{% if not isempty(role) then %}  MATCH (r:Role)<-[ur:HASROLE]-(n:User) where r.id={role}  {% end %}{% end %}  
        \n {% if not isempty(searchvalue) then %}	{% if  isempty(org) and isempty(role) then %} WHERE   {% else %} and {% end %}   (n.name=~{searchvalue} OR n.id=~{searchvalue}) {% end %} \n WITH count(*) AS cnt	{% if isempty(org) and isempty(role) then %} 	MATCH (n:User) {% else %}		{% if not isempty(org) then %} 	MATCH (g:Group)-[r:HASUSER]->(n:User) where g.id={org}  {% end %}		{% if not isempty(role) then %}  MATCH (r:Role)<-[ur:HASROLE]-(n:User) where r.id={role}  {% end %}	{% end %}{% if not isempty(searchvalue) then %}	{% if isempty(org) and  isempty(role)  then %} WHERE   {% else %} and {% end %}   (n.name=~{searchvalue} OR n.id=~{searchvalue}) {% end %}  
        \n OPTIONAL MATCH (n)-[:HASROLE]->(role:Role) 
        \n OPTIONAL MATCH (group)-[:HASUSER]->(n) 
        \n WITH  collect(DISTINCT role.id) as roles,collect(DISTINCT group.id) as groups,collect(DISTINCT role.name) as rolelist,collect(DISTINCT group.name) as grouplist, n, cnt SKIP (toInt({page})-1)*toInt({rows}) LIMIT toInt({rows})   
        \n RETURN   { total: cnt, rows:collect({id:n.id,name:n.name,createDate:n.createDate,roles:roles,org:groups,rolelist:rolelist,orgname:grouplist}) } AS molecules
    角色管理
        MATCH (role:Role)   
        OPTIONAL MATCH (group:Group)-[:HASROLE]->(role)   
        OPTIONAL MATCH (role)-[:HASAUTH]->(auth:Auth)
        OPTIONAL MATCH (role)<-[:HASROLE]-(user:User)         
        with  role,count(DISTINCT user) as usercnt ,collect(DISTINCT group.id) as orgs ,collect(DISTINCT group.name) as orglist ,collect(DISTINCT auth.id) as auths ,collect(DISTINCT auth.id) as authlist 
        return collect({id:role.id,name:role.name,createDate:role.createDate,orgs:orgs,orglist:orglist,auths:auths,authlist:authlist,usercnt:usercnt})
    权限列表
        MATCH (auth:Auth) 
        OPTIONAL MATCH (role:Role)-[:HASAUTH]->(auth)
        OPTIONAL MATCH (auth)-[:HASRES]->(resource:Resource)         
        WITH auth, collect(DISTINCT role.id) as roles ,collect(DISTINCT role.name) as rolelist ,collect(DISTINCT resource.id) as resources ,collect(DISTINCT resource.name) as resourcelist  
        return collect({id:auth.id,name:auth.name,createDate:auth.createDate,roles:roles,rolelist:rolelist,resources:resources,resourcelist:resourcelist})
        

MATCH (n:User)   
 
WITH count(*) AS cnt MATCH (n:User)  
OPTIONAL MATCH (user)-[:HASROLE]->(role:Role)   
OPTIONAL MATCH (group)-[:HASUSER]->(user:User) 
WITH   n, cnt SKIP 0 LIMIT 8 , collect(DISTINCT role.id) as roles ,collect(DISTINCT role.name) as rolelist  ,  collect(DISTINCT group.id) as groups ,collect(DISTINCT group.name) as grouplist 
RETURN   { total: cnt, rows:collect(n) } AS molecules

    组件化 datagrid  treegrid  增删改查；***无必要进行再次封装
        数据源；
        页面描述；
        单页面多个同一组件；
        

    前端将数据直接转换为数组；
    FOREACH( authline in  split(line.auths,\",\") |
    FOREACH( authline in  line.auths) |
    组织机构管理/用户管理/角色管理/权限管理/资源管理 ok ;
    
    
2017-04-05
    重构：
        用户增删改查ok;
        组织机构增删改查；关系处理 ok;
        
    fixme CQL 数组数据的处理；
    
2017-04-01
    左侧选中状态需要增加 class;
    <li class="active">
    
2017-03-31
    EasyUI 作为默认前端，废弃 Datatables ;
    
    管理平台集成 RBAC 页面；
    http://127.0.0.1/formroutenew?html=manage/my-index&query=q0&page=0&size=3
    
    组织机构
    http://127.0.0.1/formroutenew?html=rbac/group-tree&query=graph-rbac-grouplist
    
    
    
    原有页面测试恢复
    
    WEB 前端页面：http://127.0.0.1/formroutequery?html=web/web-index&query=q0&page=0&size=3
    
    MNG 管理页面：http://127.0.0.1/formroutequery?html=manage/my-index&query=q0&page=0&size=3
    

2017-03-29
    todo:组件化
    todo:数据规范化；

    RBAC+鉴权；
    csv 数据重构，Neo4jImport 专有格式；
    EasyUI 页面组件重构，lua-template;
    Neo4j 查询语句重构，查询，增加，删除，修改，lua-template;
    Json 重构，返回数据格式重构，TreeJson;


    业务场景：快查
        用户的资源列表；
        角色资源列表；
        资源下面的用户；
        资源下面的角色；
        
    
    权限列表 增删改查完成
        http://127.0.0.1/formroutenew?html=rbac/role-list&query=graphuserlist&page=1&rows=8&org=sys

    

    资源列表
    http://127.0.0.1/formroutenew?html=route-neo4j-resource-tree&query=graphresourcelist&page=1&rows=8
    http://127.0.0.1/formroutenew?html=rbac/resource-tree&query=graphorglist&page=1&rows=8
    增删改查 ok
    与权限关联 ok
    
2017-03-28
    角色列表 配置用户列表
    http://127.0.0.1/formroutenew?html=rbac/role-list&query=graphuserlist&page=1&rows=8&org=sys
    
    资源维护
    http://127.0.0.1/formroutenew?html=rbac/resource-tree&query=graphorglist&page=1&rows=8
    
2017-03-23
角色列表

组织机构
http://127.0.0.1/formroutenew?html=rbac/group-tree&query=graphuserlist&page=1&rows=8
用户列表
http://127.0.0.1/formroutenew?html=rbac/user-list&query=graphuserlist&page=1&rows=8&org=sys

2017-03-22
    用户增删改查；用户与角色，用户与组织机构关系处理；
    组织结构增删改查；组织机构与角色，组织机构与用户关系处理；
    

    组织机构关联用户查询测试
    http://127.0.0.1/formroutenew?html=rbac/user-list&query=graphuserlist&page=1&rows=8&org=sys
    
    todo: 首页预加载，后续页面 JS 渲染；
    
2017-03-21
     前后端清除 children 双重保护机制；过多的数据会导致 lua 转换错误；
     
     
2017-03-20
     luar array to tree 
     调试 URI:   http://127.0.0.1/formroutenew?html=route-neo4j-json&query=graphorglist&page=1&rows=8
     用末模板引擎替换apoc.convert.toTree(ps)进行数据结构的转换
 
    
     
     --为了提升效率 用 lua - arraytotree 替换 js 进行数据结构转换 
            
      loadFilter: function(rows){
          return convert(rows);
      }
      
      <script type="text/javascript">
      
          //array to tree json
          function convert(rows){
      
              //父节点是否存在
              function exists(rows, parentId){
                  for(var i=0; i<rows.length; i++){
                      if (rows[i].group_id == parentId) return true;
                  }
                  return false;
              }
      
              var nodes = [];
              // get the top level nodes  得到顶级节点
              for(var i=0; i<rows.length; i++){
                  var row = rows[i];
                  if (!exists(rows, row.pid)){
                      nodes.push({
                          id:row.group_id,
                          name:row.name,
                          createDate:row.createDate
                      });
                  }
              }
      
              console.log("top nodes:");
              console.log(nodes);
      
              var toDo = [];
              for(var i=0; i<nodes.length; i++){
                  toDo.push(nodes[i]);
              }
              //多父节点处理
              while(toDo.length){
                  var node = toDo.shift();	// the parent node  shift() 方法用于把数组的第一个元素从其中删除，并返回第一个元素的值。
                  // get the children nodes
                  for(var i=0; i<rows.length; i++){
                      var row = rows[i];
                      if (row.pid == node.id){
                          var child = {id:row.group_id,name:row.name,createDate:row.createDate};
                          if (node.children){
                              node.children.push(child);
                          } else {
                              node.children = [child];
                          }
                          toDo.push(child);
                      }
                  }
              }
              console.log("result nodes:");
              console.log(nodes);
              return nodes;
          }
      
      </script>
            
2017-03-15
    用户管理完工
    http://127.0.0.1/formroutenew?html=rbac/user-list&query=graphuserlist&page=1&rows=8
    
    
    开始组织机构维护
    http://127.0.0.1/formroutenew?html=rbac/group-tree&query=graphorgtree&page=1&rows=8
    
2017-03-10
    CRUD参数数据为空的兼容处理；
    难点在于模板是 Lua语法，查询是 neo4j 语法，配置文件是 Json 语法。
    
    
2017-03-09
    及时编辑存储：
        <a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-add" plain="true" onclick="javascript:$('#detailTab').edatagrid('addRow')">New</a>
        <a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-remove" plain="true" onclick="javascript:$('#detailTab').edatagrid('destroyRow')">Destroy</a>
        <a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-save" plain="true" onclick="javascript:$('#detailTab').edatagrid('saveRow')">Save</a>
        <a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-undo" plain="true" onclick="javascript:$('#detailTab').edatagrid('cancelRow')">Cancel</a>


        <script src="easyui/jquery.edatagrid.js"  type="text/javascript"></script>

        $(function(){
            $('#detailTab').edatagrid({
                url: 'get_users.php',
                saveUrl: 'save_user.php',
                updateUrl: 'update_user.php',
                destroyUrl: 'destroy_user.php'
            });
        });
        
2017-03-08
    在三个层面应用了模板引擎：查询参数处理、输出结果处理、页面渲染处理
    使用模板lu引擎对查询参数进行处理；可以灵活的构建参数；（查询模板，获取的 get+post 参数）
    之前对输出结果进行模板参数处理；可以灵活进行结果处理；（输出结果，组装成需要的格式）
    对页面渲染进行模板处理；可以提取框架以及组件；（公用框架及组件）
    
    http://127.0.0.1/formroutenew?html=route-neo4j-debug&query=graphuserlist&searchvalue=&page=1&rows=8
    
    
    
2017-03-07
    需要支持 post,安全规范需要
            http://127.0.0.1/formroutenew?html=route-neo4j-json&query=graphuserlist
            数据在 body  page=1&rows=8

    合并 get 与 post 请求的参数
    local args = ngx.req.get_uri_args()
    ngx.req.read_body()
    local postargs = ngx.req.get_post_args();
    log.debug(cjson.encode(postargs))
    for k,v in pairs(postargs) do args[k] = v end    
    
    组合查询支撑？

2017-03-06
    调试用户页面json
    http://127.0.0.1/formroutenew?html=route-neo4j-debug&query=graphuserlist&&page=1&rows=8
    http://127.0.0.1/formroutenew?html=route-neo4j-json&query=graphuserlist&page=1&rows=8
    调试用户页面
    easyui page 参数处理
    http://127.0.0.1/formroutenew?html=rbac/user-list&query=graphuserlist&namevalue=%E5%9C%B0%E5%8C%BA&page=1&rows=8
    
2017-03-01
    思维导图整合 rbac 完成页面的布局以及初步的页面框架。   
    
2017-02-15
    route-form lua 的 oo优化；
    http://127.0.0.1/formroutenew?html=route-neo4j-json&query=neo4jsimple&namevalue=%E5%9C%B0%E5%8C%BA&_dc=1467884593669&sort=name%2CASC&sort=code%2CDESC&page=0&filter=[{%22type%22:%22string%22,%22value%22:%22test%22,%22field%22:%22name%22},{%22type%22:%22string%22,%22value%22:%22test%22,%22field%22:%22code%22}]&start=0&size=23
    
2017-02-09
    route-form 支持 neo4j-rest
    增加lua log
    lua 代码优化为module
    
    
基于api的快速框架
=========================


[TOC]

#应用场景
rails 的架构风格
api 提供json数据，使用模本引擎解析；
lua-resty-template 模板引擎，使用layout及组件进行页面组装；
lua 进行A/B测试配置；


##变更日志
###2016-12-20
* 门户菜单
  http://127.0.0.1/formroutequery?html=manage/my-treegrid&query=menures&page=0&size=3&pid=resource

###2016-11-29
*增加ssl 访问端口 curl https://127.0.0.1/

###2016-11-14
* apk add --no-cache   bash  curl  wget openssl


###2016-11-12
* 提供ssl 服务
# 生成一个RSA密钥 
$ openssl genrsa -des3 -out mo.key 1024
# 拷贝一个不需要输入密码的密钥文件
$ openssl rsa -in mo.key -out mo_nopass.key
# 生成一个证书请求
$ openssl req -new -key mo.key -out mo.csr
# 自己签发证书
$ openssl x509 -req -days 365 -in mo.csr -signkey mo.key -out mo.crt


###2016-10-30
*   extjs treegrid spring-data-rest 整合；
*   一是整合127.0.0.1:9006；
'
upstream spring_rest {
    server 192.168.1.183:9006;
      ip_hash;
}
location /form {
  proxy_pass http://spring_rest/form;
}
'
*   二是引入routeform：解决query 路径与返回json问题；



###2016-10-14
*   门户菜单功能
*   http://127.0.0.1/manage/portal/drag.html
*   http://127.0.0.1/manage/portal/ResourceTree.html
*   http://127.0.0.1/formroutequery?html=manage/my-ztree&query=menures&page=0&size=3
        data: {
            simpleData: {
                enable: true,
//                idKey: "module",
//                pIdKey: "pid",
                rootPId: 0
            }
        },
        
            var zNodes =
                    //{*cjson.encode(formvalue[formvalue.params.query]._embedded.resource)*};
*       //编辑状态不能点击链接跳转
                    

###2016-09-10
*   优化整理web-apis架构
*   http://127.0.0.1/formroutequery?html=web/web-index&query=users&page=0&size=3
*   http://127.0.0.1/formroutequery?html=manage/my-users&query=users&query=avatar&page=0&size=3
*   http://127.0.0.1:9006/form/rest/user



##主要功能


###配置文件 conf/nginx
####配置文件 conf/nginx/nginx.conf
-   全局的nginx配置文件
-   如果没有特殊需求，可依据模板变动并且启用
-   主要是通过include整合其他文件

#### conf/nginx/http.d
-   http 通用设置：1-http-ext-common.conf
-   redis通用设置：2-upstream-common.conf
-   lua配置文件，waf配置；lua缓存是否生效;配置文件字典；常用lualib；lua global 函数与变量;lua worker心跳：http-lua.conf

#### conf/nginx/server.d
-   http 访问路径配置：
-   route-form.conf rails风格引擎；
    ```
    http://127.0.0.1/formroutequery?html=route-json&query=users&query=avatar&page=0&size=3
    html 输出模板,使用lua-resty-template模板引擎，根据json value组装输出结果,格式是json/text/html；
    query 查询api;在query.d/0-config-db-query.json在定义查询api；支持spring-boot api 与 elastcisearch api ;支持多个query查询，参数在多个query api中可复用；
    
    ```
-   lua-proxy-extjs.conf  增删改api提交的反向代理；
-   lua-template.conf 模板引擎的配置

###配置文件 conf/lua.app
lua 应用目录

####配置文件 conf/lua.app/lua.d
-   lua 引擎文件
-   全局变量、全局函数 0-global-init.lua
-   心跳  0-init_worker.lua
-   反向代理路径配置 0-init_worker.lua
-   elasticsearch查询filter支持 4-extjs4.lua
-   输出内容加工 6-body-filter-extjs.lua
-   rails风格引擎 route-form.lua

####配置文件 conf/lua.app/query.d
-   API查询配置文件 0-config-db-query.json
-   权限配置文件  0-config-db-acl.json

####配置文件 conf/lua.app/third.lua.lib
-   cjson lua 库
-   httpc   lua 库
-   template lua 库

####配置文件 conf/lua.app/templates
-   布局文件目录 layouts
-   管理平台模板文件 manage
-   前端暂时页面模板文件 web

####配置文件 logs
-   日志文件目录

####配置文件 www
-   管理平台资源文件
-   展示平台资源文件
-   上传文件目录


###运行环境 fig.yml

####web
-   采用alpine for waf 镜像，尺寸小，时区与中文支持环境支持；
-   设置80端口提供访问，如果本机已被占用，可更改为别的端口；
-   绑定运行环境目录volumes_from，使得运行环境可配置；
-   链接app 与 redis , 使得配置文件对于app与redis的引用营销；
-   www/manage/js/plugins/datatables/editor 需要自行下载

####dataWeb
-   进行日志路径配置链接到本地路径，使得容器日志可查看，方便调试；
-   进行运行参数文件链接到本地文件，使得同一个镜像文件可以应用于不同的业务应用；

####app
-   spring-boot 单个实例；可配置为集群再使用；

####redis
-   使用ap-redis,尺寸较小；
-   提供tomcat session 共享存储；
-   也可提供nginx 共享存储；
-   业务如果数据量大，建议单独建立redis镜像集群；



##测试
###测试

[启动rest-api]((https://github.com/supermy/rest-api))，为应用提供[api数据源](http://127.0.0.1:9006/form/rest/user)；
进入到fig.xml 目录执行以下命令进行测试,使用浏览器访问；

    fig up -d && fig ps
    编排api http://127.0.0.1/formroutequery?html=route-json&query=users&query=avatar&page=0&size=3
    管理页面 http://127.0.0.1/formroutequery?html=manage/my-users&query=users&query=avatar&page=0&size=3
    展示页面 http://127.0.0.1/formroutequery?html=web/web-index&query=users&page=0&size=3
    fig stop && fig rm -v --force
    