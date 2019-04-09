# Azure Bot Framework - Create new repository and commit code

### This guide will help you get local debugging working so you can step through code. Then we'll deploy your bot using CI/CD against your code repository. 

When you've completed this tutorial, you should expect to see this:
<br/><img src="../screens/echo_bot_deploy_test_webchat.jpg" /><br/><br/>

### Section 1: Download the Generated Boilerplate Code

<!--1. Browse to [https://portal.azure.com](https://portal.azure.com) and log in-->

1. Create a new repository to house your source code at http://github.com

1. Initialize your local project to use git:
	- go to your project root in terminal and type the following commands
		```
		git init
		git add .
		git commit -m "Initial commit"
		git remote add origin https://github.com/USERNAME/EurekaBot.git
		```
	- 


1. We can also test using the Bot Framework Emulator - just select the `production` endpoint this time and send a message
<br/><img src="../screens/bot_framework_echo_production.jpg" />

Congrats! You are now free to make changes, commit and push them and your changes will automatically be deployed to your public instance
