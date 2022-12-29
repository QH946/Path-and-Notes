1.先到gitee新建一个远程仓库

==创建仓库时，不选分支==

2.新建新建一个文件夹作为你的本地仓库

3.初始化仓库
==git init==
4.初始化本地仓库成功后回到我们的gitee，gitte复制拿到我们的远程仓库的访问路径

==git remote add origin== https://gitee.com/charken/elm-ssm.git
5.下拉远程仓库的master分支到本地仓库，然后本地仓库合并远程仓库的分支
==git pull origin master==

6.修改.gitignore文件，让它过滤我们想过滤的文件或文件夹
target/

#过滤Eclipse里用不到的文件或文件夹
#Eclipse#
.classpath
.project
.settings/

#过滤IntelliJ IDEA里用不到的文件或文件夹
#IntelliJ IDEA#
*.idea
*.iml
*.ipr
*.iws

#gitee里勾选了自动生成Maven的过滤文件，下面的就是自动生成的那个过滤文件要过滤的文件或文件夹
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties
.mvn/wrapper/maven-wrapper.jar

7.把项目拷贝进本地仓库

==cv==

8.把代码添加进暂存区
==git add .==

9.把代码提交到本地仓库
==git commit -m '初次提交'==

10.把代码从本地仓库推送到远程仓库
==git push origin master==



//遇到Push to origin/master was rejected且无法解决时，启用强制推送
git push -f origin master



### 



