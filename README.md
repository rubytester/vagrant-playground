## Vagrant playground
learning vagrant and chef provisioning

on linux mint 13 (precise)
has virtualbox 4.1.12 (using apt-get)
I should install install 4.2.16 which should work with vagrant 1.2.7

## software source add

add source `deb http://download.virtualbox.org/virtualbox/debian precise contrib`
and get the key `wget -q http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc -O- | sudo apt-key add -`
`sudo apt-get update` and `sudo apt-get install virtualbox-4.2`

that installs virutalbox 4.2.16 right now

## vagrant

now we install vagrant

http://vagrantup.com/

https://github.com/mitchellh/vagrant


## download

http://downloads.vagrantup.com/tags/v1.2.7

vagrant_1.2.7_x86_64.deb

vagrant installs itsef in `/opt/vagrant` and has embedded ruby in `/opt/vagrant/embedded/bin/ruby`

`ruby 1.9.3p448 (2013-06-27 revision 41675) [x86_64-linux]`

## getting started

let's understand what's happening here. `mkdir -p vagrant-playground ** cd vagrant-playground`

`vagrant init precise32 http://files.vagrantup.com/precise32.box`

and get this output

```
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```


# .vagrant.d

I got `~/.vagrant.d/` dir

```
$ lstree
   .
   |-boxes
   |-data
   |-gems
   |-rgloader
   |-tmp
```

And we got insecure_private_key as a file of interest. Otherwise nothing there yet. 

## vagrant up

got this message

```
Bringing machine 'default' up with 'virtualbox' provider...
[default] Box 'precise32' was not found. Fetching box from specified URL for
the provider 'virtualbox'. Note that if the URL does not have
a box for this provider, you should interrupt Vagrant now and add
the box yourself. Otherwise Vagrant will attempt to download the
full box prior to discovering this error.
Downloading or copying the box...
```

awesome! we are now downloading the box. and finally

```
Successfully added box 'precise32' with provider 'virtualbox'!
[default] Importing base box 'precise32'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] Booting VM...
[default] Waiting for VM to boot. This can take a few minutes.
[default] VM booted and ready for use!
[default] Mounting shared folders...
[default] -- /vagrant
```

## let's ssh

`vagrant ssh` I can how ssh into the machine

```
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic-pae i686)

 * Documentation:  https://help.ubuntu.com/
Welcome to your Vagrant-built virtual machine.
Last login: Fri Sep 14 06:22:31 2012 from 10.0.2.2
vagrant@precise32:~$ 
```

??? and it has postinstall.sh scritp. did it run? should I run it? 

the box is now found in `~/.vagrant.d/boxes/precise32/virtualbox` with 4 files

```
box-disk1.vmdk
box.ovf
metadata.json
Vagrantfile
```

The actual image vm built from this box is in `~/VirtualBox VMs` folder `vagrant_default_xxxxnumbersxxx`. so that's typical VBox stuff vmdk

## quick test with provisioning using :shell

`config.vm.provision :shell, :inline => "echo 'hello empire' > hello.txt"`

