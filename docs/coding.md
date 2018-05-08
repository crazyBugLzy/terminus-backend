# 开发规范

良好的代码规范可以提高团队编码、review的效率，保证项目质量，端点有一个[总的开发规范](http://docs.terminus.io/backend/standard/)。

每个项目也在此基础上制定更细化的开发规范。

## 代码规范
### 基础代码规范配置

* 代码编写完,使用IDEA自带格式化代码快捷格式化下代码,mac默认快捷键: ALT + COMMAND + L
* 安装lombok插件，简化set、get方法

### 使用DSL风格代码模板和避免空字符打印

* 右键选择Generate,选择Setter选项

	![5acf5f5a0383c.png](https://i.loli.net/2018/04/12/5acf5f5a0383c.png)
	
* 选择新建DSL风格Template,伪代码如下

![5acf6005a1b59.png](https://i.loli.net/2018/04/12/5acf6005a1b59.png)

		#set($paramName = $helper.getParamName($field, $project))
		public ##
		#if($field.modifierStatic)
		static void ##
		#else
		    $classname ##
		#end
		$StringUtil.sanitizeJavaIdentifier($helper.getPropertyName($field, $project))($field.type $paramName) {
		#if ($field.name == $paramName)
		    #if (!$field.modifierStatic)
		    this.##
		    #else
		        $classname.##
		    #end
		#end
		$field.name = $paramName;
		#if(!$field.modifierStatic)
		return this;
		#end
		}
	
	
> model方法中配合静态create方法，使用DSL风格可以相当简洁明了的进行对象的创建赋值工作，如下所示:
 
	    public static MemberProfile create() {
	        return new MemberProfile();
	    }
	    public MemberProfile id(Long id) {
		    this.id = id;
		    return this;
		}
	
	    public MemberProfile nickname(String nickname) {
	        this.nickname = nickname;
	        return this;
	    }
	    MemberProfile.create().id(1L).nickname("test")
	    
* 同样的方式重新建立toString模板，避免属性为空的输出

		public String toString() {
		#set ($autoImportPackages = "com.google.common.base.MoreObjects")
		return MoreObjects.toStringHelper(this)
		#foreach ($member in $members)
		.add("$member.name", $member.accessor)
		#end
		.omitNullValues()
		.toString();
		}

### checkStyle的使用

* 在idea中选择File->Setting->Plugins，搜索checkStyle-IDEA插件，点击安装
* 在File->Other Settings里面配置添加checkstyle.xml(checkstyle.xml见git仓库)，
下载地址：https://space.dingtalk.com/s/gwHOABHuiwLOAQtfDAPaACBiNGYzYmM3NTE0YjU0NzRlOTQxMjRiNDVkY2FhZjliZQ 
密码: uJCm
如下图
![5acf5164b4b95.png](https://i.loli.net/2018/04/12/5acf5164b4b95.png)

### Swagger的使用
Swagger可以使人和计算机在看不到源码或者看不到文档或者不能通过网络流量检测的情况下能发现和理解各种服务的功能，更加容易理解和调用接口，如下图:
![5acf4e2458a95.png](https://i.loli.net/2018/04/12/5acf4e2458a95.png)

> maven中直接添加如下依赖即可:

			<dependency>
				<groupId>io.swagger</groupId>
				<artifactId>swagger-annotations</artifactId>
				<version>1.5.10</version>
			</dependency>	
			
			<dependency>
				<groupId>io.terminus.boot</groupId>
				<artifactId>terminus-spring-boot-starter-swagger</artifactId>
				<version>1.6.2.BUILD-SNAPSHOT</version>
			</dependency>

## 流程规范

### 合并多个commit为一个commit

目前在分支开发功能时，可能会提交多次，便会造成分支上面有多个commit。这时不论是使用cherry-pick移植功能，还是rebase，都会比较繁琐。所以要求大家分支合并之前都合并为一个提交。

* 在sourceTree上选择分支，将当前分支备份
	![5acf549dcef57.png](https://i.loli.net/2018/04/12/5acf549dcef57.png)
* 切换原有分支，sourceTree勾选当前分支，仅首个父级
	![5acf551d02e48.png](https://i.loli.net/2018/04/12/5acf551d02e48.png)
* 找到该分支最初提交commit的上一个节点，sourceTree上右键，选择将分支重置到该节点提交
	![5acf55fb66768.png](https://i.loli.net/2018/04/12/5acf55fb66768.png)
* 使用混合合并模式，点击确定

	![5acf55c08b9a7.png](https://i.loli.net/2018/04/12/5acf55c08b9a7.png)
* 将所有本地变动文件提交，并使用git push -f命令，强制将本地分支提交替代远程分支

> 由于分支回退到最初节点之后，由于所有的修改记录文件都已经保存在本地，混合合并不会造成修改文件丢失。此时再将文件提交，所有的修改就变成一个commit，使用-f命令，即用本地分支替代了远程分支。

### git rebase操作

Git 作为分布式版本控制系统，所有修改操作都是基于本地的，在团队协作过程中，假设你和你的同伴在本地中分别有各自的新提交，而你的同伴先于你 push 了代码到远程分支上，所以你必须先执行 git pull 来获取同伴的提交，然后才能 push 自己的提交到远程分支。而按照 Git 的默认策略，如果远程分支和本地分支之间的提交线图有分叉的话，Git 会执行一次 merge 操作，因此产生一次没意义的提交记录，进而会造主线分支的杂乱。故当前要求大家提交到主线的时候，必须做一次rebase操作

> 假设需要rebase的分支为feature/demo,主线为master(第一次rebase的时候，建议备份下自己的分支)

* git pull master获取最新主线代码
* 切到feature/demo分支，使用git rebase master命令
* 如有冲突，则解决完冲突，git add .暂存下(不需要提交)，若无冲突，则执行 git rebase --continue
* rebase完成后，这个时候本地分支会与远程分支有版本冲突，不用管，直接git push -f




