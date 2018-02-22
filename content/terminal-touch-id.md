Title: Adding TouchID support to Terminal

I use terminal a lot and I have the new Macbook Pro from 2017 with a touch bar. Currently most of the apps I use support TouchID for authentication which makes my life easier as I dont have to type my password. Unfortunately, Terminal app in OSX does not support Touch ID by default. I found a nice and quick way to enable this feature. Here is the code snippet.

```shell
$ sudo vi /etc/pam.d/sudo
```
add the following code on line 2

```
auth sufficient pam_tid.so
```

save and exit. Now everytime I use sudo I see a prompt on the screen to use my fingerprint.