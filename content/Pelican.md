Title: Pelican Setup
Tags: blogging, tools

Blogging with Pelican is amazing. Just a note to self on how to setup one. I tested this on Ubuntu 16.04 with python 2.7. I created the website and set it up on AWS-S3.

```
$ sudo apt-get install python-pip
$ sudo pip install pelican Markdown typogrify s3cm
```

####Create a directory to host website. I created one under home.

```
$ mkdir mytest-website
$ cd mytest-website
$ pelican-quickstart
$ cd content
$ touch hello-world.md
```

####Some local testing before I upload to S3
```
$ make devserver
```

####Setup s3 using s3cmd —configure

```
$ make s3_upload
```

Only the changes get uploaded. Next I will figure out how to change the default template.