# non-official-wow
non-official wow server base on TrinityCore

## 0. 说明
基于TrinityCore源码v3.3.5版本搭建Ubuntu环境下的WOW服务器，仅供个人学习C++使用。

## 1. 安装依赖
```
apt-get install git cmake make gcc g++ libmysqlclient-dev libssl-dev libbz2-dev libreadline-dev libncurses-dev libboost-all-dev mysql-server p7zip

```

## 2. 下载源码
> 建议在Ubuntu下新建用户，隔离超级权限

```
add user ${wow_user}
su - ${wow_user}

cd {$work_space_path}
git clone -b 3.3.5 git://github.com/TrinityCore/TrinityCore.git
```

## 3. 编译源码
```
cd ${work_space_path}/TrinityCore
mkdir build
cd build
cmake ../ -DCMAKE_INSTALL_PREFIX=/home/${wow_user}/server -DTOOLS=1 
make
make install
```
> DTOOLS 参数用于指定是否编译地图导出工具，建议打开此开关

> DCMAKE_INSTALL_PREFIX 参数用于指定编译出的可执行文件的位置，可根据情况自己指定

> 其他参数详细含义详见： https://trinitycore.atlassian.net/wiki/display/tc/Linux+Core+Installation

## 4. 导出文件
> 此步操作需要客户端文件，须使用对应版本的客户端，建议版本：==v3.3.5-13930==或者==v3.3.5-12340==

> 由于此步操作需要上传客户端文件到服务器，并且导出时间比较长，可以直接使用已经导出的文件

### 4.1 导出dbc和maps
```
cd ${wow_client_path}
/home/${wow_user}/server/bin/mapextractor
cp -r dbc maps ${wow_server_data_path}
```

### 4.2 导出vmaps
```
cd ${wow_client_path}
/home/${wow_user}/server/bin/vmap4extractor
mkdir vmaps
/home/${wow_user}/server/bin/vmap4assembler Buildings vmaps
cp -r vmaps ${wow_server_data_path}
```

### 4.3 导出mmaps
```
cd ${wow_client_path}
mkdir mmaps
/home/${wow_user}/server/bin/mmaps_generator
cp -r mmaps ${wow_server_data_path}
```

### 4.4 修改配置文件
```
cd /home/{wow_user}/server/etc
cp authserver.conf.dist authserver.conf
cp worldserver.conf.dist worldserver.conf
```
> 根据实际情况修改两个配置文件中的相关配置项:

> LogsDir: 日志目录

> BindIp: 绑定IP，建议改为0.0.0.0

> LoginDatabaseInfo: 数据库连接信息

> WorldDatabaseInfo: 数据库连接信息

> CharacterDatabaseInfo: 数据库连接信息

> DataDir: 服务器数据路径，${wow_server_data_path}


## 5. 导入数据库
### 5.1 确保MySQL启动
### 5.2 确保MySQL的访问权限设置正确
### 5.3 以root用户登录MySQL
```
source ${wow_server_sql_path}/create_mysql.sql;

use auth;
source ${wow_server_sql_path}/auth_database.sql;

use characters;
source ${wow_server_sql_path}/characters_database.sql;

use world;
source ${wow_server_sql_path}/TDB_full_world_335.63_2017_04_18.sql;
```

### 5.4 修改配置
```
UPDATE auth.realmlist SET localAddress = '${wower_server_ip}' and gamebuild = '${wower_client_version}';
```

> ${wower_client_version} 为客户端的小版本号，如13930或12340

## 6. 启动服务
### 6.1 启动Auth服务
```
cd /home/${wow_user}/server/bin
./authserver
```

### 6.2 启动World服务
```
cd /home/${wow_user}/server/bin
./worldserver
```

### 6.3 创建账号
> 在World服务的命令行中执行以下命令：

```
account create ${account_name} ${account_pass}
```

> 添加GM权限：

```
account set gmlevel ${account_name} 3 -1

```

## 7. 启动客户端
### 7.1 修改客户端配置
```
${wow_client_path}/Data/zhCN/realmlist.wtf
```
> 把其中的服务器地改为${wow_server_ip}

### 7.2 启动客户端程序Wow.exe
