# Documentation for AWS Server Setup
## by Steven Slezak
## 15 January 2018

The *html* of this RMarkdown file [can be found here.](http://www.rpubs.xxx.xxx.xxx)

## Overview 

This toolkit is designed to serve as documentation for the procedure to set up cloud-based systems on Amazon Web Servics (AWS) for the purpose of running Docker and Jupyter on that platform.  It represents a schema for a data science infrastructure that makes use of AWS, Docker, Jupyter, and R.

The schema should make it possible to set up the infrastructure from scratch, using the step-by-step instructions for configuring the necessary pieces.  In general, the instructions will follow these steps:

1. How to Create a Key Pair
2. How to Create a Security Group
3. How to Create a New Instance in EC2
4. How to Launch the New Instance
5. How to Configure the New Instance for Docker

Besides instructions, the toolkit includes a diagram of the system and a budget for running a Jupyter Data Science Notebook Server on AWS.

The sequence of steps will be as follows:

Step 1:  Set Up AWS Account

Step 2:  Create the *ssh* Key Pair

Step 3:  Import the Public Key to AWS

Step 4:  Create a New Security Group

Step 5:  Choose Amazon Machine Image (AMI)

Step 6:  Configuring the New Instance for Docker

Step 7:  Configuring the Jupyter Notebook

Step 8:  Resetting After Stopping AWS

## Step 1: Set Up AWS Account

Each user needs to set up an account with Amazon Web Services for access to their cloud system and support.  AWS Documentation for setting up the account can be [found here.](https://docs.aws.amazon.com/lambda/latest/dg/setup.html)  

**PRO-TIP**  Please keep in mind that Amazon may take up to 24 hours to set up your account, so give yourself sufficient time to get started.  Don't expect to open the account and get right to work.

It is a good idea to be careful with the AWS identity and security measures you are about to set up.  Best practices for the secure management of AWS resources, including AWS Identity and Access Management (IAM) can be [found here.](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

## Step 2:  Create the *ssh* Key Pair

The infrastructure we are setting up will allow your local computer communicate with the AWS servers remotely.  In order to do this, you will need to create a secure key pair that will make it possible for the AWS servers to confirm your identity when you open a remote session.  

The pay consists of two parts -- a public key and a private key.  Your computer needs to know both.  AWS needs to know the public key.  The local private key will interface with the AWS public key to confirm your identity to the system.  This pair is known as *ssh* for "secure shell."

You will need to generate the new key pair using your computer's shell.  For my purpose, I used *Git Bash*.

Make a note of the directory you are operating in before you create the key pair.  You want to be able to find it easily.  Or create a directory for this purpose.  When you are certain you are in the right directory, enter the command:

```{r Key Pair}
#       $ ssh-keygen -t rsa
```

This will generate and save in your system the public and private keys in the key pair.  The private key will be saved as *id_rsa* while the public key will be saved as *id_rsa.pub*.  

You need to verify the new key because you will need to copy the entirety of the public key to input with AWS.  The following command will do this:

```{r Verify Public Key}
#      $ cat ~/.ssh/id_rsa.pub
```

If you need to view the key at some other time, you can also use this command:

```{r View Public Key}
#       $ view id_rsa.pub
```

In either case, copy the entire code from *ssh-rsa* to the end of the string and paste to a text editor for use in AWS.

Now you have your key pair, go back to AWS to configure the data science system.

## Step 3:  Import the Public Key to AWS

Once the AWS account is set up and available, sign into the console and go to the *AWS Services* section.  There, you will find under *All Services* a link for *EC2*.  *EC2* stands for Elastic Compute Cloud.  Click on the *EC2* link and you will be directed to the *EC2* control panel.

**PRO-TIP**  Before starting the set up, look at the bar at the top of your browser.  On the right will be your name and a georgraphic area.  You are setting up your system in a particular geographic server area, so make sure you pick one and stick with it.  You cannot access your system if the setting is pointing towards an area different than the one you started with.

You will notice under *Resources* that the *Key Pairs* indicates zero.  Click the *Key Pairs* link and choose *Import Key Pair* on the following page.   Name your key pair.  Usually a date and some other identifier is good enough.  

Copy your public key from the text editor and paste it into the *Pbulic key contents* box.  Make sure you capture the entire key.  IF you are satisfied you copied it correctly, click *Import* and you are done.  Don't worry if there is an error.  You can always delete this key and generate a new key pair following the method described above.

## Step 4:  Create a New Security Group

Go back to the *EC2* Dashboard and click on *Security Groups*.  You are going to configure this next.

Click *Create Security Group*.  Give your group a good name and a useful description but leave the VPC (Virtual Private Cloud) at its default setting.  Next, choose the Inbound tab where you will set four Rules for your configuration.  These Rules will apply to *SSH*, *Jupyter*, *Docker Hub*, and *Mongo*.  The only changes you make will be to the columns *Type*, *Port Range*, *Source*, and *Description*.

When you are done your Inbound settings should look something like this:

```{r Table 1, echo = FALSE}
Table1 <- tibble::tribble(
        ~"Type", ~"Protocol", ~"Port Range", ~"Source", ~"Description",
        "SSH", "TCP", 22, "Anywhere", "SSH", 
        "HTTP", "TCP", 80, "Anywhere", "HTTP",
        "Custom TCP", "TCP", 8888, "Anywhere", "Jupyter", 
        "Custom TCP", "TCP", 2376, "Anywhere", "Docker Hub", 
        "Custom TCP", "TCP", 27016, "Anywhere", "Mongo"
)
knitr::kable(Table1, caption = "AWS Security Group Settings")
```

**PRO-TIP**  Don't be alarmed if the settings appear doubled up on the Inbound tab when you are done.  This is not unusual and doesn't impact functioning.

You are now ready to launch the instance.  Click on the *Launch Instance* button to begin.

## Step 5:  Choose Amazon Machine Image (AMI)

The first step in launching the new instance is selecting an AMI.  There are many servers available to choose with different operating systems, storage drive types, storage sizes, architectures, and so on.  Some of these are "free tier eligible" meaning they are free for an introductory period and cost very little to operate after that.  These are the simplest and most basic devices.  

For purposes of illustration, we will set up a simple AMI. We will not complicate the set up at this point with tags, custom security selections, or concerns about network performance.  You can come back later to customize the server set up once the machine is spinning and operating as expected.

Select **Ubuntu Server 16.04 LTS (HVM), SSD Volume Type - ami-45ead225** from the list. Since we are using a "free tier eligible" set up, there is only one instance type available, a General Purpose server of the t2 micro type.  Select it and move on to the fourth step, "Add Storage".  We will skip Step 3, "Configure Instance".

AWS offers upt to 30 GB of EBS General Purpose (SSD) storage free of charge each month.  In keeping with our free tier eligiblity, change the "Size (GiB" selection to 30.  Leave all other choices as default.

Skip Step 5 for now.

For Step 6, "Configure Security Group", click on "Select an existing security group".  This will display your settings from Step 4 above. Select the group you just set up, confirm the settings are as you need them, and move on to Step 7, "Review". Select the *Review and Launch* button at the bottom of the screen or use instead the *Review* link located near the top on the right-hand side.  

Since we did not customize security settings, there should be a security warning across the top of the screen.  This is to be expected.  Review the information under Instance Type.  If all is in order, go ahead and click "Launch" to create the server.

AWS will ask you at this point to select an existing key pair or create a new pair.  Since we already loaded the public key into AWS in Step 3 above, select "Choose and existing key pair" in the top box then highlight the pair you created earlier in the second box.  Click the acknowledgement and then "Launch Instances".

Congratulations! Your instance is now running.

Next, you have to connect your local computer to the remote AWS system you just created.

## Step 6:  Configuring the New Instance for Docker

We use Docker to connect the Jupyter Notebooks to AWS and to manage our AWS server, so we need to configure the AWS instance we just created to work with Docker.  

Start by copying the IP address for the AWS instance.  Go to the AWS *EC2* Dashboard and look at the column menu on the left hand side.  Click on Instances and it will direct you to a list of instances running on the system.  Scroll to the right and you will find the IP address associated with the instance you created.  Copy it.

**PRO-TIP**  The IP address is constant until you stop the instance.  When it is restarted, a new IP address is issued.  It is necessary to keep track of the changes in order to get Docker and Jupyter Notebooks to work with AWS.  More on this later.

Go back to the shell you set up on the local computer.  At the command prompt, type the following and paste the IP address you copied from AWS after the @:

```{r ssh Command1}
#      $ ssh ubuntu@54.67.41.212
```

Next, you need to download the Docker script to the local machine.  This is done with the following command:

```{r curl Command}
#      $ curl -sSL https://get.docker.com | sh
```

The piped *sh* switch is optional but it loads the script directly into the shell.  It will take a while for the script to download so make sure the process is complete before proceeding to the next step.

The *ubuntu* user needs to be able to give commands to Docker so it must be added to the Docker Group with this command:

```{r sudo Command}
#      $ sudo usermod -aG docker ubuntu
```

Time to reboot to let the changes take effect.  This can be done with the following command or by simply closing the shell and reopening it.

```{r Reboot}
#      $ sudo reboot
```

Once the shell reopens, type the *ssh* command from above using the appropriate IP address:

```{r ssh Command2}
#      $ ssh ubuntu@54.67.41.212
```

And check that Docker is in fact running.  The version will display if all is working well.

```{r Docker -v Command}
#      $ docker -v
```

## Step 7:  Configuring the Jupyter Notebook

Now the system you have set up is ready to connect with Jupyter Notebook.  In this step you will conduct a series of commands to set up Jupyter Notebook and get it to display in a browser.

First, you need to pull Jupyter Notebook with this command:

```{r pull Command}
#      $ docker pull jupyter/datascience-notebook
```

To confirm that Jupyter Notebook has been loaded onto the system, use this command:

```{r confirm Command}
#      $ docker images
```

The next command is going to run Jupyter and generate a long passcode.  Make a note of the first four digits of the code string.

```{r run Command2}
#      $ docker run -v/home/ubuntu:/home/jovyan -p 80:8888 -d jupyter/datascience-notebook
```

The string you have generated is called the container ID and looks something like this:  

```{r string}
#       ef8e4dced644ad4f5b1ef175bf567b123ae91f55d84d9d22d52511f16db7a118
```

In the next command, insert your four digits in the space indicated *ef8e* and execute.

```{r exec Command}
#      $ docker exec ef8e jupyter notebook list
```

This will generate a url and some additional information.  Copy the url up to (but do not include) the double colon.  Paste the url into your web browser.  The url contains the passcode you will need to access the Jupyter Notebook.

Notice the phrase *localhost* in the url.  In the browser, delete that phrase and paste in its place the current IP address for the AWS *EC2* instance.

Hit Enter.  The Jupyter Notebook should open in the browser.  If so, you are good to go.

## Step 8:  Resetting After Stopping AWS

As mentioned earlier, if the AWS server is stopped the existing IP address will not be available when it is started up again.  A new IP address will be issued. This means you have to restart the Jupyter Notebook.  Unfortunately, this is not as simple as pasting the new IP address into the browser bar.  You must repeat a series of commands.  Please follow instructions above for these commands and remember to use the new IP address from AWS and the new passcode string generated in the shell:

```{r restart Command}
#      $ ssh ubuntu@54.67.41.212

#      $ docker -v

#      $ docker run -v/home/ubuntu:/home/jovyan -p 80:8888 -d jupyter/datascience-notebook

#      $ docker exec xxxx jupyter notebook list
```

Then copy and aste the url into your web browser. Be sure to update *localhost* with the new IP address.

The Jupyter Notebook should open in the browser. 

## Appendix One:  Schematic of the AWS Infrastructure
![](https://github.com/seslezak/UCLA-Data-Science/blob/master/Resources/Project_1/AWS%20Schematic.png)



## Appendix Two:  Jupyter Notebook Budget (one fiscal quarter)


