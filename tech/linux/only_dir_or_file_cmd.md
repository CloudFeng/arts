# linux系统只显示文件目录或文件

## 只显示文件目录

- 使用`ls`和`grep`命令: `ls -l | grep '^d'` 或者 `ls -l egrep '^d'`

- 仅使用`ls`命令: `ls -d */`

- 使用`find`命令： `find . -maxdepth 1 -type d`

## 只显示文件

- 使用`ls`和`grep`命令: `ls -l | egrep -v '^d'`或 `ls -l | egrep -v '^d'`


写于：2019-03-27 周三