`vagrant reload` (hm, you will see warning "stdin: is not a tty" this is not a problem according to this https://github.com/mitchellh/vagrant/issues/1673)

`vagrant ssh` and you should see hello.txt 

`cat hello.txt` should produce 'hello empire' 

ok we are ready for chef

## CHEF

https://learnchef.opscode.com/quickstart/workstation-setup/

I am going to skip rbenv for now and on my machine just get basic chef running

## sudo gem install chef

and here are the dependencies.

```
$ sudo gem install chef
Fetching: mixlib-config-1.1.2.gem (100%)
Fetching: mixlib-cli-1.3.0.gem (100%)
Fetching: mixlib-log-1.6.0.gem (100%)
Fetching: mixlib-authentication-1.3.0.gem (100%)
Fetching: yajl-ruby-1.1.0.gem (100%)
Building native extensions.  This could take a while...
Fetching: ipaddress-0.8.0.gem (100%)
Fetching: ohai-6.18.0.gem (100%)
Fetching: net-ssh-2.6.8.gem (100%)
Fetching: net-ssh-gateway-1.2.0.gem (100%)
Fetching: net-ssh-multi-1.1.gem (100%)
Fetching: highline-1.6.19.gem (100%)
Fetching: erubis-2.7.0.gem (100%)
Fetching: chef-11.6.0.gem (100%)
```

gonna use chef_solo for provisioning (not chef server yet)


the executable seems to be chef-solo but we're not going to use it. we'll use Vagrant file instead.
```
$ chef-solo -h
Usage: /usr/local/bin/chef-solo (options)
    -f, --[no-]fork                  Fork client
        --[no-]color                 Use colored output, defaults to enabled
    -c, --config CONFIG              The configuration file to use
    -d, --daemonize                  Daemonize the process
        --force-formatter            Use formatter output instead of logger output
        --force-logger               Use logger output instead of formatter output
    -F, --format FORMATTER           output format to use
    -g, --group GROUP                Group to set privilege to
    -i, --interval SECONDS           Run chef-client periodically, in seconds
    -j JSON_ATTRIBS,                 Load attributes from a JSON file or URL
        --json-attributes
    -l, --log_level LEVEL            Set the log level (debug, info, warn, error, fatal)
    -L, --logfile LOGLOCATION        Set the log file location, defaults to STDOUT
    -N, --node-name NODE_NAME        The node name for this client
    -o RunlistItem,RunlistItem...,   Replace current run list with specified items
        --override-runlist
    -r, --recipe-url RECIPE_URL      Pull down a remote gzipped tarball of recipes and untar it to the cookbook cache.
    -s, --splay SECONDS              The splay time for running at intervals, in seconds
    -u, --user USER                  User to set privilege to
    -v, --version                    Show chef version
    -W, --why-run                    Enable whyrun mode
    -h, --help                       Show this message
```

## test chef_solo with 'git' recipe

modify Vagrantfile to add_recipe "git". and let's see what we need to do to make the recipe run.

there is a recipe here https://github.com/opscode-cookbooks/git

by convention create dir tree and puts snippets of code from two files recipes/default.rb and attributes/default.rb

https://github.com/opscode-cookbooks/git/blob/master/recipes/default.rb
https://github.com/opscode-cookbooks/git/blob/master/attributes/default.rb

since I am on linux mint all I need is `package 'git'` in recipe location for package attributes right??? (flying blind but experimentig)

now run `vagrant provision` and magic happens. 

```
[default] -- /tmp/vagrant-chef-1/chef-solo-1/cookbooks
[default] Running provisioner: chef_solo...
Generating chef JSON and uploading...
Running chef-solo...
stdin: is not a tty
[2013-07-31T16:27:21+00:00] INFO: *** Chef 10.14.2 ***
[2013-07-31T16:27:21+00:00] INFO: Setting the run_list to ["recipe[git]"] from JSON
[2013-07-31T16:27:21+00:00] INFO: Run List is [recipe[git]]
[2013-07-31T16:27:21+00:00] INFO: Run List expands to [git]
[2013-07-31T16:27:21+00:00] INFO: Starting Chef Run for precise32
[2013-07-31T16:27:21+00:00] INFO: Running start handlers
[2013-07-31T16:27:21+00:00] INFO: Start handlers complete.
[2013-07-31T16:27:35+00:00] INFO: Chef Run complete in 13.329304 seconds
[2013-07-31T16:27:35+00:00] INFO: Running report handlers
[2013-07-31T16:27:35+00:00] INFO: Report handlers complete
```


Let's test. `vagrant ssh` to the machine and run `git` command. yiipppi. we are golden.

## mysterious .vagrant folder

when I ran vagrant up I got this folder

```
[~/vagrant-playground/.vagrant (master)]
$ lstree
   .
   |-machines
   |---default
   |-----virtualbox
```

with id file outputting the machine id. `0ef31af7-1786-4e87-bd27-ef15c0aeaee8` so I guess this is how vagrant keeps track of machine instance?
I bet I want to add .vagrant folder to gitignore.


## shared folder /vagrant on vm

Vagrant automatically sets up shared folder on vm in `/vagrant` pointing to the root project dir on host. (dir where we started vagrant with Vagrantfile on host `~/vagrant-playground` in this case). nice. So this is how it has access to the cookbooks ?! hm. So host and vm has a shared folder so no need to copy stuff? need to figure this out.

## provision users accounts

this tutorial to learn to create user accounts on vm.
https://gist.github.com/dergachev/3866825

So to work through confusion here. I think I need cookbooks/users so I'll get the git repo. https://github.com/opscode-cookbooks/users.git
I should probably make it a submodule of this project but for now let's stick it there.
I will use 'users::sysadmins' recipe. I guess this will create sysadmins group.
The next thing I expect it is to create a 'tesuser' account belonging to sysadmins group so let's have a databags/users/testeuser.json with password 'password123' like so

```
{
 "groups": [
    "sysadmins"
  ],
  "id": "testuser",
  "password": "$1$sSlgTsb2$uk.Y5pPU9EwTVYeGrRVKd."
}
```
hashed password, generated via openssl passwd -1 "password123"
(readmore http://www.openssl.org/docs/apps/passwd.html)


and add Vagrantfile entry for users::sysadmins
and i just discovered `chef.log_level = :debug` option so I can see what's going on with provisioning.

let's run.

## first run

- testuser was not created. apparently I need https://github.com/edelight/chef-solo-search
- sysadmins group was created succesfully
- and I just discovered git installed but with older version using apt-get and not using any provided attributes. why?

## fix git recipe

I deleted attributes/default.rb and git installed just with the package 'git' command using apt-get so I am not sure what to do with attributes


## back to fixing users


http://stackoverflow.com/questions/9882074/how-do-i-create-user-account-by-chef-solo/9884655#9884655

but first make public cookbooks into submodules

git submodule add git://github.com/opscode-cookbooks/users.git cookbooks/users
git submodule add git://github.com/edelight/chef-solo-search.git cookbooks/chef-solo-search

Found an example here https://github.com/niku/vagrant_config_files/tree/minimum_set

## success run

- data_bags path needs to be specified in Vagrantfile. (contrast that with cookbooks as a sensible default).
- recipe is 'sysadmins' but group name is 'sysadmin' singular

debug shows
```
Processing user[testuser] action create (/tmp/vagrant-chef-1/chef-solo-1/cookbooks/users/providers/manage.rb line 85)
Executing useradd -p '$1$sSlgTsb2$uk.Y5pPU9EwTVYeGrRVKd.' -m -d '/home/testuser' testuser
```

so in conclusion for chef_solo vagrant you need the search and make sure the group is singular 'sysadmin'. (there has to be another recipe just to create users and not new groups)


## READING

https://news.ycombinator.com/item?id=5559856
http://taylorlapeyre.me/2013/04/better-workflow-with-vagrant/
http://www.jetbrains.com/ruby/webhelp/configuring-remote-interpreters-via-virtual-boxes.html

https://github.com/amaia/rails-starter-box

http://www.youtube.com/watch?v=5KQUhMM_99Y
short 4 min video on on how to Config RubyMine for remote ruby.

https://github.com/pivotal-sprout/sprout

http://tumblr.nrako.com/post/22320729770/vagrant-chef-librarian

## librarian-chef

well, get rid of cookbooks directory and ignore it.
sudo gem install librarian-chef so we can manage cookbooks for Vagrantfile

