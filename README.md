## Are you curious about how such big companies, manage their code with so many developers working on a single project.

Here is a quick peek into how the development process is handled by these companies

# Let's Start !

I am assuming you have jenkins and docker already setup and know the basic commands of docker and workflow in jenkins, if not you can still follow along. I will explain each and every command clearly

You have git installed and configured in your system and know the basics of it

*(I will be using RHEL 8 in the process)*

## Setup :

Go to your github account and make a new repository named JenkinsTest, initialize it with a README.md and clone it some where in your RHEL operating system

```
cd /project
git clone https://www.github.com<your user name>/JenkinsTest.git
```

Suppose we are working on a web developmen project so lets make files in the project

```
touch index.html
cat >> index.html
Hello, I am the content in master branch index.html'
(Press Enter and then press Ctrl + D to enter a endline character, which tells cat that you have finished editing)
```

Now we have to add the file and commit the changes

```
git commit -m 'Added index.html'
```

Now this commit sits on our local repo, and no changes will be shown in remote repository To send this commit on our remote repository, we have to push it in the github repository

```
git push origin master
```
(**origin** : alias for the URL of your git repository) (**master** : the branch where you want to push the changes )

If you have not configured git for SSH keys or credential helper you will get a prompt to enter username and password, this starts irritating soon, so for this project we will be using credential helper to store our password and user name, so we don't have to enter it again and again

***NOTE : Credential helper stores password and user name in plane text file, so SSH is the more secure way but for simplicity we will be using credential helper***

## Setup credential helper

```
git push origin master
Username : <type your username>
Password : <type your password>
```

So, far we are working on master branch, the code on this branch is the production code, that is this code is the one we will use to run our website

Say a developer wants to add some feature, but he is a fresher so you don't want him to add some bug while working on the new feature, to tackle this you assign him a new development branch for him to work which will be merged to the master branch, if it passes all tests and QAT ( quality assurance team) approves the merge

Lets make a new branch and then switch to that branch

```
git branch dev
git checkout dev
```

Now you are one dev branch, lets introduce some changes in the index.html file as follows

```
cat >> index.html
Hey, Now I am changing the code from dev branch 
(press enter and ctrl + D)
```

After changing we have to add and then commit the changes

```
git add index.html
git commit -m 'Made some changes from dev branch in index.html'
```

To see all the commits and process you have done, lets check the log

```
git log --graph --oneline
```

It will show you that HEAD is pointing to dev branch and is one commit ahead of master branch

**let's push these changes in the dev branch to the remote repository**

```
git push -u origin dev
```

***Remember origin holds our remote repo URL , so we are pushing the dev branch in remote URL origin***

### Pheww !

We have our setup for the task, now lets move to automation process

# AIM :

Let's list down all the tasks we have to do

- Setup a httpd container holding the production website
  - Make a persistent storage that will hold the production code for the container
 
- Setup a httpd container holding the development website for testing
  - Make a persistent storage that will hold the production code for the container
 
- ***Job 1*** : Make a jenkins job which pulls the code from master branch and deploy it in production httpd container

- ***Job 2*** : Jenkins job which will pull from dev branch and deploy the code in development testing httpd container, an email will be sent from here to QAT for further testing

- ***Job 3*** : This job will be triggered by QAT ( quality assurance team ) by a remote URL that you will provide to them , this job will merge dev branch into master branch and call Job 1 again

- **Integrate all the things together and run it**

# Let's Start !

- Setup persistent storage for container holding the production website

First, we have to pull the httpd docker image from docker hub

```
docker pull httpd
```

*Making a persisten storage for development docker container If you dont know about it, think of it as an external pendrive having our production code which we will attach in docker to deploy the server*

```
mkdir /vol/productionStorage
``` 
-[x] **Setup persistent storage for container holding the production website**

## Now lets move to the next task.
- [ ] Setup persistent storage for container holding the development website

Make a persistent storage for httpd container

```
mkdir \vol\devContainer
```

- [x] **Setup persistent storage for container holding the development website**

## Now we head over to jenkins, open jenkins in your browser and login
- [ ] **Job 1 : Make a jenkins job which pulls the code from master branch and deploy it in production httpd container**

- Select **New Item** in Jenkins dashboard

- Name it **GetMasterBranch**

- Select defaults and then OK

- Open the **GetMasterBranch** project

- Go to **Configure**

