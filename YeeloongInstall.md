# Yeeloong笔记本安装方法 #

1. 启动旧系统, 其中hda5是旧根系统, hda6挂载到/home. 我们将弦歌系统安装在hda6上, 将旧文件拷贝到hda5的/home即可.

2. 修改boot.cfg. 笔记本的启动配置在/dev/hda1/boot.cfg. 挂载并修改, 加入弦歌启动项.

3. 将文件解压到/home目录.

4. 按文档配置.

5. 重新启动.