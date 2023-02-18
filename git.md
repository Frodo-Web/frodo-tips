# Git tips by Frodo<br>
## Setup Git
List the settings
````
git config --list  // Showing system, global, and (if inside a repository) local configs
git config --list --system
git config --list --global
git config --list --local  // Specific to that single repository
````
Direct way
````
git config user.email
git config --global user.email
git config --local user.email
````
Configure Git
````
git config --global user.name "Your Name"
git config --global user.email "123456789+example@users.noreply.github.com"  # Remember to use your own private GitHub email here
git config --global init.defaultBranch main  // Change the default branch to main
git config --global color.ui auto  // Enable colorful output
git config --global pull.rebase false  // pull --rebase:  Update your local working branch with commits from the remote, but rewrite history so any local commits occur after all new commits coming from the remote, avoiding a merge commit.
ssh-keygen -t ed25519 -C <youremail>
***Copy your public SSH key to Github***
ssh -T git@github.com  // Testing your SSH connection
````
## Fix remote URLs from HTTPS to SSH for Project
````
git remote set-url origin git@github.com:username/repo.git

// Then check remote URLs again:
git remote -v
````
## Remove directory from Git but NOT local
````
git rm -r --cached myFolder
git commit . -m "Remove duplicated directory"
git push origin <your-git-branch>
````
## Changing git commit message after push (The --force-with-lease option is the safest, because it will abort if there are any upstream changes)
````
git commit --amend -m "New commit message"
git push --force-with-lease <repository> <branch> 
````
## Create a new repository on the command line
````
echo "# Labratory-1" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:Frodo-Web/Labratory-1.git
git push -u origin main
````
## Push an existing repository from the command line
````
git remote add origin git@github.com:Frodo-Web/Labratory-1.git
git branch -M main
git push -u origin main
````
