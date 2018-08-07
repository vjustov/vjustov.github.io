---
layout: post
title: "The quest for a sane ruby environment on windows x64"
description: "There is no one who loves pain itself, who seeks after it and wants to have it, simply because it is pain..."
comments: true
keywords: "dummy content, lorem ipsum"
---

You tried ruby in a class and are starting to think about installing it on your laptop and using it for a forthcoming proyect. You google a bit and notice that most of the ruby comunity runs on macs and ubuntu's, some of them VMs, for some particular reason you dont want to run a full Virtual machine to code in and your workflow in windows is smooth enough to give it a try (or perhaps you are a windows fanboy).

You decide to pave the road ahead and want to install all you'll ever need for your new journey with ruby on windows.

In building my actual development environment for my x64 i have come across my fair share of roadblocks and issues that would have been easily avoided had I known all this. My goal is for this environment to be easy to use, easy to adopt and flexible enough to at the least \"try\" to catch up onto ruby for other platforms. Needless to say, ruby on windows is kinda like IE in the browser wars.

###TL;DR
In order to have an awesome webdev env:

* Install ruby/rails.
* Install pik to manage rubies -v. Use a path with no spaces.
* Make sure to install devkit for each ruby installed through pik.
* x64 rubies cannot be installed through pik.
* Use Vagrant for sharing your development env with other devs.

-----------------------
Before we start with anything, i must say that i encourage this setup if you are planning on toying with ruby on the long run. if you only wanna try rails to see if it fits your needs then you can make do with just the rails installer. It is strongly recomended that your user does not contain any spaces in its username. For those of you still using XP you have two options: one is setting a `%PIK_HOME%` to a folder without spaces, that folder will become the place were pik will download and install all gems and rubiesand the other one is to create a symbolic link to the `C:/document and settings/{{USER}}` folder and use that to \"mask\" the spaces.
**UPDATE:** If you ever plan on debugging gems (and you eventually will.). you must set a `%BUNDLE_EDITOR%` to the path of your editor of choice in order to use `>bundle open`.

If you are like me, you probably went the \"[RubyInstaller](http://rubyinstaller.org/) (or [RailsInstaller](http://rubyinstaller.org/) for that matter.) way\", which works marvelously and i wouldnt recommend any other way. Luis Lavena and the crew at OneClick have done a marvelous job into making ruby to run as smoothly as posible with Win32 (But this is another topic.).

![](/content/images/2014/Jan/railsinstaller_PNG-1.png)

With these installers you only need to follow the wizard and by the end of it, you will have ruby (and rails, if you used the RailsInstaller), git, DevKit, Bundler, RubyGems. Its fair to point out that RailsInstaller does not have a stable release for ruby 2.0 yet.

Later on, you will start working on a proyect that requires a diferent `ruby -v` than the one you currently have installed. For that us, windows folk use [Pik](https://github.com/vertiginous/pik).  Pik is a Ruby Version Manager on Windows. It still misses some of the features of its *nix counterpart RVM, like gemsets. While in RVM you can change your active gems by using a gemset. Pik has a fixed set of gems for each project sharing that ruby.
**UPDATE:** Pik development have been halted. Luis Lavena now encourages the use of [Uru](https://bitbucket.org/jonforums/uru), an cross-platform version manager.Which in contrast, does support gemsets. (I haven't used it but will give it a try soon enough)

There are two ways of installing pik; an installer and using the gem. as you may imagine we will walk down the gem path in order to familiarize the uninitiated (if any).

    >gem install pik
    Fetching: pik-0.2.8.gem
    ...
    Successfully installed pik-0.3.0
    1 gem installed

After the gem is installed, you can use the ‘pik_install’ script to install the pik executable. Install pik to a location that’s in your path, but someplace other than your `/ruby/bin`. For instance, the directory `C:/bin` is in my path:

    >path
    PATH=c:\\pik;C:\\Program Files\\Windows Resource Kits\\Tools\\;c:\\ruby\\Ruby-186-p383\\bin;`

So I run:

    >pik_install C:\\pik
    Thank you for using pik.
    Installing to C:\\bin
    ...
    pik is installed

Now you can install your x86 rubies through pik by doing `pik install ruby 2.0` or `pik install jruby` (also IronRuby). For each of these installed rubies you will have to enhanece its actual and future gems with DevKit by downloading the corresponding devkit -v and afterwards you will need to cd into the DevKit directory (which in my case is the one in the folder created by rails installer.). once there you will use the newly installed version of your ruby.

    >cd C:/RailsInstaller/Devkit1.9.3
    >pik use {{FRESH_RUBY_INSTALLATION_ALIAS}}
    >ruby dk.rb init
    >ruby dk.rb install

Devkit's `init` flag will create a config file with all the path to the rubies that this particular devkit will enhance. After validation the `install` will proceed with the installation. If you already installed it and something went wrong you might need to add the `-f` flag to force installation.

With this you have a nice environment ready, but another good addition to the bundle is vagrant. Vagrant is a tool for automating development environments so that each and everyone of the members of a team can work with the same tools. It removes the \"it works on my machine\" variable from the equation.

>Instead of building a virtual machine from scratch, which would be a slow and tedious process, Vagrant uses a base image to quickly clone a virtual machine. These base images are known as boxes in Vagrant, and specifying the box to use for your Vagrant environment is always the first step after creating a new Vagrantfile.

First we'll need to install Vagrant and virtual box (because it's my preferred flavor, you can use VMware, AWS, or [many other providers](http://docs.vagrantup.com/v2/providers/)). After you have run both wizards, you must fetch a box(or boxes) and add it to your \"warehouse\" (get it? boxes are mostly on warehouses, HAHAHAH... ok, i'll shut up).

    $ vagrant box add base http://files.vagrantup.com/precise32.box
    $ vagrant init
    $ vagrant up


The first command adds a box at the given url and stores it under the name `base`. The second command initializes the current directory to be a vagrant environment and creates a vagrantfile, which you can later edit to fine tune the configuration of your virtual environment. Vagrant up then proceeds to create and configure the guest machine according to the specification in your vagrant file.

>The primary function of the Vagrantfile is to describe the type of machine required for a project, and how to configure and provision these machines. Vagrantfiles are called Vagrantfiles because the actual literal filename for the file is Vagrantfile (casing doesn't matter). Vagrantfile is supposed to be committed to version control. This allows other developers involved in the project to check out the code, run vagrant up, and be on their way.

After running the above three commands, you'll have a fully running virtual machine in VirtualBox running Ubuntu 12.04 LTS 32-bit (Thats the precise32.box). You can SSH into this machine with `vagrant ssh`, and when you're done playing around, you can remove all traces of it with `vagrant destroy`.

###In Summary

Your development environment is the place were you spend most of the time working, tweeking and throwing commands. Put the effort in to ensure it's not just functional but pleasant to use.
