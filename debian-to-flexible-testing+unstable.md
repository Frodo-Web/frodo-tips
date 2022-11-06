# Install Debian with flexible testing+unstable repo configuration
## 1. Download and Install current daily snapshot (testing repo, right at the bottom of the page)
````
# This page:
https://www.debian.org/devel/debian-installer/

# For example, for amd64 system, you need to follow this link:
https://cdimage.debian.org/cdimage/daily-builds/daily/arch-latest/amd64/iso-cd/

# And download this file
>> debian-testing-amd64-netinst.iso
````
## 2. Edit /etc/apt/sources.list
````
sudo vim /etc/apt/sources.list
````
Now we need to add testing and unstable sources. If we will add them by using words "testing" and "unstable" (without the exact codenames like "bookworm")
it will help us for future distro updates (no need to edit these sources on new Debian releases). Make it look like this:
````
# Testing repository - main, contrib and non-free branches
deb http://deb.debian.org/debian testing main non-free contrib
deb-src http://deb.debian.org/debian testing main non-free contrib


# Testing security updates repository
deb http://security.debian.org/debian-security testing-security main contrib non-free
deb-src http://security.debian.org/debian-security testing-security main contrib non-free


# Unstable repo main, contrib and non-free branches, no security updates here
deb http://deb.debian.org/debian unstable main non-free contrib
deb-src http://deb.debian.org/debian unstable main non-free contrib
````
## 3. Add /etc/apt/preferences.d/my_preferences
This is an important step. You can name the file what you want.
````
sudo vim /etc/apt/preferences.d/my_preferences
````
Make it look like this:
````
Package: *
Pin: release a=testing
Pin-Priority: 700

Package: *
Pin: release a=unstable
Pin-Priority: 650
````
## 4. Now install a package from unstable repo:
````
sudo apt -t unstable install firefox
````
