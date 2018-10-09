# Setup Concourse on Mac
# Prerequisites

- [Homebrew](http://brew.sh/)

# Installs

## Concourse

    curl -Lo concourse https://github.com/concourse/concourse/releases/download/v2.5.0/concourse_darwin_amd64 && chmod +x concourse && mv concourse /usr/local/bin
    
## Fly

    curl -Lo fly https://github.com/concourse/concourse/releases/download/v2.5.0/fly_darwin_amd64 && chmod +x fly && mv fly /usr/local/bin/

## Postgres

    brew install postgres
    
## Check if everything is OK

    concourse --version
    fly --version

    # Server version
    pg_config --version 
    
    # Client version
    psql --version

# Setup

## Init the db

    initdb /usr/local/var/postgres
    
## Start the PostgreSQL server

    pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
    
### Optional: launch Postgres automatically at login

    mkdir -p ~/Library/LaunchAgents
    cp /usr/local/Cellar/postgresql/9.5.4_1/homebrew.mxcl.postgresql.plist ~/Library/LaunchAgents/
    launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
    
# Config    
    
## Setup the necessary users, roles and dbs

    createdb atc;
    createdb concourse;
    createuser concourse --pwprompt; # You will be prompted for a password.
    
## Generate the necessary keys for Concourse

Do this in an empty folder somewhere, you'll need to launch Concourse from this location. The commands are copied from https://concourse.ci/binaries.html

    ssh-keygen -t rsa -f host_key -N '' && ssh-keygen -t rsa -f worker_key -N '' && ssh-keygen -t rsa -f session_signing_key -N ''
    cp worker_key.pub authorized_worker_keys

# Start

## Web overview

    concourse web \
      --basic-auth-username concourse \
      --basic-auth-password the_password_you_gave_earlier \
      --session-signing-key session_signing_key \
      --tsa-host-key host_key \
      --tsa-authorized-keys authorized_worker_keys
      
If all went well, you should see the web interface @ http://localhost:8080

# Plumb those pipelines

Download hello.yml, then run the following:

    fly -t lite login -c http://127.0.0.1:8080

You will be prompted for username and password, concourse and the_password_you_gave_earlier.

Add the hello-world pipeline:

    fly -t lite set-pipeline -p hello-world -c hello.yml
    
By default, the pipeline will be paused, unpause it:

    fly unpause-pipeline -p hello-world -t lite
    
This can also be done in the web UI, but you know, meh :p.

# Now the final step, work it!

Spin up a worker:

    sudo concourse worker \
      --work-dir /opt/concourse/worker \
      --tsa-host 127.0.0.1 \
      --tsa-public-key host_key.pub \
      --tsa-worker-private-key worker_key
    
That's it! Check the result at http://127.0.0.1:8080/teams/main/pipelines/hello-world/jobs/hello-world

While the job has been setup, it won't run automatically, click the plus icon to execute it. For further usage (auto triggering jobs), check https://concourse.ci/hello-world.html at the bottom.
