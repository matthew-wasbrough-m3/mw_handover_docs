# Dev Machine Setup November 2018 <!-- omit in toc --> 

Everyone has different tastes on how they should get their machine set up. This is a nuts and bolts description of how I would set up a machine which aims to act as a starting point for anyone setting up a machine for the first time who might be less familiar.

## Contents <!-- omit in toc -->
- [Starting point notes](#starting-point-notes)
- [Homebrew](#homebrew)
- [Development Environment](#development-environment)
- [Github](#github)
- [Git](#git)
    - [Common Commands](#common-commands)
        - [Clone](#clone)
        - [remote](#remote)
        - [checkout](#checkout)
        - [pull](#pull)
        - [status](#status)
        - [commit](#commit)
        - [push](#push)
- [Node](#node)
- [nvm](#nvm)
- [npm](#npm)
- [docker](#docker)
- [Other notes](#other-notes)
    - [Programs I use](#programs-i-use)
    - [An old list of some of the M3Medical apps and their names:](#an-old-list-of-some-of-the-m3medical-apps-and-their-names)
    - [Clinaccess Communities and their numbers](#clinaccess-communities-and-their-numbers)

## Starting point notes
This document is written mostly OS agnostic but obviously there are differences between macos and linux so there are some differences. As I didn't have a spare mac when writing this I have written it with the help of an Ubuntu laptop.

## [Homebrew](https://brew.sh/)
This is specific to macos. With a linux system you generally have a package manager to help with installing dependencies. Homebrew acts as a package manager for macos and I recommend using it to install some of the following packages. To install it visit https://brew.sh/ and read the installation notes.

## Development Environment
This is the most personal part of this document. I have used a number of different editors and development environments. At the time of writing the most common in the M3 team is to use Visual Studio Code. To read up on it and download it then visit https://code.visualstudio.com/. If you are on a mac and have homebrew installed you can run `brew cask install visual-studio-code`.

Alternatives to Visual Studio Code that have been recommended are Atom and Sublime Text. Sublime Text is probably the better choice but it is a paid software. Atom, like Visual Studio Code, is a free download.

## Github
Assuming the main focus of development is JavaScript then github is going to be the main source of code. Once you have a github account (if you haven't created one yet then do so with the following username: firstname-surname-m3) go to account settings. Under `SSH and GPG keys` you will need to create an ssh key to allow the command line to authenticate against github. To do this click the `New SSH key` button, you can give the key any title you want, the key you will need to paste from the output of the following:

- In terminal change directory to the .ssh directory: `cd ~/.ssh`
- Create an ssh key by running `ssh-keygen`
- For the most part follow the instructions, however I recommend changing the file name of the key by typing in a new name as the answer to the first question.
- Running `ls` should then show the two keys created using the name you created in the step above. One file will have the extension .pub this is the public key. The other is the private key.
- Look at the public key by using `cat filename.pub` where filename is the name you chose above.
- Copy the resultant text into the `Key` text box in github.

Now you can authenticate against Github you need the command line tools to get the code and this command line tool is called `Git`.

## Git
The main version control system used by M3 is git. This is used in Github, Gitlab and Gogs. To read up on the documentation for Git then visit https://git-scm.com/doc. Git is a highly complex command line tool that you probably won't use half of but it is worth getting to know the bits you use regularly well because you will use them all the time.

To install Git on linux `sudo apt install git` or `brew install git`. If you are on a mac then I would recommend `brew install hub` as well. Read the details at https://hub.github.com/.

### Common Commands
The common commands for development are as follows:

#### Clone
`git clone url`: this will clone the repository from Github. Because you are authenticating with ssh you need the ssh url in place or `url` in the command. For instance if you go to a github repository (such as https://github.com/m3europe/m3.intl.offduty-controller) there is a green button reading `Clone or download`. Clicking this button will give you a text box with a url to clone the repository. If this url starts `https` then this is the https link and you need teh ssh link, to get this link then in the top right of the pop up box there is a link saying `Use SSH`, clicking on this will change the url start with `git@` which indicates you are using the ssh url.

#### remote
To view the urls of the remote repository you are working with you can run `git remote -v`. If this is your first time looking at the remote urls you will see origin as the name against the repository url. To help me stop accidentally pushing to the m3europe repository I rename this link from origin to m3europe. To go this you run `git remote rename origin m3europe`.

I also work on my own `fork` so I need to add the url of the fork to allow me to push my changes to there. To do this I find the ssh url of the fork in the same was as finding url as discussed above (for me this would be on the page https://github.com/matthew-wasbrough-m3/m3.intl.offduty-controller). Once I have the url I add it as a remote by running the command `git remote add mjw url`. In this example mjw is just a name for the remote location, I use my initials.

Once you have added your remote you should see something like the following:
```
m3europe    git@github.com:m3europe/m3.intl.offduty-controller.git (fetch)
m3europe    git@github.com:m3europe/m3.intl.offduty-controller.git (push)
mjw    git@github.com:matthew-wasbrough-m3/m3.intl.offduty-controller.git (fetch)
mjw    git@github.com:matthew-wasbrough-m3/m3.intl.offduty-controller.git (push)
```
#### checkout
If you are working on lots of different bugs in a single project it is likely that you will fix one and pass it to QA and start working on the second. To keep the code clear between two sets of work it is common to separate each piece of work as a branch and only merge them together when the work is complete. To do this you can switch between branches using the `git checkout branch-name` command. To create a branch that doesn't already exist you just add a `-b` flag so the command becomes `git checkout -b branch-name`. If you are working on many branches then it is reommended that you use the jira ticket to name your branch so it is easy to remember what work was being done.

Even if you are not working on multiple branches when you first clone a porject directory you will get the default branch of code. For most repositories at M3 this branch will be the master branch which may not always be as up to date as integration. In this case the first thing you should do is run `git checkout integration` which will make the integration branch the working branch on your system.

#### pull
If you haven't worked on the codebase for a while then it may be that there is other peoples work that you need to build on. To get the latest changes from a branch the command is `git pull`. This command should be used with slight caution as it will only pull the latest code from the currently matching branch on the upstream fork, basically what that means is that depending on where you cloned your repository from and the currently open branch you might get different requested code. To be more explicit you can name the remote fork location and the branch by adding them as arguments to the command. assuming I wanted to get the latest code from the integration branch of the m3europe fork then using the remote name as described above the command would be `git pull m3europe integration`.

Sometimes you forget to `pull` the latest code before you start work or someone else has been working on the same repository and has commited their code before you. To get any missing code on to your branch you can use the `rebase` command. The common example is that if the integration branch started with commits `A-B` then you cloned it and started work but someone else put a commit onto the branch while you were working the commits would look like `A-B-C`. If you subsequently finished your work and commited it then your branch would look like `A-B-D`. By running `git pull --rebase m3europe integration` your local branch would be reordered to `A-B-C-D` and then ready to be merged onto the integration branch.

#### status
To check what files have changed since the last commit you can run `git status`. This is a useful command both when you are trying different things and when you come back to a project after a while of not working on it.

#### commit
When you have finished developement work, or just got it in a place you want to save it back to the repository, you do this by commiting your changes. If you first run `git status` you can see what has changed then you can choose what you want to commit. A commit is simply a grouping of changes together with a label so you know why they were grouped.

If you want to commit all of your changed files together you can run `git add .` (there is a short hand which will be discussed later). If you want to add individual files you have to refer to them by path for instance a file called `index.js` in the `src` directory would be added with `git add src/index.js`. It is possible to add a directory and get all changed files within that directory, using the same example above you would run `git add src/`.

When you have chosen all the file you wish to commit then you use the `commit` command to group them together. When commiting the files you should always add a message to describe what the changes are, this is done by using the `-m` flag making the command `git commit -m "some code for commiting"`. If you are adding all changed files to the commit you can skip the add step above and add the `-a` flag to the commit command like `git commit -am "some code for commiting"` and this will add all the changed files and commit them together with the given message in one step.

#### push
When you have completed all the work and commited the changes you need to save these to the remote repository. This is essentially the opposite of the `pull` command. As with the `pull` command it is best to be explicit with where you are pushing your code to. If you are working on an integration branch and have pulled from the m3europe fork then you will want to push to your own fork using `git push mjw integration`. When you have pushed the work to the remote repository (i.e. github) you need to go to the repository and setup a pull request to ask permission to merge your code into the m3europe fork integration branch. 

## Node
When I first started (and didn't know about homebrew) I installed nodejs from the website at https://nodejs.org/. On mac I would recommend using homebrew so the installation would be `brew install node` or on linux `sudo apt install nodejs`. 

## nvm
While installing node is useful for actually getting node running on the machine sometimes it is useful to be able to change what version of node you are running. For this purpose you should use nvm (node version manager). For more information about what nvm is and how to install it then go to https://github.com/creationix/nvm. (On linux I found the curl command was not included by default in ubuntu so I had to run `sudo apt install curl` as well)

With nvm you can install different versions of node to run different apps under. The use of this is that some older applications run under older versions of node and might not work if you develop using features from a newer version of node. To see what you can do with nvm run `nvm --help`. In some of our repositories there is a `.nvmrc` file which will tell your machine which version of node it should be using when you navigate to the project on the command line.

## npm
If you don't know about it then you can find more information here https://www.npmjs.com/. It can be installed using `brew install npm` or `sudo apt install npm`. An alternative to npm is yarn. This can be installed using `brew install yarn` on mac. On linux it is slightly more complicated but instructions are at https://yarnpkg.com/en/docs/install#debian-stable.

## docker
Docker is a container service that we use to deploy our apps into the aws servers. The installation is a manual process for both linux and mac. On linux the best I have found is https://docs.docker.com/install/linux/docker-ce/ubuntu/#uninstall-old-versions. On mac there is an install guide on their website at https://www.docker.com/get-started.

There are two important documents in almost every repository in github that relate to docker. The first is the `Dockerfile` and the second is the `docker-compose.yml`. The `Dockerfile` is essentially a list of instructions for the continuous integration system to use to create a docker image of the application so that the image can be deployed on a server. Details about the structure of the `Dockerfile` can be found at https://docs.docker.com/develop/develop-images/dockerfile_best-practices/.

The `docker-compose.yml` is likely to be more used by you the developer, for more detail about all the commands see https://docs.docker.com/compose/. Usually the docker-compose file is used to start up databases to allow applications to run without touching databases on integration or production. In theory most of these file have been written in such a way that they should be run simply using `docker-compose up`.


## Other notes

### Programs I use
These are very personal notes about little things that might help with development tasks. You might find better versions but these should be a reasonable starting point.
- [dbeaver](https://dbeaver.io/) - a database client for relational databases
- [Robo-3T](https://robomongo.org/) - a database client for mongodb
- [iTerm2](https://www.iterm2.com/) - a replacement for the built in terminal which can be extended
- [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) - better than bash
- [IntelliJ IDEA](https://www.jetbrains.com/idea/) - an ide for Java development
- [Postman](https://www.getpostman.com/) - api testing client

### An old list of some of the M3Medical apps and their names:
- emily = wordpress = Blogs, News
- lime survey = market research (postgres)
- piwik = analytics
- sendy = bulk mail
- gruntbuggly = clinaccess oauth bridge
- ad-service = finds contextual ads
- audit-service = CRUD for timestamped records
- chronos = bistro chron
- colin = marketing elements
- deep-thought (postgres) = targeting
- frankie = maps jurisdiction speciality models to deep thought
- file-service = S3 backed file storage
- galaxia = profile - CRUD
- galaxia-controller = UI
- galaxia-provider = OAuth
- halfrunt = bridge to dnuk content
- humma = livery
- jeltz = user prefs
- mailcatcher = catches mail for diags
- marvin = OAuth provider and client for cross-community login (talks to Frankie)
- max-controller = UI for messages
- nerpress = articles
- pizpot = product API enable product team to drive priority for site areas/functionality
- poll-service = JS pug in + API for user polls (multi choice Q’s)
- zaphod = forum

### Clinaccess Communities and their numbers
    1: dnuk
    2: mdlinx
    3: merqurio
    4: m3medical
    5: medcenter
    6: essanum de
    7: m3j
    (8: DOUG-37)
    9: Mediquality
    10: esanum fr
    (11: maas-integration)
    12: egora