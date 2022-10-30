# Git tips by Frodo<br>
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

