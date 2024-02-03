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
