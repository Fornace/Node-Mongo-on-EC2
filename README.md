## First step: Installing node.js and npm ##
To compile node we need gcc, make and git to import node source code:

    sudo yum install gcc-c++ make
    sudo yum install openssl-devel
    sudo yum install git

Cloning node.js source code:

    git clone git://github.com/joyent/node.git

Compile and install node.js

    cd node
    ./configure 
    make
    sudo make install

Add user´s directory to BIN Paths (node binaries location)

    sudo su
    nano /etc/sudoers

inside the editor scroll to where you see the line: `Defaults secure_path = /sbin:/bin:/usr/sbin:/usr/bin`

Append the value `:/usr/local/bin`

### Installing npm (Node Package Manager) ###

    git clone https://github.com/isaacs/npm.git
    cd npm
    sudo make install

Test if node is working:

    node

> this must show a `>` indicator  
> press Ctrl+C two times to close node interpreter

### Installing nginx ###

    yum install nginx
    nano /etc/nginx/nginx.conf

Add the proxy under `location / {`

    proxy_pass http://127.0.0.1:3000/;

Test if it's working

    service nginx restart
    mkdir /var/www
    nano /var/www/app.js
    
Paste this code:

    var http = require('http');
    http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World\n'); }).listen(3000, "127.0.0.1");
    console.log('Server running at http://127.0.0.1:3000/');

### Ensuring continuos run ###

    npm install forever -g

Then run the node app:

    forever start /var/www/app.js
    
To stop you have to see the id (it's at the beginning of the line):
    forever list
    forever stop 0
    
Or regular mode (will interrupt when closing cli or rebooting)

    node /var/www/app.js


## Second step: Install and configure MongoDB database ##
> Based on [http://docs.mongodb.org/ecosystem/tutorial/install-mongodb-on-amazon-ec2/](http://docs.mongodb.org/ecosystem/tutorial/install-mongodb-on-amazon-ec2/ "Installing mongodb on Amazon EC2 tutorial")

First add an entry to the local **yum** repository for MondoDB

    echo "[10gen]
    name=10gen Repository
    baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64
    gpgcheck=0" | sudo tee -a /etc/yum.repos.d/10gen.repo

You must respect jump of lines in the previous code

Next, install **MongoDB** and the **sysstat** diagnostic tools:

    sudo yum -y install mongo-10gen-server mongodb-org-shell
    sudo yum -y install sysstat

We are using `/var/lib/mongo` folder to save database, log and journal data, you can another folder path.

    sudo mkdir /var/lib/mongo/data
    sudo mkdir /var/lib/mongo/log
    sudo mkdir /var/lib/mongo/journal

Set the storage items (data, log, journal) to be owned by the user (mongod) and group (mongod) that MongoDB will be starting under:

    sudo chown mongod:mongod /var/lib/mongo/data
    sudo chown mongod:mongod /var/lib/mongo/log
    sudo chown mongod:mongod /var/lib/mongo/journal

### Starting and testing MongoDB ###

Set the MongoDB service to start at boot and activate it:

    sudo chkconfig mongod on
    sudo /etc/init.d/mongod start

When starting for the first time, it will take a couple of minutes for MongoDB to start, setup it’s storage and become available. Once it is, you should be able to connect to it from within your instance:

    $ mongo
    MongoDB shell version: 2.4.3
    connecting to: test
    >

> For more information about how to configure storage setting for MongoDB on Amazon EC2, checkout [http://docs.mongodb.org/ecosystem/tutorial/install-mongodb-on-amazon-ec2/](http://docs.mongodb.org/ecosystem/tutorial/install-mongodb-on-amazon-ec2/ "Install MongoDB on Amazon EC2")
