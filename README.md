# A simple builder with concourse and gogs

# Running the builder

First run init.sh locally to get some pre-reqs for concourse set up.
```
$ ./init.sh
```

Now start things up (this will require a bit of time to download images the
first time)
```
$ docker-compose up -d
```

Set up fly by browsing to concourse (by default this will be localhost:8080)
```
$ open http://localhost:8080
```

You should see a download link when you log in (the username and password are
in docker-compose.yml). Make it executable for your platform if necessary then
run:
```
$ fly -t lite login -c http://localhost:8080
username: concourse
password: ********
```

Now you are set. To add the hello world and watch it work, just:
```
$ fly -t lite set-pipeline -p howdy -c hello.yml
```
Where -t is for target, -p is the name for your new pipeline, and -c is the
file to source for how to construct it.

# Integrating gogs

Concourse supports git, but it won't just automatically pull unless you set up
some webhooks.

To get gogs to POST a webhook, you unfortunately have to give gogs POST
permissions and raw credentials. So, you'll be making a webhook that looks
like this:
http://concourse:changeme@172.17.0.1:8080/api/v1/pipelines/howdy/jobs/hello-world/builds

So now you can `git push` to gogs, but the hook has all the credentials, so
you need to:
1. Make sure gogs only has permission to POST to a unprivileged job
2. Make sure the job only does a git pull from gogs again

If you do that, the only thing you loose is if someone takes that url, they
can trigger jobs as much as they want. It doesn't change the fact that if the
git pull detects no changes, no build will occur. It just could lead to denial
of service.

Still! I'm considering making a gogs plugin. It's super easy to write
concourse plugins. You just need three bash scripts... so... want to play
a game?
