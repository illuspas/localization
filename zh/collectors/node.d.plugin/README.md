# node.d.plugin

`node.d.plugin` 是一个netdata扩展插件。它是一个用`node.js`编写的数据收集模块的**编配器**。

1. 它作为一个独立的进程运行 `ps fax` 可以看到它
2. 它由netdata自动启动和停止
3. 它通过单向管道与netdata通信 (向netdata守护进程发送数据)
4. 支持任意数量的数据采集**模块**
5. 允许每个**模块**具有一个或多个数据收集**作业**
6. 每个**作业**从单个数据源收集一个或多个指标

## Node.js插件的PR清单

这是为Netdata提交一个新的Node.js插件的通用检查列表。但并不意味着是全面的。

至少，为了可构建和可测试，PM需要包括:

* 模块本身, 遵循正确的命名约定: `node.d/<module_dir>/<module_name>.node.js`
* 一个为插件编写的README.md文件。
* 模块的配置文件
* 插件在适当的全局配置文件中的基本配置: `conf.d/node.d.conf`， 也是以JSON格式。 如果默认情况下应启用该模块，请在`modules`字典中为其添加一个部分。
* A line for the plugin in the appropriate `Makefile.am` file: `node.d/Makefile.am` 在 `dist_node_DATA`下方.
* A line for the plugin configuration file in `conf.d/Makefile.am`: 在`dist_nodeconfig_DATA`下方
* 可选，图表信息在`web/dashboard_info.js`。这通常涉及为该部分指定名称和图标，并可能包括对该部分或单个图表的描述。

## 动机

Node.js非常适合异步操作。 它非常快速且非常普遍（实际上整个网络都基于它）。  
由于数据收集不是CPU密集型任务，node.js是一个理想的解决方案。

`node.d.plugin`是一个netdata插件，它提供了一个抽象层，可以方便快捷地开发node.js中的数据收集器。  
它还使用单个node实例管理所有数据收集器（放在`/usr/libexec/netdata/node.d`中），
从而降低了数据收集的内存占用。

当然，也可以用node.js编写独立的插件（放在`/usr/libexec/netdata/plugins`中）。
这些必须使用 **[扩展插件](../plugins.d/)** 的指南来开发。

要运行`node.js`插件，你需要在系统中安装`node`。

在一些较旧的系统中，名为`node`的包不是node.js. 它是一个名为`ax25-node`的终端仿真程序。  
在这种情况下，node.js包可以称为`nodejs`。 一旦你安装了`nodejs`，我们建议链接  
`/usr/bin/nodejs`到`/usr/bin/node`，这样在你的终端输入`node`就会打开node.js.  
有关更多信息，请查看 **[[Installation]]** 指南。

## 配置 `node.d.plugin`

