Title: AWS Lambda and Gateway API with Zappa
Tags: aws, lambda, zappa, gateway, api

I develop a lot of API's as part of my job and part of it involves deploying them on publicly accessible servers for the global team to test. My current setup is as follows

Development: Python-Flask-MongoDB for local development

Deployment: Local server with globally accessible static IP or AWS EC2 or Heroku

As you can see, my deployment methodology is basically a pain in the neck since I have to keep maintaining either multiple copies or remote deployment for simple things. I am kinda OK with Heroku but we do not have a corporate account and hence I have to use my personal account. Hence, Heroku is off the table. Also, AWS EC2 is fine as long as I am not behind my corporate VPN. Sometimes, I use stupid things like opening a port and give access. Hence my new found setup.

###EXAMPLE API
Let me present an example of how I plan to migrate to AWS from local developement. Here is my sample API.

```
import flask
app = flask.Flask(__name__)

@app.route('/')
def hello():
    return "Hello World from Zappa"

@app.route('/saygreeting', methods=['GET','POST'])
def greeting():
    if flask.request.method == 'GET':
        name = flask.request.args.get('name', 'Sandy')
        return "{0}, Hello World from Zappa".format(name)

    if flask.request.method == 'POST':
        name = flask.request.form['name']
        return "Data has been saved successfully"

if __name__ == '__main__':
    app.run()
```

As you can see, there are 2 API calls here and this is how I test them. First, let me run the API.

```
$ python mytestapp.py
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
$ http http://localhost:5000
HTTP/1.0 200 OK
Content-Length: 22
Content-Type: text/html; charset=utf-8
Date: Mon, 22 Jan 2018 20:51:57 GMT
Server: Werkzeug/0.12 Python/2.7.14

Hello World from Zappa

$ http http://localhost:5000/saygreeting name==Andrew
HTTP/1.0 200 OK
Content-Length: 30
Content-Type: text/html; charset=utf-8
Date: Mon, 22 Jan 2018 20:53:19 GMT
Server: Werkzeug/0.12 Python/2.7.14

Andrew, Hello World from Zappa
$ http --form POST http://localhost:5000/saygreeting name=Andrew
HTTP/1.0 200 OK
Content-Length: 32
Content-Type: text/html; charset=utf-8
Date: Mon, 22 Jan 2018 20:54:05 GMT
Server: Werkzeug/0.12 Python/2.7.14

Data has been saved successfully
```

###ENTER ZAPPA
Zappa has the ability to take the existing local API and deploy it to aws API Gateway and Lambda. First some installations.

Zappa depends on virtualenv to work and hence lets create a sample env.

```
[/tmp] $ mkdir zappa_test
[/tmp/zappa_test] $ cd zappa_test
[/tmp/zappa_test] $ virtualenv env
[/tmp/zappa_test] $ ls
env/          mytestapp.py
[/tmp/zappa_test] $ source env/bin/activate
(env) [/tmp/zappa_test] $
```

Now that we have virtualenv setup and running, we need a few more installations for the API.

```
(env) [/tmp/zappa_test] $ sudo pip install flask zappa
```

Once this step is complete, we are deploy our API to AWS.

###ZAPPA DEPLOYMENT
####Step 0: Make sure your AWS credentials are saved in ~/.aws/credentials file.

####Step 1: Creating a zappa config file to setup the environment on AWS.

```
(env) [/tmp/zappa_test] $ zappa init

Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
What do you want to call this environment (default 'dev'): dev

AWS Lambda and API Gateway are only available in certain regions. Let's check to make sure you have a profile set up in one that will work.
Okay, using profile default!

Your Zappa deployments will need to be uploaded to a private S3 bucket.
If you don't have a bucket yet, we'll create one for you too.
What do you want call your bucket? (default 'zappa-fx31d2ot5'):

It looks like this is a Flask application.
What's the modular path to your app's function?
This will likely be something like 'your_module.app'.
Where is your app's function?: mytestapp.app

You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]: n

Okay, here's your zappa_settings.json:

{
    "dev": {
        "app_function": "mytestapp.app",
        "aws_region": "us-west-1",
        "profile_name": "default",
        "project_name": "zappa-test",
        "runtime": "python2.7",
        "s3_bucket": "zappa-fx31d2ot5"
    }
}

Does this look okay? (default 'y') [y/n]: y

Done! Now you can deploy your Zappa application by executing:

    $ zappa deploy dev

After that, you can update your application code with:

    $ zappa update dev

To learn more, check out our project page on GitHub here: https://github.com/Miserlou/Zappa
and stop by our Slack channel here: https://slack.zappa.io

Enjoy!,
 ~ Team Zappa!
(env) [/tmp/zappa_test] $ ls
env/                 mytestapp.py         zappa_settings.json
```

