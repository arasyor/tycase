# 6. Gitlab Installation
## Download repo and install Gitlab repo
```
curl -O https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh 
sh script.rpm.sh 

yum -y install gitlab-ce

gitlab-ctl reconfigure 
```

## Default user and password
```
Default admin account has been configured with following details:
Username: root
Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.
```

