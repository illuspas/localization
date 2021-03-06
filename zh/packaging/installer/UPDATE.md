# 更新Netdata

![image8](https://cloud.githubusercontent.com/assets/2662304/14253735/536f4580-fa95-11e5-9f7b-99112b31a5d7.gif)

**Warning** This page is outdated. Please refer to the English version and help provide an up to date translation.

我们建议你使用最新版的Netdata，我们正在不断地完善它。

更新过程取决于您的安装方式：

## 如果您是使用Git命令从GitHub中下载的

### 手动更新

当你安装时，脚本 `netdata-installer.sh` 会在你下载目录中生成一个 `netdata-updater.sh` ,你可以使用这一脚本来更新Netdata，它将使用与安装脚本相同的选项
您只需要运行它，他便会自动下载并安装最新版的Netdata，同时，你还可以将这一个脚本放置在Cronjob中以定期自动更新Netdata

```sh
#转到Git的下载目录
cd /path/to/git/downloaded/netdata

#运行升级脚本
./netdata-updater.sh
```

_Netdata将会以新的版本运行_

如果你没有这一个脚本(比如：你删除了Netdata的下载目录), 请按照 **[安装]** 说明进行操作。请放心，安装程序会保留您的配置。您也可以手动将Netdata更新到最新版本。请按照如下命令进行操作：

```sh
#转到Git的下载目录
cd /path/to/git/downloaded/netdata

#下载最新的版本
git pull

#这个命令将会重建，安装，并运行Netdata
./netdata-installer.sh
```

_Netdata将会以新的版本运行_

请注意，Netdata现在可能具有一些新功能，或者某些旧功能现在可能表现不同。所以在更新后要注意这些变化。

### 自动升级

_请考虑自动更新的风险。或许有些事情会出错。密切关注它，如果出现故障，请进行手动更新。_

你可以在cron-job中调用`netdata-updater.sh`。但是，请注意：成功更新后不会有任何的提示。

```sh
#编辑你的cron-jobs
crontab -e

#在文件的末尾添加一个新的cron-job命令（如下）。它将在每天的06：00更新Netdata：
#更新Netdata
0 6 * * * /path/to/git/downloaded/netdata/netdata-updater.sh
```

## 如果您使用二进制包安装

如果您使用二进制包安装Netdata，最好的方法是从安装源获取一个新的安装副本。

如果您找不到最新版本的Netdata，我们建议您卸载现在版本，并按照**[安装]**说明来安装较新版本的Netdata。








[![analytics](https://www.google-analytics.com/collect?v=1&aip=1&t=pageview&_s=1&ds=github&dr=https%3A%2F%2Fgithub.com%2Fnetdata%2Fnetdata&dl=https%3A%2F%2Fmy-netdata.io%2Fgithub%2Finstaller%2FUPDATE&_u=MAC~&cid=5792dfd7-8dc4-476b-af31-da2fdb9f93d2&tid=UA-64295674-3)]()
