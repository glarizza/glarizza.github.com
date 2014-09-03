---
layout: post
title: "Using Git for Mac Sysadmins"
date: 2012-01-24 15:25
comments: true
categories: ['git', 'luggage']
---


As a former Mac Sysadmin, I frequently felt like I had one toe in the world in which Linux Sysadmins frequently found themselves, but was surrounded by a culture of GUI-Clickery that either feared, scorned, or flat-out avoided tools that were command-line based (or, as many will point out, used the GUI (Graphic User Interface) tools because 'they worked' and that was all they needed).  This sucked because Linux Sysadmins have some tools that, as a current colleague would say, 'are kind-of awesome.'

One of those tools is Git, which is a Distributed Version Control System (or, DVCS).  Git, in a TOTAL nutshell, is a tool that allows you to create multiple 'what-if scenarios' with very little cost and the ability to save yourself from...well, yourself.

## Yes - You're Going to Use the Command Line

Like I mentioned before, the GUI is ingrained in the culture of the Mac.  It wasn't until OS X that Mac Sysadmins finally had a proper command-line (which I will refer to as the CLI, which is short for Command Line Interface.  If you're not familiar with that terminology, I'm speaking about the interface you get when you run /Applications/Utilities/Terminal.app), so we're a bit 'new' to the whole idea.  I can help you if you fear or avoid the CLI; there's always room for learning and I'll try to hold your hand as much as possible.  For those that downright refuse to use the CLI...well, I'm sorry?

## What IS Git?

Git is relatively new in the DVCS world (being initially released in 2005).  Developers have been using DVCS for MANY years with tools like CVS, Subversion, Git, Mercurial, and others.  For a team of developers who ALL need to make contributions to a SINGLE project or code base, it's vital that they have a way to modify a file without breaking something that someone else is doing.  That's what DVCS tools do - allow you to take something as large as a code base like OS X 10.7 (or something as small as a directory with a couple of files) and make changes without affecting everyone else who's working on the SAME code base (or files in the directory).

So why am I choosing to talk about Git instead of, say, Subversion?  That's easy: lightweight branching and merging.  Git allows you to easily create a 'topic branch' (or, using the metaphor above, a 'what-if scenario'), make changes, and see if your changes are favorable or if they downright stink. If your changes are desirable, you can then 'merge' your topic branch back in with your master branch (or, the original state the files were in when you created the topic branch in the first place).  If the changes don't work well, you can just delete the topic branch and 'no-harm, no-foul'; your original files remain unchanged and in the pristine condition that they existed when you first created the topic branch.  True, Subversion could do the same thing, but Git allows MANY people to do the same thing and then EASILY merge in EVERYONE'S changes without having to fight Subversion (just trust me on this if you've never used Subversion).

And why am I choosing to talk about Git instead of Mercurial?  Easy - Git is what I learned :)  As I understand it Mercurial functions similarly, and I truly don't have the experience with Mercurial to tell you the differences.  I'm talking about Git because Git is what I use and Git is what my company uses.

## So what does this have to do with me as a Mac Sysadmin?

Good question.  Mac Sysadmins are more frequently finding themselves having to use the CLI to tune their clients and servers.  Also, many Mac Sysadmins are finding that certain CLI tools are working better for their teams (in terms of simplicity, responsibility, and visibility).  As they dip their toes into this CLI world, they're finding that they want the same features they had in the GUI: the ability to do a 'save-as', to create 'versions', and to have the ability to have a rollback function similar to Time Machine.  Git gives you this through the CLI (or, yes, even through the GUI as I will mention later in the article).

If you're ready to take that step then follow me&hellip;

## A note on Git and Github

