# 28/9/20

Met with rudolph. discussed project and ourselves. He showed us his work pc with 1TB RAM
The project is not to develop a neural network as much but port the application to the MOC, power9 processor and to make the application point and click so that doctors can use it.


# 29/9/20

Met with the team over Discord. Discussed a possible timeline for our project.

Step 1: Take the python inference script out of the terminal and into a GUI so that a human who doesn't know how to use the command line can use it in a "point and click" way. Aim to have this working by the November prototyping window.

Step 2: Port the application so that it works on the MOC. I'm not entirely sure what this means so there will need to be individual research conducted here. I think we need to make it work on Docker so we can put the Docker environment on the MOC

Step 3: Get it working on power9, which means converting it from CISC x86_64 to RISC, whilst still on the MOC. Yeah, IDK either.

Kind of daunting. I guess start on the research and then go from there.

Research topics to go over: 
MOC
Docker
Power9
Making an application for a python script


# 1/10/20

Also need to implement an algorithm that presents the most likely cases at the top of a list

First IDR today..

# 24/2/21

This is my documentation of how to create a new cookiecutter plugin for use with ChRIS, using the information provided here:

https://github.com/FNNDSC/cookiecutter-chrisapp

The machine I am using is Ubuntu 20.10

Start by creating and going into a directory for your plugin

`mkdir sevStack`

sevStack being the stack of ranked severity reports produced by pl-covidnet.

`cookiecutter https://github.com/FNNDSC/cookiecutter-chrisapp.git`

This will give a few options to fill out, and you can fill them out as you see fit. Just make sure the 

`app_repo_name`

option is all lowercase, because when the app goes to Dockerhub, Dockerhub will only accept containers that are lowercase.
When asked

`Select app_type`

I selected

`1 - ds`

because that will create a branch of a tree of nodes.

When asked 

`Select platforms`

`1 - linux/amd64`

`2 - linux/amd64, linux/ppc64le`

`3 - linux/amd64, linux/ppc64le, linux/arm64`

I selected option 3 because we will be porting the app to the MOC which has powerPC architecture, so this will build this for us for free, and #3 has support for Docker if it ever goes to Android.
That should be it! You now have a new plugin.

`ls`

Will show

`pl-sevstack`

Move into the new directory the cookiecutter made for you.

`cd sevstack`

`ls -la`

Will show what cookiecutter populated the directory with, with the name you chose for the app as a new directory within there. This contains

`sevstack.py`

Which is where the code for the plugin lives.

`nano sevstack.py`

To see what is in there. You'll see it includes an example of a bare bones execution.

From the README, there is a command to build the Docker container

`docker build -t local/pl-sevstack .`

If you haven't added your user to the Docker group, you will have to use 

`sudo docker build -t local/pl-sevstack .`

But we can easily add our user to the docker group

`sudo groupadd docker`

`sudo usermod -aG docker $USER`

Restart your machine and then test that you have been added to the docker group and can run without sudo.

`docker run hello-world`

Now build the docker container with the plugin. (Make sure to change back to the directory you were working in)

`docker build -t local/pl-sevstack .`

Now we have a built container, we can go to the README again and find the dummy command to execute. We'll need to create an in and out directory so that the app can actually do something, as well as touch up the dummy command so it works properly.

`mkdir in out`

You'll need to use some sanity checking, like making sure input and output arguments are appropriate, as well as directory paths and whatever you've named your plugin.

`docker run --rm -u $(id -u)                      \ `

`-v $(pwd)/in:/incoming -v $(pwd)/out/:outgoing   \ `

`local/pl-sevstack sevstack                       \ `

`/in /out                                           `

Edit your README under the "Run" heading and save it there so you always have it, then you can copy/paste it in your command line
Since we haven't done anything with 

`sevstack.py`,

Running the container will just print its name and version number to the terminal. We can go ahead and edit it to print

`Hello World` 

by editing this file, so go to your favourite text editor and do just that. Add it under 

`def run(self, options)`

Rebuild using 

`docker build -t local/pl-sevstack .`

And run again using the command you edited in the README and somewhere around the print out of the apps name, you will see your "hello world" message!

