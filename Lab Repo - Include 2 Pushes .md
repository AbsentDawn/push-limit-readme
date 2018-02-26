# Lab Repo - Include 2 Pushes 

## What is the 2 push limit? 

Git has functionality to execute commands upon the user carrying out specific actions. We have the ability to manipulate what happens when these actions are carried out. For example, git has a hidden folder called ```.git/hooks```. In this folder, there are bash scripts that run at specific events. The name of the file indicates the stage of their execution i.e. ```pre-commit``` will run before a user commits. To limit the number of pushes the ```pre-push``` file can be altered.

## Setup

### Creating a Pre-push file
A pre-push file needs to be created. Navigate to the lab repo and,
	
	mkdir .githooks
	
	# Creates the pre-push and add functionality, gets generated in the .git/hooks directory
	npm install --save-dev pre-push
	
	chmod +x .git/hooks/pre-push
	mv .git/hooks .githooks
	
>**NOTE:** The `pre-push` file must be outside of the .git folder as that folder is only for your local machine. However, the file itself will only work when it is in the `.git/hooks` directory. This is solved by using a NPM script which will be explained further on.


### Writing the script

Add the following code to the top of the file, just under the `#!/bin/bash` line. These are the variables setting up our next task of creating the limits. 
	
	# Max number of pushes 
	MAX_NUMBER=2
	
	# Reads in from this text files
	BASE_NUM="$(cat ../textfile.txt)"
	
	# Converts a string into an integer
	NUM="$(($BASE_NUM+0))"
	
To prompt the user to confirm whether or not they would like to push. Add this code to the bottom of the pre-push.

	read -p "You're about to push, are you sure you won't break the build? " -n 1 -r < /dev/tty
		echo
		if [[ $REPLY =~ ^[Yy]$ && $NUM -lt $MAX_NUMBER ]]; then
		    echo You are now pushing...
		    echo $BASE_NUM
		    echo
		    let BASE_NUM=BASE_NUM+1
		    echo $BASE_NUM > ../textfile.txt
		    exit 0 # push will execute
		else
		    read -p"You cannot push..." -n 1 -r < /dev/tty
		    echo
		    exit 1 # push will not execute
		fi

This will prompt the user to confirm their push. Textfiles to store the number of pushes will be created if non-existant. Otherwise, the number inside will be incremented on push.

### NPM script

In the `package.json` file, there is a object for adding custom scripts. Change the file to reflect the following:
	
	...
	
	"scripts": {
	    "test": "eslint .",
	    "spartalab": "mv ./.githooks/pre-push ./.git/hooks && npm install && git config --global user.name"
		},
	
	...
	
When the user types in `npm run spartalab <GITHUB_USERNAME>` the `pre-push` file will be moved to the correct place and the user will have their github userame added to the global git configuration (for slack messages).