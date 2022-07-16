# 随机抽号器

#### 介绍
这是一个简单的随机抽号系统，类似于我们学生时代老师上课使用的，该程序仅限于抽数字，抽取的数字范围可以指定自定义。默认是0到50

#### 软件架构
采用python3.7+pyside2搭建的，64位的


#### 安装教程

1.在main.py文件所在的路径下打开cmd
2.执行以下命令：
`pyinstaller main.py --noconsole --hidden-import PySide2.QtXml --icon="random_logo.ico"`
即可将该程序打包为exe文件。

#### 使用说明

1.直接在main.py文件中使用python解释器运行即可
2.如果打包成的exe文件被某些杀毒软件给拦截了，选择跳过就好。