Now, building a Docker container is time consuming so an easy way to run the .py file is to run it in a virtual environment.
Create it and activate a virtual environment with

`python3 -m venv ss-env && source ss-env/bin/activate`

Change into the directory with the setup.py file

`cd pl-sevstack`

And to install to your python virtual environment

`pip install -e .`

`sevstack --man`

Shows it has been installed to the environment.

To build the plugin with a debugger, add to the top of sevstack.py under 

`from chrisapp.base import ChrisApp`

`import pudb`

Once you have done that, try to re-install and run

`pip install .`

`sevstack --man`

If you haven't added pudb to requirements.txt, or installed pudb in your virtual environment:

`pip install pudb`

And in requirements.txt, add

`pudb~=2021.1`

So that when you rebuild the container, it will include pudb.

`docker build -t local/pl-sevstack .`

Map your source code live into the container with a new command. I put it in the README with the other command we edited so I can get at it easily. For me it worked best if I copied it out of 

`nano README.rst`

But YMMV

`docker run --rm -u $(id -u)                                                                                            \ `

`-v $(pwd)/sevstack:/usr/local/lib/python3.9/site-packages/sevstack:ro -v $(pwd)/in:/incoming -v $(pwd)/out/:outgoing   \ `

`local/pl-sevstack sevstack                                                                                             \ `

`/in /out                                                                                                                 `      

This command means that you can just run this without rebuilding the container everytime you change the source code :)
Remember pudb? Here's how to run it. Add 

`pudb.set_trace()`

In your source code, where you want a break to happen. Then run the container with the `-ti` flag for interactive mode.

`docker run --rm -u $(id -u) -ti                                                                                        \ `

`-v $(pwd)/sevstack:/usr/local/lib/python3.9/site-packages/sevstack:ro -v $(pwd)/in:/incoming -v $(pwd)/out/:outgoing   \ `

`local/pl-sevstack sevstack                                                                                             \ `

`/in /out                                                                                                                 `

# 25/2/21

Run pudb without error messages by removing the 

`-u $(id -u)`

From the first line of this latest command.

### Automatic builds

