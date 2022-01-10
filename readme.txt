
1、在本地创建一个版本库（即文件夹），通过git init把它变成Git仓库;
2、把项目复制到这个文件夹里面，再通过git add .把项目添加到仓库；
3、再通过git commit -m "注释内容"把项目提交到仓库；
4、在Github上设置好SSH密钥后，新建一个远程仓库，通过git remote add origin xxx（复制项目中Clone or download地址）将本地仓库和远程仓库进行关联；
若github仓库中有文件，合并使用git clone https://github.com/LostStata/Openoppo.git
5、最后通过git push -u origin main把本地仓库的项目推送到远程仓库（也就是GitHub）上；（若新建远程仓库的时候自动创建了README文件会报错，解决办法看上面


上传文件前出现
fatal: unable to access 'https://github.com/LostStata/Openoppo.git/': OpenSSL SSL_read: Connection was reset, errno 10054
打开Git命令页面，执行git命令脚本：修改设置，解除ssl验证
git config --global http.sslVerify "false"
重新git add .
提交git commit -m "提示信息"
git push -u origin main