You might have heard of [Github](http://www.github.com).  Github is a free (they have a free tier and plenty of subscription tiers of service) service that allows you to 'push' your git repositories up to them for storage in 'the cloud'.  Github is basically centralized-storage for your git repositories and is NOT NECESSARY for you to use git as a tool.  In reality, you CAN use Github or setup something like Gitolite on a local server in your environment to get most of the benefits of Github without having to push your repositories to Github's servers.  

The bottom line: Github and git (the tool) are two different things.  Github didn't create git, and git doesn't need Github to work properly.  Keep that in mind as you read through.

## Installing Git

Git can be installed in a variety of ways.  First and foremost, you can download the latest package from [Git's Googlecode Page](http://code.google.com/p/git-osx-installer/).  You can also choose to install git through Macports or Homebrew as long as you have them installed (both of which require the Developer Tools...but so does The Luggage, the tool we'll be using later).  Simply download the package from [Git's Googlecode Page](http://code.google.com/p/git-osx-installer/), install it, and you should be good to go!

If you're using Macports, make sure the Developer Tools are installed, Macports is installed, and then do the following from the command line (**PLEASE NOTE: The dollar sign is a prompt - you SHOULDN'T type it.  Lines BEGINNING with a dollar sign, or a prompt, should be typed by you.  Lines that DON'T begin with a prompt represent the output you receive from typing the command.  If you don't see a line that DOESN'T begin with a prompt, then I'm not listing the output.  Got it?  Good!**):

```
    $ sudo port install git-core
```

If you're using Homebrew, make sure the Developer Tools are installed, Homebrew is installed, and then do the following:

```
    $ sudo brew install git
```

To check to see where and what version of git is installed, execute the following commands :

```
    $ which git
    $ git --version
```

You should receive the path to where git is installed after you run the first command, and you should receive the version of git that's currently installed on your system.  Any version of Git greater-than or equal to 1.7 is probably ideal, though most of the commands we'll be using will probably work just fine with previous versions.

## Your first Git repo

I find it's easier to understand Git if I describe an actual scenario instead of talking in terms of 'git can do...'.  I'm going to talk about using git with [The Luggage](http://github.com/unixorn/luggage).  [I've written about The Luggage in several articles on this site](http://glarizza.posterous.com/an-intro-to-using-the-luggage-for-packaging), so feel free to familiarize yourself with it if you haven't used the tool before.  As a quick review, The Luggage is a tool that lets you create plain text Makefiles that can be used to build packages to install files, deploy scripts, and basically distribute 'things' to the users in your organization.  Auditing HOW a package is made can be incredibly difficult for a team of sysadmins, but The Luggage helps you with this by using plain text Makefiles.  Git will give you the ability to track WHO made changes, WHEN they were made, and ROLLBACK the changes if you notice something undesirable happening.  Git and The Luggage are a perfect combo if you're a sysadmin.

Incidentally, The Luggage exists as a Github repository, so we have to CLONE it (or download it) from Github first before we can start playing with it.  I recommend creating a directory in your home directory (/Users/-username-) and cloning your repositories there - that way you retain ownership of the files and can work on them at will.  I put all my git repositories in ~/src, but if you're using something like Mobile Home Directories or even Network Home Directories, then you'll want to make sure that you're not syncing these directories back to some central location and burning up your quota in some fashion.  Let's create our repositories directory and clone The Luggage source code with these steps:

```
    $ mkdir -p ~/src
    $ cd ~/src
    $ git clone git://github.com/unixorn/luggage.git
```

Doing this will create the ~/src directory where we're putting all our repositories and download The Luggage source code to ~/src/luggage.  As you might be able to infer, git clone will clone an existing git repository from its location on another server (we call that the remote location, or just remote for short) down to a local directory.  This is usually how many sysadmins first encounter git - by needing to download something from Github or a remote server.  Let's look and see what git knows about this repo by doing the following:

```
    $ git remote -v
    origin  git://github.com/unixorn/luggage.git (fetch)
    origin  git://github.com/unixorn/luggage.git (push)
```

This command will list all the remote repositories that we're tracking.  By default, you will get a remote by the name of 'origin' that lists the location where you originally cloned the repository (in our case, this is Github).  

Why is it listed twice?  Well, with git, you can pull down, or fetch, changes from a remote repository as well as push up changes from your LOCAL repository to the remote repository.  Frequently these paths will be the same, but in our case these links are to the READ-ONLY version of this repository, so trying to push changes will always fail because we don't have access.  We'll talk later about how you can push up changes, but for now let's create our OWN repository instead of working with someone ELSE'S.

## Creating your OWN git repo

The Luggage has a directory called Examples, but it's absent of any actual examples.  We want to create our own directory where we can store our Luggage Examples.  For the purposes of this demo, I'm going to create a directory called luggage\_examples that will contain our...well... Luggage examples:

```
    $ cd ~/src
    $ mkdir luggage_examples
```

Now that we have a directory, we need to make sure that git is tracking our changes so that we can create those 'what if' scenarios and save our work.

```
    $ cd ~/src/luggage_examples
    $ git init
    Initialized empty Git repository in /Users/gary/src/luggage_examples/.git/
```

Git created a new empty repository at ~/src/luggage\_examples and created a .git directory inside luggage\_examples.  This .git directory is where git stores all its dynamic data that's pertinent to your repository, so MAKE SURE not to delete it or modify any files in there (without knowing what you're doing).

That's it!  We've created a repository!  Now let's actually see how git works locally on your machine.

## Working with Files

As I mentioned before, you don't NEED Github to use git.  Git functions as a mechanism to 'version' your files so you can rollback to a previous version or create our 'what-if' scenarios and then accept (merge) or deny (destroy) them depending on our needs.  You can easily create a git repository on your local computer ONLY, but of course you lose the ability to restore your work if your hard drive malfunctions.  This is what Github gives us, and we'll look at that functionality later.

The example I'm going to use is based on the previous blog post I wrote on [Using the Google Mac Sysops Crankd Code](http://glarizza.posterous.com/using-googles-crankd-application-tracking-cod)  Let's create a directory for this example and create a starter Makefile:

```
    $ mkdir -p ~/src/luggage_examples/crankd_google
    $ cd ~/src/luggage_examples/crankd_google
    $ vim Makefile
```

I use vi or vim to edit files from the Terminal.  Feel free to use any text editor you like (nano, emacs, Textmate, TextWrangler, etc...), but CREATE a file called 'Makefile' (note the capital M) and put it in ~/src/luggage\_examples/crankd_google.  Here's what I'll put into this file:

{% codeblock Crankd Luggage Example lang:make %}
    # Title:       Crankd-Google Example
    # Author:      Gary Larizza
    # Description: Something so Marnin doesn't kill me w/ postinstalls :)

    include /Users/gary/src/luggage/luggage.make

    TITLE=Crankd-Google
    REVERSE_DOMAIN=com.googlecode
    PAYLOAD=pack-crankd
{% endcodeblock %}

Great, so we have our skeleton of a file!  The file is saved in our folder, but we haven't actually told git to track the file (and so git is AWARE that the file EXISTS, but it isn't tracking its changes...yet).  This is a good time to explain the three states in which files can exist in a git repository.