`node.d.plugin`甚至可以在没有任何配置的情况下工作。 它的默认配置文件是
[/etc/netdata/node.d.conf](node.d.conf）（在你的系统上运行`/etc/netdata/edit-config node.d.conf`来编辑它）。

## 配置 `node.d.plugin` 模块

`node.d.plugin`模块接受`JSON`格式的配置。

不幸的是，`JSON`文件不接受注释。 因此，描述它们的最佳方法是使用带有说明的markdown文本文件。

`JSON` has a very strict formatting. If you get errors from netdata at `/var/log/netdata/error.log` that a certain
configuration file cannot be loaded, we suggest to verify it at [http://jsonlint.com/](http://jsonlint.com/).

`JSON`格式非常严格。 如果你在`/var/log/netdata/error.log`中收到netdata发出错误，无法加载某个配置文件，我们建议在[http://jsonlint.com/](http://jsonlint.com/)上验证它）。

此目录中的文件提供了用于配置每个`node.d.plugin`模块的可用示例。

## 调试为node.d.plugin编写的模块

要测试位于`/usr/libexec/netdata/node.d`中的`node.d.plugin`模块，可以手动运行`node.d.plugin`，   
像这样：

```sh
# become user netdata
sudo su -s /bin/sh netdata

# run the plugin in debug mode
/usr/libexec/netdata/plugins.d/node.d.plugin debug 1 X Y Z
```

`node.d.plugin`将以`debug`模式运行（大量调试信息），更新频率为`1`秒，  
仅评估收集器脚本`X`（即`/usr/libexec/netdata/node.d/X.node.js`），`Y`和`Z`。  
您可以定义零个或多个模块。 如果没有定义，`node.d.plugin`将评估所有可用的模块。

请记住，如果你的配置不在`/etc/netdata`中，你应该在运行`node.d.plugin`之前执行以下操作：

```sh
export NETDATA_USER_CONFIG_DIR="/path/to/etc/netdata"
```

---

## 开发`node.d.plugin`模块

你的数据收集模块应分为3部分：

   - 一个从其源获取数据的函数。 `node.d.plugin`已经可以从Web源获取数据，  
      因此你不需要为http做任何事情。

   - 一个用来处理获取/操作获取数据的函数。 此功能将进行多次调用  
      创建图表和维度并将收集的值传递给netdata。  
      这是你收集http JSON数据时唯一需要编写的函数。

   - 一个`configure`和一个`update`函数，分别负责你的模块配置和数据刷新。  
      你可以使用已提供的。


你的模块将自动处理任意数量的服务，具有不同的设置（甚至不同的数据收集频率）。你将只编写一个工作所需的，`node.d.plugin`将完成剩下的工作。  
你必须为你获取数据的每个服务，创建一个`service`（稍后介绍）。

### 编写数据收集模块

要提供一个名为`mymodule`的模块，你可以使用以下结构创建文件`/usr/libexec/netdata/node.d/mymodule.node.js`：
```js

// the processor is needed only
// if you need a custom processor
// other than http
netdata.processors.myprocessor = {
	name: 'myprocessor',

	process: function(service, callback) {

		/* do data collection here */

		callback(data);
	}
};

// this is the mymodule definition
var mymodule = {
	processResponse: function(service, data) {

		/* send information to the netdata server here */

	},

	configure: function(config) {
		var eligible_services = 0;

		if(typeof(config.servers) === 'undefined' || config.servers.length === 0) {

			/*
			 * create a service using internal defaults;
			 * this is used for auto-detecting the settings
			 * if possible
			 */

			netdata.service({
				name: 'a name for this service',
				update_every: this.update_every,
				module: this,
				processor: netdata.processors.myprocessor,
				// any other information your processor needs
			}).execute(this.processResponse);

			eligible_services++;
		}
		else {

			/*
			 * create a service for each server in the
			 * configuration file
			 */

			var len = config.servers.length;
			while(len--) {
				var server = config.servers[len];

				netdata.service({
					name: server.name,
					update_every: server.update_every,
					module: this,
					processor: netdata.processors.myprocessor,
					// any other information your processor needs
				}).execute(this.processResponse);

				eligible_services++;
			}
		}

		return eligible_services;
	},

	update: function(service, callback) {
		
		/*
		 * this function is called when each service
		 * created by the configure function, needs to
		 * collect updated values.
		 *
		 * You normally will not need to change it.
		 */

		service.execute(function(service, data) {
			mymodule.processResponse(service, data);
			callback();
		});
	},
};

module.exports = mymodule;
```

#### 配置(config)

当`node.d.plugin`启动时，`configure（config）`只被调用一次。  
配置文件将包含`/etc/netdata/node.d/mymodule.conf`的内容。  
该文件应具有以下格式：

```js
{
	"enable_autodetect": false,
	"update_every": 5,
	"servers": [ { /* server 1 */ }, { /* server 2 */ } ]
}
```



如果配置文件`/etc/netdata/node.d/mymodule.conf`没有给出`enable_autodetect`或`update_every`，  
这些将由`node.d.plugin`添加。所以你的模块将始终拥有它们。

配置文件`/etc/netdata/node.d/mymodule.conf`可能包含`mymodule`所需的任何其他内容。

#### 处理响应(data)

`data`可以是`null`或者`service`中处理规定的返回值。

`service`对象定义了一组函数，允许你将信息发送到netdata核心：

1. 图表和维度定义
2. 从收集的更新值

---

*FIXME: document an operational node.d.plugin data collector - 最佳用例是
[snmp collector](snmp/snmp.node.js)*

[![analytics](https://www.google-analytics.com/collect?v=1&aip=1&t=pageview&_s=1&ds=github&dr=https%3A%2F%2Fgithub.com%2Fnetdata%2Fnetdata&dl=https%3A%2F%2Fmy-netdata.io%2Fgithub%2Fcollectors%2Fnode.d.plugin%2FREADME&_u=MAC~&cid=5792dfd7-8dc4-476b-af31-da2fdb9f93d2&tid=UA-64295674-3)]()
