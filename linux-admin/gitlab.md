# GitLab
### Installation
```
// For free version
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
yum install gitlab-ce

vi /etc/gitlab/gitlab.rb
..
external_url 'http://192.168.122.247'
gitlab_rails['gitlab_email_from'] = 'git@gitlab-server-01.localdomain'

gitlab-ctl reconfigure
sudo gitlab-ctl restart
```
### Reset password
```
gitlab-rails console -e production

user = User.find_by(username: 'root')
user.password = 'newpassword'
user.password_confirmation = 'newpassword'
user.save!

exit
```
### GitLab Runner Installation
```
sudo curl -L --output /usr/local/bin/gitlab-runner "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/binaries/gitlab-runner-linux-amd64"
sudo chmod +x /usr/local/bin/gitlab-runner
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```
#### Update GitLab Runner
```
sudo gitlab-runner stop
sudo curl -L --output /usr/local/bin/gitlab-runner "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/binaries/gitlab-runner-linux-amd64"
sudo chmod +x /usr/local/bin/gitlab-runner
sudo gitlab-runner start
```
### Can't ssh to GitLab even though everything is right
The problem is when we offering the right ssh key, sshd says it's wrong
```
ssh -Tv git@192.168.122.247
```
git clone via ssh also doesnt work
```
git clone git@192.168.122.247:git_user/awesome_ci.git
```
For me, the solution was is to disable SELINUX
```
sestatus
..
SELinux status: enabled

vim /etc/selinux/config
..
SELINUX=disabled

reboot
```
The more verbose solution, without disabling SELinux maybe like this, when ssh service is being denied permission to read files:
```
sudo semanage fcontext -a -t ssh_home_t /gitlab/.ssh/
sudo semanage fcontext -a -t ssh_home_t /gitlab/.ssh/authorized_keys
sudo restorecon -F -Rv /gitlab/.ssh/
```

For someone, the solution was is to reconfigure git
```
gitlab-ctl reconfigure
gitlab-ctl restart
```
For someone on older distribution the solution was is to add git to ssh group because AllowGroups in sshd
```
AllowGroups ssh
```
For someone the solution was to delete .lock file near authorized_keys
```
cat /etc/passwd | grep git
..
git:x:997:994::/var/opt/gitlab:/bin/sh

ls -a /var/opt/gitlab/.ssh
..
authorized_keys authorized_keys.lock
```
