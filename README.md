![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 查找孤立的数据文件
#### Find Orphaned Data Files

![#](images/##############?raw=true "#")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
通过这些SQL逻辑（Logic），你将能够读取所有数据文件位置（在操作系统中），并查看哪些文件与任何实时数据库无关。这将帮助你查找孤立文件，或一次性备份，分离的数据文件，或几乎与数据文件位于同一文件夹下的任何其他
“文件”。 这真的很有用。
这是它的工作原理。 它使用sys.master_files构建一个你所有的数据文件路径的列表。然后，它使用这些路径来推断这些驱动器位置下的所有已知文件。然后在几个临时表之后，它对数据文件路径位置上的文件与实际连接到数据库的文件进行比较。你将得到
那些与任何数据库无关的文件，它们是真正的孤立数据库文件，无关的安全文件，随机备份文件或安装SQL Server时常用的内置文件，例如系统资源数据库，复制等。
在逻辑（logic）的底部，你可以看到我删除了其中一些。 你可以随意进行修改，或添加。 这非常适合高频使用的环境，在这些环境中会有大量分离的数据文件，备份文件，复制的拉链（一种程序压缩的档案文件格式
）。
下面是代码。



## English
With this little bit of SQL Logic you’ll be able to read across ALL your data file locations (in the OS) and see what files are NOT associated with any live databases. This will help you find orphaned files, or any one-off backups, detached data files, or virtually any other ‘files’ which are located under the same folder as your data files. It’s really useful.

Here’s how it works. It builds a list of ALL your data file paths using sys.master_files. Then it uses those paths to extrapolate all known files under those drive locations. Then after a few temporary tables it runs a comparison between files that on the data file path locations and those that are actually connected to databases. What you get following that are files that are not associated to any databases; therefore they are either genuine orphaned database files, extraneous security files, random backup files or the usual built-in files you get upon installation of SQL Server such as for the system resource databases, replication etc.

At the bottom of the logic you can see where I am excluding some of those. Feel free to mod, or add your own. This is perfect for highly used environments where there is a multitude of detached data files, backup files, copied zips what have you.
Here’s the code.



---
## Logic
```SQL
use [master];
set nocount on
 
if object_id('tempdb..#paths') is not null
drop table  #paths
create table    #paths ([path_id] int identity (1,1), [data_paths] varchar(255))
insert into #paths ([data_paths])
 
select distinct left([physical_name], len([physical_name]) - charindex('\', reverse([physical_name])) -0)
from sys.master_files
 
if object_id('tempdb..#found_files') is not null
    drop table  #found_files
create table    #found_files ([files] varchar(255), [file_path] varchar(255), [depth] int, [file] int)
 
declare @get_files  varchar(max)
set @get_files  = ''
select  @get_files  = @get_files +
'
insert into #found_files ([files], [depth], [file]) exec master..xp_dirtree ''' + [data_paths] + ''', 1,1;
update #found_files set [file_path] = ''' + [data_paths] + ''' where [file_path] is null;
' + char(10) from #paths
exec    (@get_files)
 
select
    'no_associated_database'= [files]
,   'path'          = [file_path]
from 
    #found_files
where
    [files] not in (select right([physical_name], charindex('\', reverse([physical_name])) - 1) from sys.master_files)
    and [files] not in
    (
        'mssqlsystemresource.mdf'
    ,   'mssqlsystemresource.ldf'
    ,   'distmdl.mdf'
    ,   'distmdl.ldf'
    )
    and [files] not like '%.cer'
```
希望对你有用。 (Hope you find it helpful.)


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

