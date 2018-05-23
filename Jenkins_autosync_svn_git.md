# Jenkins auto sync svn from svn to git
## config and create Jendkins task
## Jenkins run shell
## after basic create/configure mapping from svn to git
## details about configuration of mapping, looking for git/subgit/config
```# Subversion repository URL
   url = svn://localhost:3031
```
## create configure cli
`./subgit-3.2.7/bin/subgit configure svn://localhost:3031 cloudBranch.git`

## Jenkins running shell details
```
export JAVA_HOME=/home/xying/jdk1.8.0_171
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

cd /home/xying/gitMigration/
svnsync sync file:///home/xying/gitMigration/svn_cloudBranch
## As port 3031 maybe killed by building env manager, ignore cli failure
svnserve --root svn_cloudBranch --listen-port 3031 -d || echo "continue"
./subgit-3.2.7/bin/subgit install cloudBranch.git

# git remote : gerrit and git branch: CLOUDTRSW_BR
cd cloudBranch.git/
git push -f gerrit CLOUDTRSW_BR
```