## The Three Stages of Files: Working Tree, Index, and Repository

Git has three 'locations' by which a file can exist (even though it technically exists, in our case, in ~/src/luggage\_examples/crankd_google).  These three locations are called the working tree, the index, and the repository

##The Working Tree

When you put a file in a directory that's being managed by git, but you haven't told git to track the files changes, then this file is said to be in the working tree.  Our file we just created, Makefile, exists in the working tree currently.  Git knows that the file EXISTS, but it hasn't saved the file's 'state' so we can rollback the file if need be.  

##The Index

The index is also known as the cache or the staging area, and it's a place to put files before you're ready to make a 'commit' (which is something of a 'milestone' or a version).  A commit is a representation of a point in time in the history of your git repository.  Think of it as a snapshot of your repository - 'this is how our files looked at this specific time'.  With commits, you can always rollback to a specific 'commit' (or even advance to a future commit if you had rolled-back to an earlier commit).  In order to rollback to a specific version of a file, though, you need to make a commit, and in order to make a commit you need to tell git what FILES will comprise our commit.  As I mentioned, a commit is just a point in time...but it doesn't HAVE to be a point in time for a SINGLE file.  A commit can represent a specific point in time for a NUMBER of files all at once.  To make life easier for everyone, though, if you make a commit comprised of multiple files, then you want to make sure that the changes to the multiple files all represent a single functional change (for example, say we wanted to change a file that existed in our crankd_google folder that was being copied into a package, but we ALSO had to change the Makefile to change WHERE this file was to be put in the final package.  These two changes are related and comprise a single functional change, so you would want to make a commit with the changes to both files at once.). 