First, a repo needs to be initialised for this project. Dockerhub is very particular how to do this. Go to Dockerhub and create an account (if you don't have one), and a repo with the _exact_ name of your cookiecutter project. So in this example

`pl-sevstack`

Link it to a GitHub account that has the same repo name and do the following to upload your code to GitHub and build container images

`cd pl-sevstack`

`git init`

`git add .`

`git commit -m 'First commit'`

`git remote add origin https://github.com/JonoCameron/pl-sevstack.git`

`git push origin master`

`git tag 0.2`

`git push --tags`

This will push your new app to GitHub. Next, push it to Dockerhub:

`docker build -t jonocameron/pl-sevstack:0.2`

`docker push jonocameron/pl-sevstack:0.2`

Then I made it a plugin on the ChRIS store:
https://chrisstore.co/

To automate builds and updates to container images  each time a push is made to GitHub, I followed this guide here:
https://github.com/FNNDSC/cookiecutter-chrisapp/wiki/Automatic-Builds

To add secrets to my GitHub actions.


# 1/3/21

Today I added enabled automatic pushes and builds to dockerhub from github on my plugin.

To do this I added the following secrets to the github repo github.com/JonoCameron/pl-sevstack

`DOCKERHUB_USERNAME`

`DOCKERHUB_PASSWORD`

It also required a re-run of the cookie cutter to select 

`1 - publish to github automatically`

# 2/3/21

Today I will make a plugin that takes an input file 

`in/randomnumbers.txt`

 (in the input directory of the plugin) of randomly generated numbers, tab/comma/new line separated I haven't decided, sorts these numbers and outputs them to the output directory. 

 `out/sortednumbers.txt`

 I will start by populating 

 `in/randomnumbers.txt` 

 with 

 `generaterandom.py`

In the end, I managed to use python IO operations to read the file, and then used numpy to sort the numbers, after a bit of formatting(remove newlines, int(string), etc..) I could not work out how to output to the output directory, and this is something I'm going to follow up with the client on.

# 4/3/21

Today I will get a local instance of ChRIS backend running, so that I can test covidnet.sh.

To run the ChRIS backend with no fuss: https://github.com/FNNDSC/ChRIS_ultron_backEnd/blob/master/README.md

`git clone https://github.com/FNNDSC/ChRIS_ultron_backend`

`./unmake.sh ; sudo rm -fr FS ; rm -fr FS ; ./make.sh`


`./unmake.sh ; sudo rm -fr FS ; rm -fr FS ; ./make.sh -U -I -i`

# 11/3/21

In the first terminal I ran 

`./unmake.sh ; sudo rm -fr FS ; rm -fr FS ; ./make.sh -U -I -i`# 28/9/20

Met with rudolph. discussed project and ourselves. He showed us his work pc with 1TB RAM
The project is not to develop a neural network as much but port the application to the MOC, power9 processor and to make the application point and click so that doctors can use it.


# 29/9/20

Met with the team over Discord. Discussed a possible timeline for our project.

Step 1: Take the python inference script out of the terminal and into a GUI so that a human who doesn't know how to use the command line can use it in a "point and click" way. Aim to have this working by the November prototyping window.

Step 2: Port the application so that it works on the MOC. I'm not entirely sure what this means so there will need to be individual research conducted here. I think we need to make it work on Docker so we can put the Docker environment on the MOC

Step 3: Get it working on power9, which means converting it from CISC x86_64 to RISC, whilst still on the MOC. Yeah, IDK either.

Kind of daunting. I guess start on the research and then go from there.

Research topics to go over: 
MOC
Docker
Power9
Making an application for a python script


# 1/10/20

Also need to implement an algorithm that presents the most likely cases at the top of a list

First IDR today..

# 24/2/21

This is my documentation of how to create a new cookiecutter plugin for use with ChRIS, using the information provided here:

https://github.com/FNNDSC/cookiecutter-chrisapp

The machine I am using is Ubuntu 20.10

Start by creating and going into a directory for your plugin

`mkdir sevStack`

sevStack being the stack of ranked severity reports produced by pl-covidnet.

`cookiecutter https://github.com/FNNDSC/cookiecutter-chrisapp.git`

This will give a few options to fill out, and you can fill them out as you see fit. Just make sure the 

`app_repo_name`

option is all lowercase, because when the app goes to Dockerhub, Dockerhub will only accept containers that are lowercase.
When asked

`Select app_type`

I selected

`1 - ds`

because that will create a branch of a tree of nodes.

When asked 

`Select platforms`

`1 - linux/amd64`

`2 - linux/amd64, linux/ppc64le`

`3 - linux/amd64, linux/ppc64le, linux/arm64`

I selected option 3 because we will be porting the app to the MOC which has powerPC architecture, so this will build this for us for free, and #3 has support for Docker if it ever goes to Android.
That should be it! You now have a new plugin.

`ls`

Will show

`pl-sevstack`

Move into the new directory the cookiecutter made for you.

`cd sevstack`

`ls -la`

Will show what cookiecutter populated the directory with, with the name you chose for the app as a new directory within there. This contains

`sevstack.py`

Which is where the code for the plugin lives.

`nano sevstack.py`

To see what is in there. You'll see it includes an example of a bare bones execution.

From the README, there is a command to build the Docker container

`docker build -t local/pl-sevstack .`

If you haven't added your user to the Docker group, you will have to use 

`sudo docker build -t local/pl-sevstack .`

But we can easily add our user to the docker group

`sudo groupadd docker`

`sudo usermod -aG docker $USER`

Restart your machine and then test that you have been added to the docker group and can run without sudo.

`docker run hello-world`

Now build the docker container with the plugin. (Make sure to change back to the directory you were working in)

`docker build -t local/pl-sevstack .`

Now we have a built container, we can go to the README again and find the dummy command to execute. We'll need to create an in and out directory so that the app can actually do something, as well as touch up the dummy command so it works properly.

`mkdir in out`

You'll need to use some sanity checking, like making sure input and output arguments are appropriate, as well as directory paths and whatever you've named your plugin.

`docker run --rm -u $(id -u)                      \ `

`-v $(pwd)/in:/incoming -v $(pwd)/out/:outgoing   \ `

`local/pl-sevstack sevstack                       \ `

`/in /out                                           `

Edit your README under the "Run" heading and save it there so you always have it, then you can copy/paste it in your command line
Since we haven't done anything with 

`sevstack.py`,

Running the container will just print its name and version number to the terminal. We can go ahead and edit it to print

`Hello World` 

by editing this file, so go to your favourite text editor and do just that. Add it under 

`def run(self, options)`

Rebuild using 

`docker build -t local/pl-sevstack .`

And run again using the command you edited in the README and somewhere around the print out of the apps name, you will see your "hello world" message!

Now, building a Docker container is time consuming so an easy way to run the .py file is to run it in a virtual environment.
Create it and activate a virtual environment with

`python3 -m venv ss-env && source ss-env/bin/activate`

Change into the directory with the setup.py file

`cd pl-sevstack`

And to install to your python virtual environment

`pip install -e .`

`sevstack --man`

Shows it has been installed to the environment.

To build the plugin with a debugger, add to the top of sevstack.py under 

`from chrisapp.base import ChrisApp`

`import pudb`

Once you have done that, try to re-install and run

`pip install .`

`sevstack --man`

If you haven't added pudb to requirements.txt, or installed pudb in your virtual environment:

`pip install pudb`

And in requirements.txt, add

`pudb~=2021.1`

So that when you rebuild the container, it will include pudb.

`docker build -t local/pl-sevstack .`

Map your source code live into the container with a new command. I put it in the README with the other command we edited so I can get at it easily. For me it worked best if I copied it out of 

`nano README.rst`

But YMMV

`docker run --rm -u $(id -u)                                                                                            \ `

`-v $(pwd)/sevstack:/usr/local/lib/python3.9/site-packages/sevstack:ro -v $(pwd)/in:/incoming -v $(pwd)/out/:outgoing   \ `

`local/pl-sevstack sevstack                                                                                             \ `

`/in /out                                                                                                                 `      

This command means that you can just run this without rebuilding the container everytime you change the source code :)
Remember pudb? Here's how to run it. Add 

