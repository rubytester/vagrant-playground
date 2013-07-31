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



