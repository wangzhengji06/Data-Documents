

# Environment  Setup

## VScode

- Make sure `code` command work in powershell. (Add to PATH)

## WSL2

- Powershell: `wsl.exe --install <Distro>`  (ubuntu 20.04 if possible)
- Install wsl extension on vscode
- In vscode's linux terminal: `mkdir example_folder` `cd example_folder` `code` . Then the vscode will be installed in that place.

## Git 

- Check linux command by `git --version` If it is 2.25, you will have to upgrade it. 
- `sudo add-apt-repository ppa:git-core/ppa`
- `sudo apt-get update`
- `sudo apt-get install git`

# ZSH

- In your ubuntu terminal, run `sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
- If it does not work, `sudo apt-get install zsh` Then run the above command again
- `code ~./.zhsrc`, change the theme to `ZSH_THEME = 'bira'`, save.
- (Optional) Use powerlevel10k.  Download the file from GitHub and configure. `p10k configure` To make powerlevel10k work, you will have to modify the fonts in all the terminals (Gnome, Vscode) 
- Install al in one markdown extension.
- plugins=(git web-search python pyenv virtualenv pip zsh-autosuggestions zsh-syntax-highlighting)

# Semantic  Versioning and Python Version Control

1. Major Change: something happen that is not backward compatible: 1.2.0 -> 2.0.0
2. Minor Change: A new feature has been added but is backward compatible: 1.1.5 -> 1.2.0
3. Patch: No new feature, backward compatible, change for existing function: 1.1.0 -> 1.1.1
4. 0.0.0 -> Alpha phase 
5. Deprecation Warning should be necessary. Add a print function will do.  Or `from warning import warn` 
6. Why do we need to use different versions of python: 1. Package depends on different python version 2. Deployment environment requires certain python version.

# Pyenv

1. `curl -fsSL https://pyenv.run | bash`  Act according to instruction
2. Install the build environment for pyenv. Pyenv also needs python to run, so basically it will try to run from source python. The source python needs C to build. Therefore 
3. Use `pyenv install xx` and `pyenv shell xx` to install and change the current python version.  
4. Use `pyenv global xx` to make the python version works for other terminal (There will be warning here lol)



# Vscode Setting

Short cut to memorize: 

1. `ctrl + ~` open terminal  
2. `ctrl + shift + p` show all commands
3. `ctrl + k`  `z` enter Zen mode 
4. `ctrl + d` Highlight the next same string and create multiple cursors
5.  `shift + end` highlight until the end of the line
6. `alt + arrow` move the line
7. `alt + shift + arrow` Duplicate on certain location
8. `ctrl + alt + up/down` Add multiple cursors
9. `alt + cursor` create multiple cursors 

Add following plugins: python, pylance, 



# Python Virtual Environment

Always make sure that you are installing package in a virtual environment version of your pyenv python version!

1. Create `python -m venv ./venv/` venv is a standard library in python that lies in your current `which python/lib/pythonxxx/venv.py` By executing this command, you create a virtual environment in a folder called `/venv`
2. Activate `source ./venv/bin/activate` This will make `pip install` land into `./venv/lib/site-packages/` and python programs which was softly linked to your true environment, when it evaluates the `import` statement, it will locate packages in `./venv/lib/site-packages/` 
3. Deactivate `deactivate`

Note: In active file, it will export to `PATH`, which is nothing but when the python runs, it will first check the file located in the `PATH`. Just  think of it as a bookmark to avoid writing absolute path. 



# Git

Preference:

1. Install GitLens vscode plugin
2. In vscode settings, delete the .git excludes so that you can see the git file
3. `git config --global user.name "lzabry"        ` `git config --global user.email "wangzhengji06@gmail.com"`
4. You want to avoid moving the folder otherwise there would be no previous data. 
5. You are supposed to commit frequently 
6. Gitlens -> views -> commits -> files: layout  Change to Tree type. 
7. Don't break the master branch. 

Concepts:

1. repositories: A folder that is full of file that is controlled by git. has a `.git` file. 
2. commits: A set of changes done to the repository

![img](https://img-c.udemycdn.com/redactor/raw/article_lecture/2023-03-10_09-21-31-1a8adae8355b55d97c7ef0737fbee0a2.png)



# Github and Code Review 

To collaborate with other developers.

- create the repo locally: local repository -> 'remote' repository (`git add remote <name> <URL>`)
- create the repo remotely: `git clone <URL>` 
- create a branch locally: `git checkout <branch>; git checkout -b <new branch>; git add ...; git commit -m ""` After several state and commits, `git push -u origin <new branch>` 
- create a new branch remotely: Just use Github UI `git fetch --all` `git checkout <branch>` This will allow git to create a counterpart locally.
- `Git pull` = `git fetch + git merge` 

![img](https://ericriddoch.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F02c230b9-a16f-4767-9db4-03abf1e103f5%2F4bc1ce0c-99c6-497e-be1a-8bef781657bb%2F2023-03-10_09-34-21-66b28ed47809d56d98e137941b5e90bb.png?table=block&id=6e6dbb12-d567-410d-8613-8c06abb6d5b4&spaceId=02c230b9-a16f-4767-9db4-03abf1e103f5&width=1420&userId=&cache=v2)

# Continuous Integration

## Clean Code

- Pure Function: There is no global variable in your function
- Self Explanatory: Intuitively understandable
- Variable name: Cap lock the global variable etc. 
- Type: Add input, output type for the function
- Document: Add some comment to explain if necessary

### Resources / Links

- [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)
- [Python PEP 8](https://peps.python.org/pep-0008/)

### Quick Reference

- [Clean Code](https://testdriven.io/blog/clean-code-python/) Book mark this and repeatedly reading it. 



Does it work? Is it secure? Is it clean? It is very important to make small pull request. By making large pull request, you are interfering with other contributor. 

### Black