- In **General Tab** select **GitHub Project** and add your project URL

- Head to **Source Code Management**

  - Select **Git** radio button and Enter the **Repository URL**
  - If there is no **Git radio button**, then install the *Git plugin for jenkins by going to Dashboard and Manage Jenkins then Manage Plugins*
  - Check the **Branches to Build** is set to ***/master***
  - Head to **Build Triggers**

   - Select **GitHub hook trigger for GITScm polling** ( we will get into this later, this allows github to trigger this job )

- Head to Build

  - Check **GIT SCM for polling**
  - Select **Add build** step drop down
  - Select **Execute shell** and then type in this command
```
if sudo docker ps |  grep masterContainer
then
echo "already running masterContainer"
else
sudo docker run -dit -p 8082:80 -v /vol/masterBranch:/usr/local/apache2/htdocs/ --name               masterContainer httpd
fi
sudo cp -f * /vol/masterBranch/
```
- **Save**

### Lets talk about the command we wrote in the Execute shell

*First it checks whether a container named **masterContainer** is running, if not then run the container using :

```
sudo docker  run -dit -p 8082:80 -v /vol/masterBranch:/usr/local/apache2/htdocs/ --name masterContainer httpd
```

- **-dit** : runs the container having an ***i***nteractive ***t***erminal which is ***d***etached, i.e. we don't get a bash shell after the conatiner is launched

- **-p 8082:80** : we expose the docker, so that other system can connect to it. If u user from base system goes to **(base system IP):8082** then he will be PATed to port 80 in docker, where our web server httpd is running

- **-v /vol/masterBranch:usr/local/apache2/htdocs/** : attaching our persistent storage to docker

- **--name masterContainer httpd** : run a httpd docker image with the name *'master Container'*

***- [x] Job 1 : Make a jenkins job which pulls the code from master branch and deploy it in production httpd container***


## Lets head to JOB 2:
- [ ] **Job 2** : Jenkins job which will pull from dev branch and deploy the code in development testing httpd container, an email will be sent from here to QAT for further testing
This will be configured in the same way as job1 only few changes are to be made

- In **Source Code Management** change the **branches to build** to ***/dev**

- In **Execute Shell** add the following command

```
`
if sudo docker ps |  grep devContainer
then
echo "already running devContainer"
else
sudo docker run -dit -p 8083:80 -v /vol/devBranch:/usr/local/apache2/htdocs/ --name devContainer httpd
fi
sudo cp -f * /vol/devBranch/
```

*The command is same as for the job 1 but we have change the name and persistent storage location

- Head to Post-Build Actions
  - Select Add post-build action dropdown and select E-mail notification
  - Enter your QTA team tester's ID in the Recipients
*
This will mail the tester that the developer has pushed to **dev branch** and now its the testing engineer duty to approve the merge and check the site in **development docker container is working fine**

*( we are assuming that the whole company works on a single network )

So, the devContainer can be accessed at **(base system IP):8083**

If everything is working fine in the development container, then the QTA guy will trigger the Job 3 using remote URL we will provide to him ***(the remote URL will be provided later in the job 3)***

- [x] **Job 2 : Jenkins job which will pull from dev branch and deploy the code in development testing httpd container, an email will be sent from here to QAT notifying a push is made, for testing**

## Finally Job 3
- [ ] Job 3 : This job will be triggered by QAT ( quality assurance team ) by a remote URL that you will provide to them , this job will merge dev branch into master branch and call **Job 1** again

- Start by making a new job named **MergeDevInMaster**, add the basic configuration

- Then in **Source Code Management** select **None**

- Head to **Build Triggers**

  - Select **Triggers builds remotely**
  - You will be prompted with **authentication token**, authentication Token is used so that only people having this token can trigger the job
  - Set it to **build**
  
- A URL will be shown there as follows : ***JENKINS_URL/job/MergeDevInMaster/build?token=TOKEN_NAME***

- This is the URL that the QAT will already have, here

  - Jenkins URl = URL of jenkins, eg : **(Base system IP):8080**
  - **token = build**
Final URL : **(base system):8080/job/MergeDevInMaster/build?token=TOKEN_NAME or /buildWithParameters?token=build**

This URL should be used as follows by the testing team

```
curl --user "(jenkins_user_name:Jenkins_password)" (base system IP):8080/job/MergeDevInMaster/build?token=build
```

***Now we have to make this job, merge the two branches ONE way would be to use some additional plugin or a bash script placed in the project folder (folder where index.html is placed )***

- Bash Script has following commands :

```
git merge dev
git push
```

- Head over to **Build** now :

- Select **Execute Shell**

- Type the following command
```
cd /..
cd /project/jenkinsTest
sudo bash merge   
```

- **Save**

 - [x] ***Job 3 : This job will be triggered by QAT ( quality assurance team ) by a remote URL that you will provide to them , this job will merge dev branch into master branch and call Job 1 again***

# Final Steps
- [ ] Integrate all the things together

Remeber, we selected **GitHub hook trigger for GITScm polling**, this trigger is initiated by Github by using **Github webhooks**

Before we configure our webhooks, our current setup is placed in a private network, so a job in jenkins which is placed in a **private network** can't be accessed by github, to connect our system to Internet, we will be using *tunneling with the help of ngrok software*

Download ngrok from internet, Extract it and then run following command in RHEL terminal
```
./ngrok http 8080
```

You will get this output in terminal with a weird looking URL ending with io, we will use that IP to connect our jenkins to github

This will connect our jenkins on ***private network to Internet***

Now,

- Head over to your project on **GitHub**
- Go to **Settings**
- From the menu in the left select **Webhooks**
- Click on add **webhook**
- In the **Payload URL** add the URL ngrok provided to you with following extra edit:
```
https://78871dad.ngrok.io/github-webhook/
(Replace the URL with the one ngrok gave you)
```
- Select **Content type** to **application/json**
- Click **Update Webhook**

***NOTE : ngrok should be running for always for using GIT triggers***

*If you dont want to use GitHub hooks, then in jenkins, under Triggers section select Poll SCM and specify the time after which you will check the github repo for changes*

The method using poll SCM increasese overhead cost because it keeps checking repo after fixed interval of time, whereas GitHub WebHooks trigger jenkins when some event like push happens

## Adding merge bash script in project folder

We are using a bash script to merge two branches

- Navigate to your project folder in RHEL

```
cd /project/jenkinsTest
```
- Make a **bash script** and add the following commands

```
touch merge