Our Makefile exists in the working tree currently, but we want to put it in the index because we're ready to make a commit (no, our Makefile functionally doesn't DO anything yet, but we want to make a commit so we can rollback to this point in time should we mess up anything).  Understand that the locations of the 'working tree' and 'index' are relative to git ONLY.  Yes, this file lives in ~/src/luggage\_examples, and it will CONTINUE to exist in that directory (as far as the OS X filesystem is concerned), but to git it currently exists in the working tree.  How do we know that the file is in the working tree and NOT in the index?  Use the git status command to show you that info:

```
    $ git status

    # On branch master
    #
    # Initial commit
    #
    # Untracked files:
    #   (use "git add ..." to include in what will be committed)
    #
    # Makefile
    nothing added to commit but untracked files present (use "git add" to track)
```

Anything listed under 'Untracked files:' is in the working tree.  You can add a file to the index by doing the following:

```
    $ git add Makefile
    $ git status

    # On branch master
    #
    # Initial commit
    #
    # Changes to be committed:
    #   (use "git rm --cached ..." to unstage)
    #
    # new file:   Makefile
    #
```

Great, the file is in the index!  Notice that the file is listed under 'Changes to be committed:' meaning that it's in the index but NOT YET committed (or, 'in the repository').

##The Repository

The Repository is where your files exist once they've been made part of a commit.  As I mentioned before, files must be committed before we can roll back their states (and their states can only be rolled back to individual commits).  You can never rollback a file's state UNLESS it's tied to a commit.  If you made a commit to a file last week and haven't made another since, but wanted to revert a file to how it existed three days ago, you would be out of luck.  Git ONLY rolls back to specific commits (this is why it's important to make frequent commits).  Before we make a commit, however, we need to make sure to identify ourself to git (if you haven't previously set this up).  The user.email and user.name settings are the most important settings as they will be used to trace back what code you committed.  If you're not sure of how git is setup, use the following command to list the git config settings:

```
    $ git config -l
    user.name=Gary Larizza
    user.email=gary@puppetlabs.com
```  

Should the user.name and user.email settings NOT be configured, you can configure them with the following commands:

```
    $ git config user.name 'Your name'
    $ git config user.email your@email.com
```

Now that you've been identified (so we can lay the blame on you in the future), let's actually commit some code.  The **git commit** command allows git to commit all staged files to the repository in one fell swoop.  

Every commit has three things: the list of files and CONTENT of the files that should be committed, a SHA1 hash value representing the commit itself (essentially, a unique identifier for the commit so it can be tracked), and the commit message describing the what and why of the commit.

**Creating a commit message seems insignificant but should NOT be taken lightly.**  Great commit messages usually consist of a title line containing no more than 60 characters followed by a paragraph describing the changes.  Because many people may potentially need to trace your code and commits, it's important to list how your code functioned previously, the reasoning behind the change, and what specifically was changed.  Don't skimp with your commit messages!  Always assume that the next person to maintain your code owns many dangerous weapons and knows where you sleep, so do that person a favor and be as kind to them (in the form of great commit messages) as possible!

Let's begin the process of making a commit by issuing the following command:

```
    git commit
```

Executing **git commit** will drop you into the default text editor (which is vim by default - you can change this by running **git config core.editor /path/to/your/editor** before doing git commit) and show the following:

```
    # Please enter the commit message for your changes. Lines starting
    # with '#' will be ignored, and an empty message aborts the commit.
    # On branch master
    #
    # Initial commit
    #
    # Changes to be committed:
    #   (use "git rm --cached ..." to unstage)
    #
    # new file:   Makefile
    #
```

