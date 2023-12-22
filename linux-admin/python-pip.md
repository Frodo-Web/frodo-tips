# Python and pip tips
Uninstall all pip packages installed by current user
````
$ pip freeze --user | xargs pip uninstall -y
````
