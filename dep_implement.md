## 1. 显示依赖关系 ##

启动命令: gpkg -p package
全局变量: GPKG\_DEP\_LIST

1. 置空 GPKG\_DEP\_LIST
2. 调用函数 gpkg\_get\_dep package
3. 退出


### 1.1 gpkg\_get\_dep package ###

这个函数打印软件包的依赖关系，可能被递归调用。

  * 1. 获取package对应的xgb文件，放在$xgbfile中，并设置对应的$N, $V, $T, $R
  * 2. 如果软件包已在 GPKG\_DEP\_LIST中， 返回0。
  * 3. 如果软件包已经安装，返回0.
  * 4. 将软件包加入GPKG\_DEP\_LIST
  * 5. 包含$xgbfile文件，把$DEP变量复制到局部变量 $deps 中。
  * 6. 对$deps中的每个软件包， 递归调用 gpkg\_get\_dep subpackages
  * 7. 结束。


## 2. 带依赖关系的安装 ##
启动命令: gpkg -i package

  * 1. 调用 gpkg\_install\_package package.
  * 2. 结束

## 2.1 gpkg\_install\_package ##

这个函数安装实际的软件包，可能被递归调用.

  * 1. 获取package对应的xgb文件，放在$xgbfile中，并设置对应的$N, $V, $T, $R
  * 2. 如果软件包已经安装，返回0.
  * 3. 包含$xgbfile文件，把$DEP变量复制到局部变量 $deps 中。
  * 4. 对$deps中的每个软件包， 递归调用 gpkg\_install\_package subpackage
  * 5. 安装实际的软件包
