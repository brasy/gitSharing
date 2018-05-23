## Git migration, how to use subgit

###Linux下修改/设置环境变量JAVA_HOME：
1. 下载 jdk: jdk-8u171-linux-x64.tar.gz
2. for a user, set env for ever
    //modify user directory files: .bash_profile
     $ vi /home/myuser/.bash_profile

    //add
     export JAVA_HOME = /home/myuser/jdk1.8.0_171
     export PATH = $JAVA_HOME/bin:$PATH
     export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

     #source /home/myuser/.bash_profile

[xying@xxxx ~]$ cat .bash_profile 
...



###Migrantion remote svn to local git

1. download subgit.
Unzip it to your home directory: unzip subgit-3.3.1.zip

2. Create configuration:
./subgit-3.3.1/bin/subgit configure https://xxx.svn.net/source/PROJECT/branches  gitrepo.git

(1)limit versions which are imported by using option --minimal-revision REV.

(2)make automatic mapping from SVN committers to Git committers, download authors.txt file and copy it to gitrepo.git/subgit directory.
Download passwd file and copy it to gitrepo.git/subgit directory. Or you can edit current gitrepo.git/subgit/passwd file and add SVN user name with passwd to it.

(3)Edit gitrepo.git/subgit/config file.
Modify default domain: defaultDomain = domain.com

Exclude unwanted branches, for example:
    excludeBranches = branches/xx*

Add "unstandard" directories (by default these are included: trunk, branches, tags, shelves):
    branches = tst/*:refs/heads/tst/*

config: how branches are taken to Git, what is exluded / included, what kind of directory structure is in Git. Below is examples how you can configure trunk and branches.

    trunk = branch1:refs/heads/master
    branches = branch2:refs/heads/branch2

    trunk = tst/branch1:refs/heads/master
    branches = tst/branch2:refs/heads/branch2

notes: branch1 branch in SVN should be taken as master branch in Git. branch2 branch in SVN should go to branch2 branch in Git.
Third and fourth line tells that MT cases in tst/branch1 branch should taken as a master branch in Git and MT cases in tst/branch2 should put to branch2 branch in Git.

(4)Then start conversion:
./subgit-3.3.1/bin/subgit import gitrepo.git

Conversion might take long time, depending on repository size. You can interrupt conversion, it is able to continue when it is next time started. 



###Migrantion remote svn to local svn, then to git
With some program blocks, SubGit import fails to corrupted repository announcement. If this error occurs, replication SVN repository to local disk will help. Following instruction will show how replication is done.

1. Prepare replication:

mkdir gitMigration
cd gitMigration
svnadmin create svn_repo
cd svn_repo/hooks
echo '#!/bin/sh' > pre-revprop-change
chmod 755 pre-revprop-change
cd ../..

2. init local svn
single branch repo: svnsync init file:///home/xying/gitMigration/svn_repos https://xxx.svn.net/source/PROJECT/branches/branches1 --source-username xying --source-password n123
mult branches repo: svnsync init file:///home/xying/gitMigration/svn_project https://xxx.svn.net/source/PROJECT/branches --source-username xying --source-password n123


3. Replicate: sync from remote svn repo
svnsync sync file:///home/xying001/gitMigration/svn_repos
svnsync sync file:///home/xying001/gitMigration/svn_project
Replication will take some time. Sync command can run again to get latest changes.


4. Activate local repository:
svnserve --root svn_repos --listen-port 3030 -d
svnserve --root svn_project --listen-port 3031 -d
netstat -an | grep LISTEN
ps -ef | grep svnserve

5. Create configuration for git:
./subgit-3.3.1/bin/subgit configure svn://localhost:3030 abranch.git
./subgit-3.3.1/bin/subgit configure svn://localhost:3031 project.git

6. configure author & passwd & config
(1)For mapping from SVN committers to Git committers, download authors.txt and passwd
(2)for single branch repo, follow the default config about trunk/branches
(3)for mult branches repo, do config, write down the unused branches
After that you can configure and run SubGit, only difference is that in config file you need to instruct trunk directory differently:
    trunk = trunk:refs/heads/master

a. authors.txt,passwd
[xying@xxxx gitMigration]$ cd /home/xying/gitMigration/project.git/subgit/
[xying@xxxx subgit]$ diff authors.txt ../../abranch.git/subgit/authors.txt 
10c10,22
< 
---
> xying = Ying Xin <xin.ying@email.com>
> huax = Xiao Hua <hua.xiao@email.com>
[xying@xxxx subgit]$ cp ../../abranch.git/subgit/authors.txt  authors.txt
#
subgit secret
xying xxx

b. config
[xying@xxxx gitMigration]$ vi project.git/subgit/config 
......
        trunk = trunk:refs/heads/master
        branches = branches1:refs/heads/branches1
        branches = branches2:refs/heads/branches2
        branches = branches/*:refs/heads/*
        tags = tags/*:refs/tags/*
        shelves = shelves/*:refs/shelves/*
......


7. install or import git repo, from svn to git
[xying@xxxx gitMigration]$  ./subgit-3.3.1/bin/subgit install project.git
'notes: if you modify config and cut down some branches，need parameter(rebuild), force install
[xying@xxxx gitMigration]$ ./subgit-3.3.1/bin/subgit install --rebuild project.git


8. after import success, check the branch, select which branch need to push to remote gitlab or gerrit
[xying@xxxx gitMigration]$ cd project.git/
[xying@xxxx project.git]$ git branch -a
  branches1
  branches2

[xying@xxxx project.git]$ git remote add origin https://name@gitlab.com/proj/PROJ.git
[xying@xxxx project.git]$ git remote add ger https://name@gitlab.com:443/a/PROJ
[xying@xxxx project.git]$ git remote -v
ger  https://name@gitlab.com:443/a/proj (fetch)
ger  https://name@gitlab.com:443/a/PROJ (push)
origin  https://name@gitlab.com/proj/PROJ.git (fetch)
origin  https://name@gitlab.com/proj/PROJ.git (push)

[xying@xxxx project.git]$ git push origin branches1
[xying@xxxx project.git]$ git push ger branches2