`pudb.set_trace()`

In your source code, where you want a break to happen. Then run the container with the `-ti` flag for interactive mode.

`docker run --rm -u $(id -u) -ti                                                                                        \ `

`-v $(pwd)/sevstack:/usr/local/lib/python3.9/site-packages/sevstack:ro -v $(pwd)/in:/incoming -v $(pwd)/out/:outgoing   \ `

`local/pl-sevstack sevstack                                                                                             \ `

`/in /out                                                                                                                 `

# 25/2/21

Run pudb without error messages by removing the 

`-u $(id -u)`

From the first line of this latest command.

### Automatic builds

First, a repo needs to be initialised for this project. Dockerhub is very particular how to do this. Go to Dockerhub and create an account (if you don't have one), and a repo with the _exact_ name of your cookiecutter project. So in this example

`pl-sevstack`

Link it to a GitHub account that has the same repo name and do the following to upload your code to GitHub and build container images

`cd pl-sevstack`

`git init`

`git add .`

`git commit -m 'First commit'`

`git remote add origin https://github.com/JonoCameron/pl-sevstack.git`

`git push origin master`

`git tag 0.2`

`git push --tags`

This will push your new app to GitHub. Next, push it to Dockerhub:

`docker build -t jonocameron/pl-sevstack:[TAG] .`

`docker tag local/pl-sevstack:[TAG] jonocameron/pl-sevstack:[TAG]`

`docker push jonocameron/pl-sevstack:[TAG]`

Then I made it a plugin on the ChRIS store:
https://chrisstore.co/

To automate builds and updates to container images  each time a push is made to GitHub, I followed this guide here:
https://github.com/FNNDSC/cookiecutter-chrisapp/wiki/Automatic-Builds

To add secrets to my GitHub actions.


# 1/3/21

Today I added enabled automatic pushes and builds to dockerhub from github on my plugin.

To do this I added the following secrets to the github repo github.com/JonoCameron/pl-sevstack

`DOCKERHUB_USERNAME`

`DOCKERHUB_PASSWORD`

It also required a re-run of the cookie cutter to select 

`1 - publish to github automatically`

# 2/3/21

Today I will make a plugin that takes an input file 

`in/randomnumbers.txt`

 (in the input directory of the plugin) of randomly generated numbers, tab/comma/new line separated I haven't decided, sorts these numbers and outputs them to the output directory. 

 `out/sortednumbers.txt`

 I will start by populating 

 `in/randomnumbers.txt` 

 with 

 `generaterandom.py`

In the end, I managed to use python IO operations to read the file, and then used numpy to sort the numbers, after a bit of formatting(remove newlines, int(string), etc..) I could not work out how to output to the output directory, and this is something I'm going to follow up with the client on.


# 4/3/21

Today I will get a local instance of ChRIS backend running, so that I can test covidnet.sh.

To run the ChRIS backend with no fuss: https://github.com/FNNDSC/ChRIS_ultron_backEnd/blob/master/README.md

`git clone https://github.com/FNNDSC/ChRIS_ultron_backend`

`./unmake.sh ; sudo rm -fr FS ; rm -fr FS ; ./make.sh`

## Aside into why I was struggling to do python IO operations the other day....

I wasn't mapping inputs and outputs correctly in this command:

`docker run --rm -u $(id -u)                                                                                            \ `

`-v $(pwd)/sevstack:/usr/local/lib/python3.9/site-packages/sevstack:ro -v $(pwd)/in:/incoming -v $(pwd)/out/:outgoing   \ `

`local/pl-sevstack sevstack                                                                                             \ `

`/in /out                                                                                                                 `      

So volume mapping has to be specified as `<hostDir>:<containerDir>` otherwise the plugin won't find the directory it wants outside of the container. The two directories in the last line have to match the directories in the second line. i.e, here `/in` is the host machine directory and it is mapping into the container at `/incoming` and then `/outgoing` in the container is mapped to `/out` on the host machine

## Back to ChRIS backend

I started working from: https://github.com/FNNDSC/ChRIS_ui,
with the instruction:

`$ docker run --rm -it -v $(pwd):/home/localuser -p 3000:3000 -u $(id -u):$(id -g) --name chris_ui fnndsc/chris_ui:dev`

This first pulls the docker image if it isn't found locally and then runs it at 

`https://localhost:3000`

But it doesn't do anything apart from be there and look good


# 5/3/21

Today I will continue on with the ChRIS_ultron_backend service. In my first terminal I did:

`git clone https://github.com/FNNDSC/ChRIS_ultron_backend`

`cd ChRIS_ultron_backend`

`./unmake.sh ; sudo rm -fr FS ; rm -fr FS ; ./make.sh`

In a second terminal I did:

`git clone https://github.com/FNNDSC/ChRIS_ui`

`cd ChRIS_ui`

`docker run --rm -ti -v $(pwd):/home/localuser -p 3000:3000 -u $(id -u):$(id -g) --name chris_ui fnndsc/chris_ui:dev`

Then in a third terminal I ran:

`cd ChRIS_ultron_backend`

`./postscript.sh`

Once these steps have been finished, go to localhost:3000 and check out the ChRIS ui. Log in with chris:chris1234.
I added local files just to see it do something.

## Solving issues with ./covidnet.sh failing to find plugin IDs

Make sure to run the backend with

`./unmake.sh ; sudo rm -fr FS; rm -fr FS; ./make.sh -U -I -i`

The flags will help include the plugins that 

`./covidnet.sh -a localhost`

Needs to run. But we also need to add those plugins to postscript.sh, so in the top list of plugins needed, add `pl-covidnet`, `pl-pdfgeneration`, `pl-lungct` and `pl-med2img`

Then run

`./ChRIS_ultron_backend/postscript.sh`

before `./CHRIS_docs/workflows/covidnet.sh`

Now if you go to localhost:3000 you will see a new feed called "COVIDNET_lung_CT_subjects" and if you click on that you'll be able to watch ChRIS work through that workflow.

# 11/3/21

## Adding my plugin, `pl-sevstack` to my local ChRIS environment.

First I'm going to try adding 

`local/pl-sevstack` to `postscript.sh` 

Then I ran 

`./postscript.sh`

and that added `pl-sevstack` to my ChRIS instance. From there I was able to use the frontend to upload `randomnumbers.txt` as the root node of a workflow and run `pl-sevstack` on it, and obverse in the UI that the plugin had outputted `sortednumbers.txt`, the same way that the local instance ran.

Now I will test running it from Dockerhub. I will do this by first pushing to Dockerhub, then replacing `local/pl-sevstack` with `jonocameron/pl-sevstack`

`docker tag local/pl-sevstack:[TAG] jonocameron/pl-sevstack:[TAG]`

`docker push jonocameron/pl-sevstack:0.7`

Then I rebooted my ChRIS instance to make sure that the `local/pl-sevstack` wasn't being used. After completing this, I found that version 0.4 of my plugin was being pulled from dockerhub rather than the most recent 0.7. So now I will try and work out how to pull the most recent version of pl-sevstack.

I'm going to try 

`docker tag local/pl-sevstack:0.7 jonocameron/pl-sevstack:latest`

`docker push jonocameron/pl-sevstack:latest`

By tagging my most recent as the latest on dockerhub, I hope this will resolve my issue hard-coding version numbers into `postscript.sh`.

That worked. However, I would not need to hack around it like that if the automatic builds worked between GitHub and Dockerhub. Why don't they work?? Because I am hard-coding a filename (`sortednumbers.txt`) into my plug-in, GitHub actions looks for a file of that name whilst building, which it subsequently fails on. Therefore to fix the automatic builds I need to make my plugin take any generic name and operate on that file.

# 18/3/21

I've been in quarantine since the 15th.

Since losing my work laptop, our client set me up with a machine at his workplace, by setting up ssh tunnels. I logged it by:

1. Setup an ssh tunnel for direct access:

`ssh -g -f -N -X -p 7778 -L 2222:localhost:2222 chris@108.49.45.138`

The 'chris' passwd is
chris1234

2. Setup another tunnel for the ChRIS UI:

`ssh -g -f -N -X -p 7778 -L 3333:localhost:3333 chris@108.49.45.138`

3. You will also need to setup another tunnel to get to the ChRIS backend

`ssh -g -f -N -X -p 7778 -L 8333:localhost:8333 chris@108.49.45.138`

Once these tunnels were set up, I logged in to the machine with

`ssh -p 7778 chris@108.49.45.138`

Then created a directory on the 'gondwanaland' machine for myself to work in.

`ssh -g -f -N -X -p 7778 -L 2222:localhost:2222 chris@108.49.45.138 && ssh -g -f -N -X -p 7778 -L 3333:localhost:3333 chris@108.49.45.138 && ssh -g -f -N -X -p 7778 -L 8333:localhost:8333 chris@108.49.45.138`

# 24/3/21

## Restoring tunnels to Megalodon

`ssh -g -f -N -X -L 2222:localhost:22 chris@192.168.1.216`

`ssh -g -f -N -X -L 3333:localhost:3000 chris@192.168.1.216`

`ssh -g -f -N -X -L 8333:localhost:8000 chris@192.168.1.216`

## Testing pl-covidnet output

Whilst figuring out the output of pl-covidnet I deleted a few lines from `FNNDSC/CHRIS_docs/workflows/covidnet.sh` to remove the pl-pdfgeneration part of the workflow. I did this so I could understand how the script was written and so I could see easily the output of pl-covidnet.

Each sub-directory has input/output meta data that we don't care about, `sample.png` which the image file the machine learning algorithm works on and the `prediction-default.json`. If `prediction-default.json` has "covid" as the prediction, it also has a `severity.json`, which we will use to rank the patients.

This means that we will be ranking patients on covid first and the pneumonia second.

## Creating a topological plugin (ts plugin)

I went back through the cookiecutter again to make a new plugin that is essentially a sink node that will catch the output of many instances of pl-covidnet and put all of those sub-directories in one directory to be used.

I started by getting a "hello world!" from it.

After some research here: `https://github.com/FNNDSC/pl-tsdircopy`

I will try creating some dummy input directories and copying them to a single output directory.


# 30/3/21

Login to Megalodon from Gondwanaland with 

`ssh -p 22 chris@localhost`

# 31/3/21

Log in to Megalodon from local machine with

`ssh -p 2222 chris@localhost`

Today the goal is to get the to fix this problem:
Today I added pl-topologicalcopy to the covidnet.sh workflow. I did this by editing the attached covidnet.sh file.

pl-topologicalcopy is a plugin that allows us to collect the output directories of many ChRIS plugins and put them into one singular output directory that we can then work on. Think of a sink node pulling everything into one place.

On lines 41-44 I linked the new plugin to the previous level of the workflow, and declared its variables (--previous_id=@prev_id)
![image info](Capture1.PNG)
And on lines 573-577 I corrected the logic so that pl-topological had the correct input arguments so it could run.
![image info](Capture2.PNG)
Then you can see here that result-->[pinst: 991] shows a successful running of pl-topologicalcopy, and this was verified by
![image info](Capture3.PNG)
Where it says "status finishedSuccesfully"

Unfortunately, I only managed to make it work for a single branch of the covidnet.sh workflow, so I have reached out to some members of my clients team to ask questions about whether the input "combined" on line 573 needs to be looped through to collect all branch outputs.

See here that,
![image info](Capture4.PNG)

A) the plugin has been added, and,

