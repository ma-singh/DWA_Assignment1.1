# Description

This is a demonstration of setting up a LEMP stack with WordPress on a remote server, and transfer files with Secure Copy Protocol to a remote server.

## Prerequisites

On your local machine, you must have
* [PHP v5.4+](http://php.net/manual/en/install.php)

If you need to recreate the server for your own testing purposes, you can do so by following this [guide](https://github.com/ma-singh/DWA_Assignment1.1/blob/master/ServerSetup.md)

# Local Installation

From the terminal of your local machine, clone this repository to a project directory of your choice
```
git clone https://github.com/ma-singh/DWA_Assignment1.1.git
```

Make sure you change into that project directory after cloning the repository
```
cd <PATH/TO/REPOSITORY/>
```

## Deploying Files with SCP

Open up a terminal window, and find the file that you want to deploy from your local machine to your remote.

```
$ pwd
/Users/nashsingh/Desktop/DWA
$ ls
humans.txt
```
To transfer just a single file using `scp` you do not need to use `-r` as it is not recursive.

---
The format for an `scp` transfer is

```
scp /path/to/file/filename.ext username@serverdomain:[/target/file/path/]
```
As we want to transfer our file to our document root on our remote machine, we will be using the following:

```
sudo scp ~/Desktop/DWA/humans.txt <USERNAME>@<IP_ADDRESS>:/var/www/html/
```
You will then be asked for the remote server user's password. If the prompt is **not** for the server user's password first, enter the local machine's password to continue. If successful, you should be able to then connect to the server with:

```
ssh <USERNAME>@<IP_ADDRESS>
```
Then, navigate in your browser to your server's address where you should be able to view the file

```
http://104.236.87.252/humans.txt
```
