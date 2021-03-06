Agora Elections
===============

Installation
=========

Installing java 7 in debian

    http://www.webupd8.org/2012/06/how-to-install-oracle-java-7-in-debian.html

Download the play activator from

    https://www.playframework.com/download

Once you have installed, clone the repository

    git clone https://github.com/agoravoting/agora_elections.git

Database setup

    postgres=# create user agora_elections;

    postgres=# create database agora_elections owner agora_elections;

    postgres=# create database agora_elections_test owner agora_elections;

    postgres=# ALTER ROLE agora_elections WITH PASSWORD 'mypassword';

Example configuration files

application.local.conf

    db.default.url="jdbc:postgresql://localhost:5432/agora_elections"
    db.default.driver="org.postgresql.Driver"
    db.default.user=agora_elections
    db.default.pass=agora_elections

    app.datastore.public="/tmp/agora_elections/datastore/public"
    app.datastore.private="/tmp/agora_elections/datastore/private"

    app.api.root="http://vota.podemos.info:8000"
    app.datastore.root="http://94.23.34.20:8000"

    app.api.max_revotes=5

    app.eopeers.dir=/etc/eopeers/

    app.partial-tallies=false

    booth.auth.secret=hohoho
    booth.auth.expiry=600000

    ws.ssl {
      loose.disableHostnameVerification=true
      keyManager = {
        stores = [
          { type = "JKS", path = "/tmp/agora_elections/keystore.jks", password = "password" }
        ]
      }
      trustManager = {
        stores = [
          { type = "JKS", path = "/tmp/agora_elections/keystore.jks", password = "password" }
        ]
      }
    }

    app.authorities {
      test-auth1.agoravoting.com = {
        name = "Agora Voting auth 1"
        description = "My authority 1"
        url = "http://foo.bar.1"
        image = "http://foo.bar.1/img.img"
      }
      "test-auth2.agoravoting.com" = {
        name = "Agora Voting auth 2"
        description = "My authority 2"
        url = "http://foo.bar.2"
        image = "http://foo.bar.2/img"
      }
    }


test.local.conf

    db.default.url="jdbc:postgresql://localhost:5432/agora_elections_test"
    db.default.driver="org.postgresql.Driver"
    db.default.user=agora_elections
    db.default.pass=agora_elections

    app.datastore.public="/tmp/agora_elections/datastore/public"
    app.datastore.private="/tmp/agora_elections/datastore/private"

    app.api.root="http://vota.podemos.info:8000"
    app.datastore.root="http://94.23.34.20:8000"

    app.eopeers.dir=./test

    app.partial-tallies=false

    booth.auth.secret=hohoho
    booth.auth.expiry=600000

    ws.ssl {
      loose.disableHostnameVerification=true
      keyManager = {
        stores = [
          { type = "JKS", path = "/tmp/agora_elections/keystore.jks", password = "password" }
        ]
      }
      trustManager = {
        stores = [
          { type = "JKS", path = "/tmp/agora_elections/keystore.jks", password = "password" }
        ]
      }
    }

    app.authorities {
      test-auth1.agoravoting.com = {
        name = "Agora Voting auth 1"
        description = "My authority 1"
        url = "http://foo.bar.1"
        image = "http://foo.bar.1/img.img"
      }
      "test-auth2.agoravoting.com" = {
        name = "Agora Voting auth 2"
        description = "My authority 2"
        url = "http://foo.bar.2"
        image = "http://foo.bar.2/img"
      }
    }

    logger.scala.slick.jdbc.JdbcBackend.statement=DEBUG


Key store set up

create the client keystore with the client certificate

    openssl pkcs12 -export -in /srv/certs/selfsigned/cert.pem -inkey /srv/certs/selfsigned/key-nopass.pem -out certs.p12 -name client

    keytool -importkeystore -deststorepass password -destkeypass password -destkeystore keystore.jks -srckeystore certs.p12 -srcstoretype PKCS12 -srcstorepass password -alias client

add the director ca to the keystore

    keytool -import -file auth1.pem -keystore keystore.jks

Admin tool set up

specify the follwing configuration parameters in the admin tool, at <app_root>/admin/admin.py

    # set configuration parameters
    datastore = '/tmp/agora_elections/datastore'
    shared_secret = 'hohoho'
    db_user = 'agora_elections'
    db_password = 'agora_elections'
    db_name = 'agora_elections'
    app_host = 'localhost'
    app_port = 9000
    node = '/usr/local/bin/node'

set the executable permissions if not already set for several admin scripts

    chmod u+x admin.py
    chmod u+x cycle.py
    chmod u+x batch.py
    chmod u+x encrypt.sh
    chmod u+x results.sh

Create a virtualenv for the admin script and install admin script requirements:

    mkvirtualenv agora_elections -p $(which python2)
    workon agora_elections
    cd agora_elections/admin
    pip install -r requirements.txt

Agora-Results set up

