# Python and pip tips
Uninstall all pip packages installed by current user
````
$ pip freeze --user | xargs pip uninstall -y
````
## Create and Use Virtual Environments
````
pwd
..
/home/frodo/Testing/rabbitmq

python3 -m venv .venv
source .venv/bin/activate
which python && which pip
..
/home/frodo/Testing/rabbitmq/.venv/bin/python
/home/frodo/Testing/rabbitmq/.venv/bin/pip

ls -alh .venv/
..
drwxr-xr-x 2 frodo frodo 4.0K Jan 21 09:27 bin
drwxr-xr-x 3 frodo frodo 4.0K Jan 21 09:26 include
drwxr-xr-x 3 frodo frodo 4.0K Jan 21 09:26 lib
lrwxrwxrwx 1 frodo frodo    3 Jan 21 09:26 lib64 -> lib
-rw-r--r-- 1 frodo frodo  173 Jan 21 09:26 pyvenv.cfg

python3 -m pip --version
python3 -m pip install --upgrade pip
python3 -m pip --version
..
pip 23.3.2 from /home/frodo/Testing/rabbitmq/.venv/lib/python3.11/site-packages/pip (python 3.11)

python3 -m pip install ...

// Install from source
cd google-auth
python3 -m pip install .

// Install from local archives
python3 -m pip install requests-2.18.4.tar.gz
````
#### Freezing dependencies
Pip can export a list of all installed packages and their versions using the freeze command:
````
python3 -m pip freeze
..
pika==1.3.2
````
The pip freeze command is useful for creating Requirements Files that can re-create the exact versions of all packages installed in an environment.
#### Deactivate a virtual environment
````
deactivate
````
