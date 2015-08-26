title: git常用指令
date: 2015-04-26 16:10:15
tags: [git]
---
<xmp>
git config --global user.name "name"
git config --global user.email email@email.com
git config --global alias.co checkout 
git init
git add -A 
git commit -m "message"

git remote add origin git@gitservicename:username/reponame.git

git remote rm origin

git push -u origin master 推送master分支
git push -u origin --all  推送所有分支
-u表示追踪分支，下次可无参使用git push

git clone url

ssh-keygen 三次空格，.ssh/id_rsa.pub下为key
</xmp>