Clone and install agora-results

    git clone https://github.com/agoravoting/agora-results.git
    mkvirtualenv agora-results -p $(which python3)
    workon agora-results
    cd agora-results
    pip install -r requirements.txt

you must also configure these two settings in results.sh, found in agora-elections/admin

    AGORA_RESULTS=/tmp/agora-results
    VENV=/root/.virtualenvs

Local vote encryption set up

If you want to encrypt votes locally (for testing purposes, see below), you must configure these
settings in admin/encrypt.sh

    IVY=/root/.ivy2/cache/

you must also make sure that the software is packaged from the activator console, with

    [agora-elections] $ package

which will generate the required jar in the target directory (note that encrypt.sh assumes scala v2.11)

Postgresql parameters

Some tuning changes can be made in /etc/postgresql/<version>/main/postgresql.conf

    max_connections = 200
    checkpoint_segments = 16
    checkpoint_completion_target = 0.7

restart the server

    service restart postgresql

Unit tests
=====

    activator
    test

Administration
==============

An election cycle can be run with the admin.py tool in the admin directory

You must first create an election config json file. Here's an example

    {
    "id": 66,
    "title": "My vote",
    "description": "choose stuff",
    "director": "my_director",
    "authorities": ["my_authority"],
    "layout": "pcandidates-election",
    "presentation": {
      "share_text": "share this",
      "theme": "foo",
      "urls": [
        {
          "title": "",
          "url": ""
        }
      ],
      "theme_css": "whatever"
    },
    "end_date": "2013-12-09T18:17:14.457000",
    "start_date": "2013-12-06T18:17:14.457000",
    "questions": [
        {
            "description": "",
            "layout": "pcandidates-election",
            "max": 1,
            "min": 0,
            "num_winners": 1,
            "title": "My question",
            "randomize_answer_order": true,
            "tally_type": "plurality-at-large",
            "answer_total_votes_percentage": "over-total-valid-votes",
            "answers": [
                {
                    "id": 0,
                    "category": "Cat A",
                    "details": "",
                    "sort_order": 1,
                    "urls": [
                      {
                        "title": "",
                        "url": ""
                      }
                    ],
                    "text": "Option A"
                },
                   {
                    "id": 1,
                    "category": "Cat B",
                    "details": "",
                    "sort_order": 1,
                    "urls": [
                      {
                        "title": "",
                        "url": ""
                      }
                    ],
                    "text": "Option B"
                }

            ]
        }
    ]
    }


Register the election (config must be named 50.json here)

     ./admin.py register 50

create the election

    ./admin.py create 50

dump the pks

    ./admin.py dump_pks 50

encrypt votes (normally only used for testing; you need an votes.json file to do this)

    ./admin.py encrypt 50

start the election

    ./admin.py start 50

cast votes (normally only used for testing, votes are generated with encrypt)

    ./admin cast_votes 50

tally the election

    ./admin.py tally 50

calculate results

    ./admin results 50 --results-config config.json

publish results

    ./admin.py publish_results 50

You can also carry out administration commands in batch mode, with batch.py. batch.py expects a directory
with configuration files for both election definition and agora-results set up. The files must be in this
format

    <id>.results.json

for election configuration files, and

    <id>.config.results.json

for agora-results configuration files. batch.py will run the command for all the configuration files
found in the given directory, in increasing order of id. For example

    ./batch.py -c create -d json_files

will register, create and start elections for all configuration files in directory 'json_files'. If you
only want to operate on a range, you can use this form

    ./batch.py -c create -d json_files -s 1000 -e 2000

which will create elections with id beginning with 1000 up to 2000 (both inclusive)

Automated testing of an election cycle
==============

You can do automated tests with the cycle.py tool in the admin directory. This requires
correctly setting up admin.py first, as described earlier. You need an election configuration as base,
you can use one like the one above. cycle.py looks for such a file with default name 'election.json'.
You also need an agora-results configuration file to calculate results. cycle.py looks for such a file
at 'config.json'. Here's an example agora-results configuration file

    [
      [
        "agora_results.pipes.results.do_tallies",
        {"ignore_invalid_votes": true}
      ],
      [
        "agora_results.pipes.results.to_files",
        {"paths": ["results.json"]}
      ]
    ]

To run 5 cycles serially, starting with id 50

    ./cycle.py -t 5 -i 50

to run 5 cycles serially, casting 100 votes in each (votes duplicated)

    ./cycle.py -t 5 -i 50 -e 100

to run 10 cycles in parallel, using election.json as election config, config.json as agora-results config,
casting 100 votes and starting at id 50:

    ./cycle.py -t 10 -i 50 -e 100 -c election.json -r config.json -p

Running (dev mode)
======

    activator
    run

Running (production mode)
======

    activator
    clean
    stage
    exit
    target/universal/stage/bin/agora-elections -v

you probably want to pass in a configuraiton file

    target/universal/stage/bin/agora-elections -Dconfig.file=/full/path/to/conf/application-prod.conf

the log will be found in

     target/universal/stage/logs/application.log
