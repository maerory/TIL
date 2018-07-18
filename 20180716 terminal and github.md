## Today I Learned 20180716

### Linux Terminal Commands

#### 1. Commands

- echo : print
- man : manual for function! (`man echo` prints the manual for echo function)
- `clear` /`ctrl + l` - clears the window
- `less` : shows you the text file
- `alias` : adds a shortcut to a command
- `pwd` : print working directory
- `ls -a` : shows all files including hidden files, also `la`
- `ls -l` : prints the specific info about file, also `ll`
- `touch` : creates new file!
- `cat` : shows the files contents
- `curl` (an terminal library) - allows for connection to a url
  - `-I` Option shows info about the website
- `which` : checks whether an application is installed and returns the location if it is
- `wc` : word count of a file. 
- `head` and `tail` : print the few lines in the top or bottom of a file
  - with `-f` option, you can add lines below
- `ping` : send ping to a website
- `ps` : process status - current running processes
  - `ps aux` to show detailed processes
- `top` : shows the CPU status
- `find` : search for a certain file (ex `find / -name '*.rb` find all file with ruby extension from the root directory) 
- `cp` : copy and paste a file from one place to another (EX. `cp ../website/README.md <FILE NAME TO BE>` copy the readme file to my location) 
  - there is also `mv` which is cut and paste

#### 2. Hot Keys 

- When reading document in terminal, U and D for uppage or downpage
- `ctrl+a` : to the front
- `ctrl+e` : to the end
- `ctrl+u` : get read of the line
- `ctrl+w` : get read of the word
- `alt + click` : use the mouse to move the cursor
- `>` - "redirect" creates a text file with all the text generated before the redirect, 
  - you have to use `>>` to append to an already existing file
- `/` : when viewing document with less, use backslah with a word to for that word, `N` or `SHIFT+N` to move through the findings
- `|` : pipe, use the return of the former function to the latter function
- `~` : home directory
- `;` : semicolon executes all the function in order
- `&&` : executes all the function iff the formers are completed

### GITTY HUBBY!

> Git and Github is different!

1. Configure your git

   ```a
   # Global SETUP!
   git config --global user.name "<NAME>"
   git config --global user.email "<EMAIL>"
   ```

2. git help (도움말 창)

   - `git help | less` : easier to read

3. Start git with `git init` or remove with `rm -rf .git`

4. `git add <FILE NAME>` to register to the current version of repo 

   - `git rm --cached <FILE NAME>` : deleted a commited cache if the file is deleted on origin

5. `git commit -m "<your message>"`- creates the version of our repo

6. `git status` - checks the file that can be committed

7. `git log` - checks the commited files which can be pushed

8. `git remote add origin <SOURCE>` : add a remote repo to origin (`git remote` to see all remote)

9. `git push` : to upload commit to remote repo

10. `git branch <BRANCH NAME>` : to create a new branch (`git branch` to see all branches)

    - `git branch -d <BRANCH NAME>` : to delete a branch (`-D` to delete while in commit, delete no matter what)
    - `git checkout <BRANCH NAME or COMMIT ID>` : to move to a branch (`git checkout -b <BRANCH>` : to create branch and move)
    - `git merge <BRANCH>` (from master checkout) : to merge the branch to master
    - `git checkout -f` : to force to the last commit
    - `git diff master` : to check the difference between the master branch and previous commit