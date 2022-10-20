# CoderRoad

**git上传：**

```bash
git add .
git commit -a -m "tips"
git push -u origin main
```







| 命令                        | 作用                                                         |
| --------------------------- | ------------------------------------------------------------ |
| git status                  | 查看本地库的状态                                             |
| git add [file]              | 把工作区的修改添加到暂存区                                   |
| git add .                   | 添加当前目录中的所有修改到暂存区                             |
| git rm --cached [file]      | 将一个未tracking文件从暂存区撤销修改  (都可以撤销)           |
| git restore --staged [file] | 将一个tracking文件从暂存区撤销修改  （都可以撤销）           |
| git restore [file]\|        | 将工作区的修改撤销                                           |
| git checkout -- [file]      | 从本地库检出最新文件覆盖工作区的文件(文件还没有提交到暂存区, 否则无效) |
| **git commit -a -m “说明”** | 用于将tracked file添加到本地库  （只能提交 tracked file）    |
| git commit –m “说明” [file] | 将暂存区的修改提交到本地库                                   |

```bash
ssh-keygen -t rsa -C "383205969@qq.com"

echo "# CoderRoad" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:ybllcodes/CoderRoad.git
git push -u origin main
```



```bash
git remote add origin https://github.com/ybllcodes/CoderRoad.git
git branch -M main
git push -u origin main
```