B) the directory of the pl-topological plugin only has the output from the final branch, the branch that that node is linked to.

# 2/4/21

Changing the prev_id of pl-topologicalcopy to the last instance of pl-covidnet made the output of pl-topologicalcopy work, but since the outputs of pl-covidnet all have the same name, everything is overwritten. I'm going to need to edit the pl-covidnet to output a single directory with a unique identifying name. I think stamping the folder with a time stamp will be good, if I can't prefix the output with the plugin instance ID.

# 3/4/21

## Editing local/pl-covidnet to produce single, uniquely named output.

I copied pl-covidnet into pl-sevstack. Then I changed all of the necessary book keeping files so it would upload to ChRIS.
This included sevstack.json, setup.py, the README and the Dockerfile. After this I edited pl-sevstack so that it would reflect that I had changed it to mostly pl-covidnet

# 4/4/21

Today I edited tcovidnet.sh, the shell script that runs the covidnet workflow.

![image info](Capture7.PNG)

On lines 28-33 I replaced pl-covidnet with a edited version of itself, pl-demo. This was so that pl-pdfgeneration could work along with pl-topologicalcopy, both of which are nodes which have pl-covidnet/pl-demo as their parent node.

On lines 46-48 I added our ranking algorithm, jonocameron/pl-rank to rank the output of pl-topologicalcopy. 

