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
sudo scp ~/Desktop/DWA/humans.txt nashsingh@104.236.87.252:/var/www/html/
```
You will then be asked for the remote server user's password. If the prompt is **not** for the server user's password first, enter the local machine's password to continue. If successful, you should be able to then spin up the server with:

```
ssh nashsingh@104.236.87.252
```
Then, navigate in your browser to your server's address where you should be able to view the file

```
http://104.236.87.252/humans.txt
```