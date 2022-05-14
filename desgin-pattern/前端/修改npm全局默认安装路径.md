# 修改npm全局默认安装路径





查看当前npm配置路径

```shell
npm config ls
```

结果大致是这样

```shell
; cli configs
metrics-registry = "https://registry.npm.taobao.org/"
scope = ""
user-agent = "npm/6.14.8 node/v12.13.1 win32 x64"

; userconfig C:\Users\wstiehs\.npmrc
disturl = "https://npm.taobao.org/dist\u0002"
registry = "https://registry.npm.taobao.org/"

; globalconfig C:\Users\wstiehs\AppData\Roaming\npm\etc\npmrc
python = "C:\\Users\\wstiehs\\.windows-build-tools\\python27\\python.exe"

; builtin config undefined
prefix = "C:\\Users\\wstiehs\\AppData\\Roaming\\npm"

; node bin location = C:\Program Files\nodejs\node.exe
; cwd = C:\Users\wstiehs
; HOME = C:\Users\wstiehs
; "npm config ls -l" to show all defaults.
```



需在更换路径位置创建新的文件夹，名为nodejs，在nodejs创建两个文件夹：node_global(新增的全局)和node_cache(新增的缓存)

```shell
E:\nodejs\node_global

E:\nodejs\node_cache
```



修改npm路径，输入命令

```shell
npm config set prefix "E:\nodejs\node_global"

npm config set cache "E:\nodejs\node_cache"
```



再次查看配置路径，原始路径已修改为所设置的路径

```shell
npm config ls

; cli configs
metrics-registry = "https://registry.npm.taobao.org/"
scope = ""
user-agent = "npm/6.12.1 node/v12.13.1 win32 x64"

; userconfig C:\Users\wstiehs\.npmrc
cache = "E:\\nodejs\\node_cache"
disturl = "https://npm.taobao.org/dist\u0002"
prefix = "E:\\nodejs\\node_global"
registry = "https://registry.npm.taobao.org/"

; builtin config undefined

; node bin location = C:\Program Files\nodejs\node.exe
; cwd = C:\Users\wstiehs
; HOME = C:\Users\wstiehs
; "npm config ls -l" to show all defaults.
```



配置环境变量

修改系统变量中的path，添加刚才新建的文件路径 E:\nodejs\node_global