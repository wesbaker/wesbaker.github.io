---
layout:     post
title:      "My ExpressionEngine Add-On Development Setup"
date:       2015-05-14 17:40:00 -0400
categories:
    - programming
---
I make an add-on for ExpressionEngine called [Subscriber](https://devot-ee.com/add-ons/subscriber). It’s a pretty simple little add-on: when a visitor fills out a form on your website they’re added to a newsletter list either automatically or based on a selection they make.

Developing Subscriber has been pretty simple. I have a directory  (and therefore repository) for Subscriber and a directory for my test site. I'll work almost exclusively in the test site and copy things over when I'm satisfied with how it's working. I've missed some changes in the past when copying things over. In addition, it was something of a process when I had files in two different directories.

I knew I was being inefficient and I knew it could be better than copying and pasting files back and forth while hoping I grabbed everything. I knew my text editor could help me out, so I started doing some digging.

## Sublime

It all starts with [Sublime Text](http://www.sublimetext.com). Sublime Text is a fantastic editor, one that I’ve been using for a few years now, and I’ve been using [Atom](https://atom.io) here and there, but I’m still in Sublime Text for the majority of the day.

Sublime Text doesn’t look like much, but there’s a lot that makes it powerful under the hood. In particular, it’s ability to let you customize nearly everything and add in various packages and add-ons make it key.

For my purposes, the first step was…

## Project Settings

As I said before, I keep my add-on's directory separate from the site I use to test my add-on. When I'm working on my add-on I need access to both the test site's files and the add-on. Sublime Text makes this easy, simply open one directory and then drag a second in and all of the files are available.

However, I now had a new problem. I heavily rely on Sublime Text's Go to File. If you've not used it before, you hit `⌘ + T` and it shows you all of the files in your project, start typing and you're now filtering the files. If you have two files with the same name you can tell which directory they're in, but it's a little confusing still.

In order to remove one of those directories, you're going to have to create a new Sublime Project from the Project menu (Project -\> Save Project As...). It's asks for a name and location for the project file and I chose my add-on's directory, but it can be placed anywhere. Now with that created, you're going to have to edit the settings by selecting Project -\> Edit Project. You should see something like the following:

	{
			"folders":
			[
				{
				"path": "/Users/wes/Sites/sandbox",
			},
			{
				"path": "/Users/wes/Development/ee.addons/subscriber",
			}
		]
	}

All I had to do was add one line to remove the duplicate add-on directory and I removed it from my sandbox site:

	{
			"folders":
			[
				{
				"path": "/Users/wes/Sites/sandbox",
				"folder_exclude_patterns": ["subscriber"]
			},
			{
				"path": "/Users/wes/Development/ee.addons/subscriber",
			}
		]
	}

With that in place I'm editing the add-on's files directly in the add-on's directory and the test site's files in the test site's directory. However, I still had a problem: copying the files.

## Build Script

Sublime Text has a system in place for project builds. With some projects that might mean a makefile or a rakefile. For my purposes, I only needed to copy files from one directory to another.

Back in the project's settings file, we need to define the build system for my add-on project. Just after the closing `]` for `folders` I added this:

	"build_systems":
	[
		{
			"name": "Copy",
			"cmd": ["cp", "-r", "/Users/wes/Development/ee.addons/subscriber/system/expressionengine/third_party/subscriber/", "/Users/wes/Sites/sandbox/system/expressionengine/third_party/subscriber/"]
		}
	],

The name is pretty self explanatory, but the `cmd` might need a quick description. You take your raw command and break it up into array items at spaces. For example:

	cp -r old new

Would turn into:

	"cmd": ["cp", "-r", "old", "new"]

Now, all I have to do to copy the files over is run the build by pressing ⌘ + B. Yet, I'm lazy and there should be a way to do this automatically...

## Autobuild

The last piece of the puzzle is a package called [SublimeOnSaveBuild](https://github.com/alexnj/SublimeOnSaveBuild) that does exactly what you're thinking: runs the build whenever you save. However, I ran into two problems with it:

1. By default it runs Build on any and every project, and I want that off by default.
2. By default it only works when you save `css`, `js`, `sass`, `less`, and `scss` files.

In order to fix this, I needed to make changes in two places. First, I went to Preferences -\> Package Settings -\> SublimeOnSaveBuild -\> Settings - User and added the following settings:

	{
	    "filename_filter": "\\.(css|js|sass|less|scss|php|json)$",
	    "build_on_save": 0
	}

The first line ensures that it runs when I'm saving `php` files as well as `json` files and the second line disables the functionality by default.

Next, I needed to enable this for my project, so back in the project settings, below the `]` from `build_systems` I added this:

	"settings":
	{
		"build_on_save": 1
	}

This changes Sublime's settings specifically for this project, turning SublimeOnSaveBuild back on.

## Final Project Settings

With everything in place, I'm now able to work on my add-on in a test site while leaving it in its own directory so I can ensure that all changes are committed to the repository. I'm saving time by not copying over files and everything is kept up to date instead of missing something from time to time. Here's the final `sublime-project` file:

	{
		"folders":
		[
			{
				"path": "/Users/wes/Sites/sandbox",
				"folder_exclude_patterns": ["subscriber"]
			},
			{
				"path": "/Users/wes/Development/ee.addons/subscriber",
			}
		],
		"build_systems":
		[
			{
				"name": "Copy",
				"cmd": ["cp", "-r", "/Users/wes/Development/ee.addons/subscriber/system/expressionengine/third_party/subscriber/", "/Users/wes/Sites/sandbox/system/expressionengine/third_party/subscriber/"]
			}
		],
		"settings":
		{
			"build_on_save": 1
		}
	}

If you have a different approach to building your add-on, I'd love to hear about it. Also, if you've been using Atom to do something similar, I'd definitely love to hear from you. Send me a tweet [@wesbaker](https://twitter.com/wesbaker).
