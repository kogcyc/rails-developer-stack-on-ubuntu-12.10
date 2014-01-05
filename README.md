A Full Rails Development Stack on Ubuntu
=========

Since Rails runs on Linux, why not develop it on Linux using open source tools?

Here's how to install Rails from scratch on Ubuntu 12.10.

The first portion installs Ruby, Git, Rails and databases.

The second portion installs Sublime Text, GIMP, Inkscape and other superb open source tools.

Portion One: Ruby, Git, Rails and databases
-

####1) We start by updating the system and then installing the libraries we will need:

    $ sudo apt-get update
    $ sudo apt-get upgrade
    $ sudo apt-get install build-essential curl git-core libpq-dev libreadline-dev libsqlite3-dev libssl-dev libxml2-dev libxslt-dev zlib1g-dev

####2) But we have to build node.js from source...

    $ git clone https://github.com/joyent/node.git
    $ cd node
    $ ./configure && make && sudo make install

####3) Next we config Git

    $ git config --global user.name "John Doe"
    $ git config --global user.email jdoe@gmail.com
    $ git config --list

####4) Then we install rbenv

The trick with rbenv is to run: rbenv rehash every time you change something, like install a new version of Ruby or a new gem.

    $ git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
    $ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
    $ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
    $ exec $SHELL
    
    $ mkdir ~/.rbenv/plugins
    $ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
    $ exec $SHELL
    $ rbenv install 1.9.3-p448
    
    $ rbenv rehash
    $ rbenv global 1.9.3-p448

####5) Time to install Rails
The first step I take is to turn off ri and rdoc to speed things up.

    $ echo "install: --no-ri --no-rdoc" >> ~/.gemrc
    $ echo "update: --no-ri --no-rdoc" >> ~/.gemrc

    $ gem install rails -v 3.2.14
    $ rbenv rehash

####6) sqlite3

The server portion of sqlite3 is a set of libraries that binaries (and gems) can link to and use.

Rails installs those sqlite3 libraries and the sqlite3 gem.

    $ irb
        irb(main):001:0> require 'sqlite3'
    	=> true
    	irb(main):002:0> db = SQLite3::Database.new ":memory:"
    	=> #<SQLite3::Database:0x000000014467a8>
    	irb(main):003:0> puts db.get_first_value 'SELECT SQLITE_VERSION()'
    	3.7.13
    	=> nil

If you want to use a command line interface to sqlite3, you can install the whole package:

    $ sudo apt-get install sqlite3

After that you can run the cli which is also named: sqlite3

	$sqlite3
	SQLite version 3.7.13 2012-06-11 02:05:22
	Enter ".help" for instructions
	Enter SQL statements terminated with a ";"
	sqlite> select sqlite_version();
	3.7.13

####7) RMagick

I use the RMagick gem to do image processing.

    $ sudo apt-get install libmagickwand-dev
    $ gem install rmagick

Here's a test to see if it is installed OK, uncomment the -display- method is you want to see the Red Square:

    $ irb
        irb(main):001:0> require 'RMagick'
        => true
        irb(main):002:0> include Magick
        => Object
        irb(main):003:0> # Create a 100x100 red image.
        irb(main):004:0* f = Image.new(100,100) { self.background_color = "red" }
        =>   100x100 DirectClass 16-bit
        irb(main):005:0> # f.display
        irb(main):006:0* exit

####8) Postgresql

    $ sudo apt-get install postgresql
    $ gem install pg

Here's how to use the command line interface:

    $ sudo su postgres
    $ psql -U postgres
    
    	DROP USER john;
    	CREATE USER john WITH PASSWORD 'password';
    
    	DROP DATABASE exampledev;
    	CREATE DATABASE exampledev;
    	GRANT ALL PRIVILEGES ON DATABASE exampledev to john;
    
    	DROP DATABASE exampletest;
    	CREATE DATABASE exampletest;
    	GRANT ALL PRIVILEGES ON DATABASE exampletest to john;
    
    	DROP DATABASE exampleprod;
    	CREATE DATABASE exampleprod;
    	GRANT ALL PRIVILEGES ON DATABASE exampleprod to john;
    
    	\q

DON'T FORGET TO DO AN exit TO GO BACK TO YOUR LOGIN!

And here's a Ruby test:

    $ irb
    
    	irb(main):001:0> require 'pg'
    	=> true
    	irb(main):002:0> n = PGconn.connect("localhost",5432,"","","exampledev","john","password")
    	=> #<PG::Connection:0x00000000cf9950>
    	irb(main):003:0> n.exec('CREATE TABLE posts (title string, content text);')
    	=> #<PG::Result:0x00000000d08518>
    	irb(main):004:0> n.exec("INSERT INTO posts VALUES ('Item 1','Contents here!');")
    	=> #<PG::Result:0x00000000ec0400>
    	irb(main):005:0> res = n.exec("SELECT * FROM posts")
    	=> #<PG::Result:0x00000000e21328>
    	irb(main):006:0> res.count
    	=> 1
    	irb(main):007:0> res.each { |row| puts row.values_at('title', 'content') } 
    	Item 1
    	Contents here!
    	=> #<PG::Result:0x00000000e21328>
    	irb(main):008:0> res = n.exec("SELECT datname FROM pg_database;")
    	=> #<PG::Result:0x00000000f7e770>
    	irb(main):009:0> res.each { |row| puts row.values_at('datname') } 
    	template1
    	template0
    	postgres
    	exampledev
    	exampletest
    	exampleprod
    	=> #<PG::Result:0x00000000f7e770>
    	irb(main):010:0> res = n.exec("SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';")
    	=> #<PG::Result:0x0000000108c860>
    	irb(main):011:0> res.each { |row| puts row.values_at('table_name') } 
    	posts
    	=> #<PG::Result:0x0000000108c860>
        
####9) GitHub and Heroku

First, create an SSH key:

    $ ssh-keygen -t rsa -C "jdoe@gmail.com"

Then, cut and paste it into a key at Github:

    $ cd .ssh
    $ cat id_rsa.pub 
    GO TO GITHUB AND ADD THAT KEY 

And then run a test to see if it worked:

    $ ssh -T git@github.com

Next install the Heroku Toolbelt:

    $ wget -qO- https://toolbelt.heroku.com/install-ubuntu.sh | sh

And then test to see if it installed OK:

    $ heroku apps 

Portion Two: Sublime Text, GIMP, Inkscape and the rest
-

    sudo add-apt-repository ppa:webupd8team/sublime-text-2
    sudo apt-get update
    sudo apt-get install sublime-text
    
    sudo add-apt-repository ppa:otto-kesselgulasch/gimp
    sudo apt-get update
    sudo apt-get install gimp
    START GIMP AND DO Windows->Single-Window Mode
    
    sudo apt-get install inkscape
    sudo apt-get install nautilus-open-terminal