Note that everything that begins with a hash ( # ) is solely for YOUR benefit and will be stripped out of the commit message, so don't worry about it being included.  Here's an example of a commit message I would use in this instance:

```
    Initial commit for crankd_google code

    Previously we didn't have an example for creating a package of the 
    Google Mac Sysops code.  This commit adds the shell for the Makefile and
    includes the luggage.make file in ~/src/luggage.
```

Great!  When you save and close your editor, you should see something that looks like this:

```
    [master (root-commit) f423ac6] Initial commit for crankd_google code
     1 files changed, 9 insertions(+), 0 deletions(-)
     create mode 100644 Makefile
```

This tells us that we made a commit on the master branch with a SHA1 hash ID beginning in f423ac6 being attributed to this commit.  In most cases, the first 7 digits of the hash are all you need for uniqueness when you refer to this commit, and that's usually all that git will provide you in THIS instance.  YOUR SHA1 hash won't match the hash provided above (hence a unique ID), so don't panic if it doesn't match up for you.  And that's it!  We've done it!  Now what?

##Examining your Work

Git tracks the ENTIRE history of a repository, and thus will track any and every commit made.  To display this history, run the following command:

```
$ git log
commit f423ac6e9548d93f53b15d1154723c5abcd69738
Author: Gary Larizza 
Date:   Sun Jan 22 23:27:31 2012 -0500

    Initial commit for crankd_google code

    Previously we didn't have an example for creating a package of the
    Google Mac Sysops code.  This commit adds the shell for the Makefile and
    includes the luggage.make file in ~/src/luggage.
```

By default, git log will show ALL information about ALL of your commits (including the full commit message).  There's a LARGE number of arguments that git log will accept that allows you to format the output for ANY purpose.  One of the common lists of arguments that I like to employ is the following (long) command:

```
    $ git log --pretty=format:'%C(yellow)%h%C(reset) %s %C(cyan)%cr%C(reset) %C(blue)%an%C(reset) %C(green)%d%C(reset)' --graph --date-order
    * f423ac6 Initial commit for crankd_google code 10 minutes ago Gary Larizza  (HEAD, master)
```

This format both colorizes the output for readability and also shows a nice graph of commits as you do things like branching and merging.  Here's an example of this command (with line length chopped for brevity) being run on a repository with many commits:

```
    * 72a2fe0 (#5445) Create /var/lib/puppet in OS X Package 26 hours ago Gary Larizza  (HEAD, gary/bug/2.7.x/5445_crea
    *   c6667c5 Merge pull request #324 from glarizza/bug/2.7.x/2273_launchd_dashw 12 days ago Daniel Pittman  (origin/
    |\  
    * \   bc6c642 Merge pull request #318 from glarizza/bug/2.7.x/11202_apple_rake 12 days ago Michael Stahnke 
    |\ \  
    | * | 6c14a28 Build a Rake task for building Apple Packages 12 days ago Gary Larizza  (gary/bug/2.7.x/11202_apple_r
    * | |   5ec5657 Merge pull request #127 from adrienthebo/ticket/2.7.x/8341-prevent_duplicate_loading_of_facts 12 da
    |\ \ \  
    * \ \ \   450da6a Merge pull request #92 from duritong/tickets/2.7.x/1886 12 days ago Daniel Pittman 
    |\ \ \ \  
    | | | | * d5bef5e (#2773) Use launchctl load -w in launchd provider 12 days ago Gary Larizza  (gary/bug/2.7.x/2273_
    * | | | |   1cb2b6a Merge branch 'ticket/2.7.x/11714-envpuppet' into 2.7.x 12 days ago Josh Cooper 
    |\ \ \ \ \  
    | |_|_|_|/  
    |/| | | |   
    | * | | | 5accc69 (#11714) Use **%~dp0** to resolve bat file's install directory 12 days ago Josh Cooper 
    * | | | |   fe79537 Merge pull request #323 from glarizza/bug/2.7.x/fix_launchd_spec 12 days ago Jeff McCune 
    |\ \ \ \ \  
    | * | | | | c865a80 Clean up launchd spec tests 12 days ago Gary Larizza  (gary/bug/2.7.x/fix_launchd_spec, bug/2.7
    |/ / / / /
```

As you can see, git log is powerful and handy to list out what's happened to your repository.  You'll also notice that it lists WHO made the changes, which is handy if you have MANY people contributing to the same code base.

##That's it for now

Hey , what gives?! We haven't really DONE anything!  Git has a tremendous amount of power, and it's important to know WHAT'S happening before I can progress and show you the power of git.  This article is ALREADY quite long, so I've broken it up for 'easier' reading.  Rest assured, in parts 2 and beyond we will modify files, create topic branches, merge topic branches, use Github (for backup AND to allow others to contribute to your project), and much more.  Stay tuned for the following article(s) and as always, put any questions in the comments or email me directly!