# 5/4/21

Why we need this for ECE 

Say we get identical results. provides opportunity for hospitals to run advanced computations.

say that AWS/MOC executes large volumes of scans and does it quicker than weaker hospital machines.

can we get a time output for the MOC vs. AWS

Buzzword: hybrid cloud

# 17/4/21

Today I tested a more final version of our workflow on our clients remote compute. It worked as it should have done.

# 19/4/21

# 26/4/21

ssh -g -f -N -X -p 7778 -L 2222:localhost:2222 chris@173.48.102.146 && ssh -g -f -N -X -p 7778 -L 3333:localhost:3333 chris@173.48.102.146 && ssh -g -f -N -X -p 7778 -L 8333:localhost:8333 chris@173.48.102.146   

From `~/ChRIS_ultron_backend`

Then immediately added plugins be running 

`./postscript.sh`

In a second terminal I 

`docker run --rm -it -v $(pwd):/home/localuser -p 3000:3000 -u $(id -u):$(id -g) --name chris_ui fnndsc/chris_ui:dev`

This connects the ChRIS backend to the ChRIS_ui frontend, so that plugins can be run from the ChRIS UI.
The frontend can be found at `localhost:3000`

Finally to run the covidnet workflow:

`git clone https://github.com/FNNDSC/CHRIS_docs`

`cd CHRIS_docs/workflows`

`./covidnet.sh`