cat >> merge
git merge dev
git push
(press Ctrl + D after Enter)

```

**One last thing, if you observed closely we are using sudo before some commands, so what is that ?**

*Jenkins exist as a separate user in our RHEL system, using sudo before commands gives jenkins user the power of **root account***

For this we have to change the **sudoers file**

- Open a terminal and run the following commands

```
gedit /etc/sudoers
```
- This will open **sudoers file** in **gedit editor**

- Navigate to the bottom of the file and find the line
root ALL=(ALL) ALL

- Add the following after the above line

```
jenkins ALL=(ALL) NOPASSWD: ALL
```

(For simplicity we are giving all root account power to jenkins user too or jenkins can behave as pseudo root )

- Save the file and exit

- [x] **Integrate all the things together**


# Finally , we have completed all the steps now lets run our whole setup

Follow the following steps

- Go to your project in RHEL 8 and switch to master branch
- Make some changes to index file and push it to github

```
git checkout master
cat > index.html
Some changes from master branch
(Ctrl + D)

git add index.html
git commit -m 'Made some changes from master'

git push
```

- As soon as you push this commit, github will trigger the **webhook** and **GetMasterBranch** Job will run in Jenkins

  - You can check if github was successful in running the job, by going to settings of your repo in github
  - Go to webhooks
  - Select your **webhooks** and see the **recent delieveries** section
  - There should be a **green tick sign** there
  - If not, click on delievery and troubleshoot it **( remember ngrok should be running)**
  - Now do a push from dev branch in the project folder and **GetDevBranch** runs

```
cd /project/jenkinsTest
git checkout dev
cat >> index.html
Changes from dev branch
(Ctrl + D)

git add index.html
git commit -m 'changes from dev branch'
git push
```

- The **GetDevJenkinJob** send the email to QAT notifying a new push event
- The QAT can test the new push deployed in **docker container at (base system IP):8082**
- If everything works fine then, QAT will run the following command in terminal

```
curl --user "(jenkins_user_name:Jenkins_password)" (base system IP):8080/job/MergeDevInMaster/build?token=build
```

This will trigger the last Job that is to **merge the dev and master branch together** and then push the code back in the repo

This push event will again trigger **GetMasterBranch** and deploy the container or change the persistent storage with the new code