####Step 2: zappa deploy

```
(env) [/tmp/zappa_test] $ zappa deploy
Calling deploy for stage dev..
Downloading and installing dependencies..
Packaging project as zip.
Uploading zappa-test-dev-1516655568.zip (3.4MiB)..
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 3.58M/3.58M [00:00<00:00, 4.51MB/s]
Scheduling..
Scheduled zappa-test-dev-zappa-keep-warm-handler.keep_warm_callback with expression rate(4 minutes)!
Uploading zappa-test-dev-template-1516655573.json (1.6KiB)..
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1.62K/1.62K [00:00<00:00, 12.4KB/s]
Waiting for stack zappa-test-dev to create (this can take a bit)..
 75%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████                                                                                  | 3/4 [00:12<00:04,  4.63s/res]
Deploying API Gateway..
Deployment complete!: https://1uybd9nwpc.execute-api.us-west-1.amazonaws.com/dev
```

####Step 3: Testing zappa API

As you can see, zappa has successfully deployed my local API to AWS and provided an API endpoint. Let's test it out now.

```
(env) [/tmp/zappa_test] $ http https://1uybd9nwpc.execute-api.us-west-1.amazonaws.com/dev
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 22
Content-Type: text/html; charset=utf-8
Date: Mon, 22 Jan 2018 21:15:49 GMT
Via: 1.1 7bdfe469102431e5bc0938ba6b094eb7.cloudfront.net (CloudFront)
X-Amz-Cf-Id: y5TP1Wsgs-8Q8R0Rfqoad3Eg1v46ONM28aFYnp8KaTAk52qBlcTFOQ==
X-Amzn-Trace-Id: sampled=0;root=1-5a665485-68c2f04b23b744ed2c1e2ca0
X-Cache: Miss from cloudfront
x-amzn-Remapped-Content-Length: 22
x-amzn-RequestId: 6b299831-ffb9-11e7-b72e-5f9e514d7aeb

Hello World from Zappa
```

```
(env) [/tmp/zappa_test] $ http https://1uybd9nwpc.execute-api.us-west-1.amazonaws.com/dev/saygreeting name==Andrew
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 30
Content-Type: text/html; charset=utf-8
Date: Mon, 22 Jan 2018 21:16:16 GMT
Via: 1.1 3a9dca02f1ba6ecd49fee9a3ca7fcb81.cloudfront.net (CloudFront)
X-Amz-Cf-Id: YLstSAuAfsf3nQyJh8rE-Zz7Nev9X8UpsWtqhzkqLMOlyilH0QGo8g==
X-Amzn-Trace-Id: sampled=0;root=1-5a6654a0-b56fc3cb523bd290593d14c9
X-Cache: Miss from cloudfront
x-amzn-Remapped-Content-Length: 30
x-amzn-RequestId: 7ba89dcb-ffb9-11e7-a090-31d329e3f7fd

Andrew, Hello World from Zappa
```

```
(env) [/tmp/zappa_test] $ http --form POST https://1uybd9nwpc.execute-api.us-west-1.amazonaws.com/dev/saygreeting name=Andrew
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 32
Content-Type: text/html; charset=utf-8
Date: Mon, 22 Jan 2018 21:16:31 GMT
Via: 1.1 ce270f4a88edde7438864bc44406e83a.cloudfront.net (CloudFront)
X-Amz-Cf-Id: j1MDNcb0RuT4W7kBEaLlnEGBwVXDnSb2nq8ejTpBALENHp9gYaz_iA==
X-Amzn-Trace-Id: sampled=0;root=1-5a6654af-db2ed211e110fecc0086e760
X-Cache: Miss from cloudfront
x-amzn-Remapped-Content-Length: 32
x-amzn-RequestId: 84461f36-ffb9-11e7-93f0-b7b607a3b6ac
```
Data has been saved successfully
That's it. It is so simple.

####Step 4: Now lets kill it. In most cases, I like to kill the deployment after I get all the feedback I need. Here is how it is done.

```
(env) [/tmp/zappa_test] $ zappa undeploy
Calling undeploy for stage dev..
Are you sure you want to undeploy? [y/n] y
Deleting API Gateway..
Waiting for stack zappa-test-dev to be deleted..
Unscheduling..
Unscheduled zappa-test-dev-zappa-keep-warm-handler.keep_warm_callback.
Deleting Lambda function..
Done!
```
Thats it. I longer haveto worry about the API anymore.

####Step 5 (Bonus): Based on the feedback I receive about the API, I usually update the API code in mytestapp.py and re-deploy the API as follows

```
(env) [/tmp/zappa_test] $ zappa update dev
```

Thank you for dropping by.