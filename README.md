Play framework 2 java application on OpenShift Express
============================

This git repository helps you get up and running quickly with a Play framework 2 (with Java) application
on OpenShift Express.


Running on OpenShift
----------------------------

Register at http://openshift.redhat.com/, and then create a raw (do-it-yourself) application:

    rhc app create -a play2java -t diy-0.1 -l yourlogin@yourmail.com

Add this upstream play-example repo:

    cd play2java
    git remote add upstream -m master https://github.com/opensas/play2-java-example.git
    git pull -s recursive -X theirs upstream master

Now you should update your appName setting in project/Build.scala to match your application name.

Then push the repo upstream:

    git push

That's it, you can now see your application running at:

    http://play2java-yournamespace.rhcloud.com


Working with a mysql database
----------------------------

Just issue:

    rhc app cartridge add -a play2java -c mysql-5.1

Don't forget to write down the credentials.

Then uncomment the following lines from your conf/application.conf, like this:

    # openshift mysql database
    %openshift.db.url=jdbc:mysql://${OPENSHIFT_DB_HOST}:${OPENSHIFT_DB_PORT}/${OPENSHIFT_APP_NAME}
    %openshift.db.driver=com.mysql.jdbc.Driver
    %openshift.db.user=admin
    %openshift.db.pass=<write your password here>

You can manage your new MySQL database by embedding phpmyadmin-3.4.

    rhc app cartridge add -a play2java -c phpmyadmin-3.4

It's also a good idea to create a different user with limited privileges on the database.

Updating your application
----------------------------

To deploy your changes to openshift just add your changes to the index, commit and push:

    git add . -A
    git commit -m "a nice message"
    git push origin


Trouble shooting
----------------------------

To find out what's going on in openshift, issue

    rhc app tail -a play2java

If you feel like investigating further, you can

    rhc app show -a play2java

    Application Info
    ================
    play
        Framework: raw-0.1
        Creation: 2012-03-18T12:39:18-04:00
        UUID: youruuid
        Git URL: ssh://youruuid@play-yournamespace.rhcloud.com/~/git/raw.git/
        Public URL: http://play-yournamespace.rhcloud.com

Then you can connect using ssh like this:

    ssh youruuid@play-yournamespace.rhcloud.com

Having a look under the hood
----------------------------

**.openshift/action_hooks/pre_build** does the following everytime you push changes

* reads play version from _openshift.play.version_ at conf/openshift.conf (2.0 by default)

* checks if the desired version is installed, if not it downloads and installs play framework at $OPENSHIFT_DATA_DIR

* removes any other play framework version

then **.openshift/action_hooks/start** goes like this

* it executes **.openshift/action_hooks/stop** to stop the application

* cleans environment, compile sources and builds the stage

```bash
    play clean compile stage
```

* finally it starts the application, using _conf/openshift.conf_. You can specify additional parameters with openshift.play.params.

```bash
    target/start $PLAY_PARAMS 
        -Dhttp.port=${OPENSHIFT_INTERNAL_PORT}
        -Dhttp.address=${OPENSHIFT_INTERNAL_IP}
        -Dconfig.resource=openshift.conf
```

Play will then run your app in production mode. The server will listen to ${OPENSHIFT_INTERNAL_PORT} at ${OPENSHIFT_INTERNAL_IP}.

* **.openshift/action_hooks/stop** just tries to kill the RUNNING_PID process, and then checks that no "java" process is running. If it's there, it tries five times to kill it nicely, and then if tries another five times to kill it with -SIGKILL.

Acknowledgments
----------------------------

I couldn't have developed this quickstart without the help of [marekjelen](https://github.com/marekjelen) who answered [my questions on stackoverflow](http://stackoverflow.com/questions/9446275/best-approach-to-integrate-netty-with-openshift) and who also shared his [JRuby quickstart repo](https://github.com/marekjelen/openshift-jruby#readme). (I know, open source rocks!)

It was also of great help Grant Shipley's [article on building a quickstart for openshift](https://www.redhat.com/openshift/community/blogs/how-to-create-an-openshift-github-quick-start-project).

Play framework native support for openshift was a long awaited and pretty popular feature (you are still on time to vote for it [here](https://www.redhat.com/openshift/community/content/native-support-for-play-framework-application)). So it's a great thing that Red Hat engineers came out with this simple and powerful solution, that basically let's you implement any server able to run on a linux box. Kudos to them!!!

Licence
----------------------------
This project is distributed under [Apache 2 licence](http://www.apache.org/licenses/LICENSE-2.0.html). 