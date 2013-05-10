---
layout: post
title: "Refining My Vim: Better Rakefile"
date: 2012-03-04 18:30
comments: true
categories: vim
---
So tonight I am taking a closer look at my [Vim config](https://github.com/vim-files) to see what I can change to help make my vim experience better. A few months ago I got away from using pre-configured setups of Vim (such as [janus](https://github.com/carlhuda/janus)) and setup my own Vim config from scratch. I figured this would make a good blog series focusing on one thing at a time, so here we go.

## Better Rakefile
One of the things I did when creating my own Vim configuration was created a Rakefile in my ~/.vim directory that makes my setup more mobile. My current rakefile basically has a list of plugins that I want installed, then checks if a folder already exists with a name matching the plugins name, if it doesn't then it will download the plugin via git. There is some functionality missing from this that I would like to have, and there is some repetition that bothers me. Before we begin, here is what my current rakefile looks like.

``` ruby rakefile
task :default do
  clone "https://github.com/scrooloose/nerdtree.git", "NERDTree"
  clone "https://github.com/vim-scripts/The-NERD-Commenter.git", "NERD-Commenter"
  clone "https://github.com/vim-scripts/L9.git", "L9"
  clone "https://github.com/wincent/Command-T.git", "Command-T"
  clone "https://github.com/mileszs/ack.vim.git", "Ack"
  clone "https://github.com/tpope/vim-rails.git", "Rails"
  clone "https://github.com/vim-ruby/vim-ruby.git", "Ruby"
  clone "https://github.com/mattn/gist-vim.git", "Gist"
  clone "https://github.com/davidoc/taskpaper.vim.git", "TaskPaper"
  clone "https://github.com/vim-scripts/vimwiki.git", "VimWiki"
  clone "https://github.com/tpope/vim-endwise.git", "Endwise"
  clone "https://github.com/ervandew/supertab", "SuperTab"
  clone "https://github.com/lucapette/vim-ruby-doc.git", "Ruby-Doc"
  clone "https://github.com/rbgrouleff/bclose.vim.git", "BClose"
  clone "https://github.com/bartekd/better-snipmate-snippets.git", "betterSnipMate"
  clone "https://github.com/tpope/vim-surround", "surround"

  # Syntax Files
  clone "https://github.com/tpope/vim-markdown.git", "Markdown"
  clone "https://github.com/leshill/vim-json.git", "JSON"
  clone "https://github.com/tpope/vim-haml.git", "HAML"
  clone "https://github.com/kchmck/vim-coffee-script.git", "CoffeeScript"
  clone "https://github.com/pangloss/vim-javascript.git", "Javascript"
  clone "https://github.com/vim-scripts/eruby.vim.git", "eruby"
  clone "https://github.com/groenewege/vim-less.git", "Less"

  # Colors
  clone "https://github.com/altercation/vim-colors-solarized.git", "Color-Solarized"
  clone "https://github.com/squil/vim_colors.git", "Squil-Colors"
  clone "https://github.com/tomasr/molokai.git", "Molokai"
end

def clone(url, name)
  if File.exists?("bundle/#{name}")
	puts "[Exists] #{name}"
  else
	puts "[Installing] #{name}"
	system("git clone #{url} bundle/#{name}")
	puts "[Installed] #{name}"
  end
end
```

### Lets identify the problems I have with this.
* "clone" is getting called over and over, wouldn't it be better if we could just have an object of the git repos and their names, then grab them as needed?
* I cannot list installed plugins.
* I cannot update a single plugin, in order to update a plugin I have to remove the folder that the plugin is installing into, and run the rake command.
* I cannot easily remove a single plugin using the rake command

### Lets fix those problems
Lets start at the top and see what we can do to refactor this code to remove the repetition. The simplest thing I can think of is to create a method called plugins that returns a hash with the plugin name as a key and the git url as its value. So we end up with something like this. (Shortened for times sake)

``` ruby rakefile
def plugins
  { "NERDTree"		=> "https://github.com/scrooloose/nerdtree.git",
	"NERD-Commenter"  => "https://github.com/vim-scripts/The-NERD-Commenter.git",
	"L9"			  => "https://github.com/vim-scripts/L9.git",
	"Ack"			 => "https://github.com/mileszs/ack.vim.git",
	"Rails"		   => "https://github.com/tpope/vim-rails.git",
	"Ruby"			=> "https://github.com/vim-ruby/vim-ruby.git",

	# Syntax Plugins
	"Markdown"		=> "https://github.com/tpope/vim-markdown.git",
	"JSON"			=> "https://github.com/leshill/vim-json.git",
	"HAML"			=> "https://github.com/tpope/vim-haml.git",

	# Colors
	"Solarized"	   => "https://github.com/altercation/vim-colors-solarized.git",
	"Squil-Colors"	=> "https://github.com/squil/vim_colors.git",
	"Molokai"		 => "https://github.com/tomasr/molokai.git"
  }
end
```

In order to make use of this new hash method we will need to update our default rake task like so.

``` ruby rake
task :default do
  plugins.each_pair { |key, value| clone key, value}
end
```
To me this is much more pleasing for a couple reasons, one reason being I am not repeatedly calling having to type clone, can just add the name and url of the plugin to the hash. Secondly and more importantly is the fact that the plugins method is reusable to fix my other issues.

### One down three to go
So another feature I want in my rakefile is the ability to call ```rake list``` and getting a list of installed plugins. This is really easy to do, we are going to create 2 methods, the rake task and then a method to get a list of installed apps. First we will make the method that gets the list for us.

``` ruby rakefile
def installed_plugins
  Dir.foreach("bundle").drop(2)
end
```
This is a very simple method that just gets a list of of files/directories in the "bundle" directory. However when you use ```Dir.foreach()``` it also returns 2 unnecessary entries, "." and "..". Luckily they are at the top of the list so we easily get rid of them with ```.drop(2)```. We now have our list so lets put it to use and create our rake task.

``` ruby rakefile
task :list do
  puts "Plugins installed in ~/.vim/bundle"
  installed_plugins.each { |plugin| puts plugin }
end
```
We can now use this rake task by running ```rake list``` in terminal.

### Halfway there
We have two problems out of the way and now we are going to tackle updating single plugins. Before we do that we need to consider how we want to handle updating. Do we want to check the git repository and see if there are updates? Or do we want to just delete the directory and re-install the plugin? I personally do not store any configuration in my plugin folders, so I have no issues with taking the easy route and just deleting the directory and re-installing the plugin, however if this is not the case for you, you might want to find a better way to handle this.

Before we try and delete the existing copy of the plugin we want to make sure it exists with this simple method.

``` ruby rakefile
def plugin_installed?(name)
  if installed_plugins.include? name
    return true
  else
    puts "Plugin not found in bundle direcotry"
    return false
  end
end
```

We also want to make sure that plugin is in our list of plugins before we delete the local copy. We can easily do that by checking our hash that we created earlier includes the name of the plugin.

``` ruby rakefile
def plugin_in_plugins?(name)
  if plugins.include? name
    return true
  else
    puts "Plugin not in hash of plugins"
    return false
  end
end
```

Once we have verified that the plugin exists in our hash and locally in the "bundle" directory we can remove it.

``` ruby rakefile
def delete_plugin(name)
  if plugin_installed?(name)
    puts "[Removing] #{name}"
    FileUtils.rm_rf("bundle/#{name}")
    puts "[Removed] #{name}"
  end
end
```

We will then use our existing clone method to re-install the plugin. With all our building blocks in place we will now put them together in a update method to handle the logic of verifying that we can in fact update the plugin and then execute the methods to update it.

``` ruby rakefile
def update(name)
  if plugin_installed?(name) && plugin_in_plugins?(name)
    delete_plugin(name)
    clone(name, plugins.values_at(name)[0])
  end
end
```

Finally we need to create the rake task to put all this code to use!

``` ruby rakefile
task :update, [:name] do |t, args|
  update(args.name)
end
```

This rake task will accept an argument for the name of the plugin and can be used by running ```rake update[pluginName]``` in terminal. (If you are using Zsh you will need to add \ in front of the square brackets like ```rake update\[pluginName\]```)

### Last but not least
The last item on my wish list is to be able to be able to remove a plugin by running ```rake destroy[pluginName]```. Luckily by trying to make our methods as reusable as possible we already developed the "delete_plugin" method and just need to add the rake task.

``` ruby rakefile
task :destroy, [:name] do |t, args|
  delete_plugin(args.name)
end
```
After running ```rake destroy[pluginName]``` of course we will need to open up our rakefile and remove the plugin from our hash.

### Wrap up
After we have gone through all the steps here is what we end up with.

``` ruby rakefile
task :default do
  plugins.each_pair { |key, value| clone key, value}
end

task :list do
  puts "Plugins installed in ~/.vim/bundle"
  installed_plugins.each { |plugin| puts plugin }
end

task :update, [:name] do |t, args|
  update(args.name)
end

task :destroy, [:name] do |t, args|
  delete_plugin(args.name)
end

def plugins
  {
    "NERDTree"        => "https://github.com/scrooloose/nerdtree.git",
    "NERD-Commenter"  => "https://github.com/vim-scripts/The-NERD-Commenter.git",
    "L9"              => "https://github.com/vim-scripts/L9.git",
    "Ack"             => "https://github.com/mileszs/ack.vim.git",
    "Rails"           => "https://github.com/tpope/vim-rails.git",
    "Ruby"            => "https://github.com/vim-ruby/vim-ruby.git",
    "Gist"            => "https://github.com/mattn/gist-vim.git",
    "TaskPaper"       => "https://github.com/davidoc/taskpaper.vim.git",
    "VimWiki"         => "https://github.com/vim-scripts/vimwiki.git",
    "Endwise"         => "https://github.com/tpope/vim-endwise.git",
    "SuperTab"        => "https://github.com/ervandew/supertab",
    "Ruby-Doc"        => "https://github.com/lucapette/vim-ruby-doc.git",
    "BClose"          => "https://github.com/rbgrouleff/bclose.vim.git",
    "BetterSnipMate"  => "https://github.com/bartekd/better-snipmate-snippets.git",
    "Surround"        => "https://github.com/tpope/vim-surround",

    # Syntax Plugins
    "Markdown"        => "https://github.com/tpope/vim-markdown.git",
    "JSON"            => "https://github.com/leshill/vim-json.git",
    "HAML"            => "https://github.com/tpope/vim-haml.git",
    "CoffeeScript"    => "https://github.com/kchmck/vim-coffee-script.git",
    "Javascript"      => "https://github.com/pangloss/vim-javascript.git",
    "Eruby"           => "https://github.com/vim-scripts/eruby.vim.git",
    "Less"            => "https://github.com/groenewege/vim-less.git",

    # Colors
    "Solarized"       => "https://github.com/altercation/vim-colors-solarized.git",
    "Squil-Colors"    => "https://github.com/squil/vim_colors.git",
    "Molokai"         => "https://github.com/tomasr/molokai.git"
  }
end

def clone(name, url)
  if File.exists?("bundle/#{name}")
    puts "[Exists] #{name}"
  else
    puts "[Installing] #{name}"
    system("git clone #{url} bundle/#{name}")
    puts "[Installed] #{name}"
  end
end

def update(name)
  if plugin_installed?(name) && plugin_in_plugins?(name)
    delete_plugin(name)
    clone(name, plugins.values_at(name)[0])
  end
end

def installed_plugins
  Dir.foreach("bundle").drop(2)
end

def plugin_installed?(name)
  if installed_plugins.include? name
    return true
  else
    puts "Plugin not found in bundle direcotry"
    return false
  end
end

def plugin_in_plugins?(name)
  if plugins.include? name
    return true
  else
    puts "Plugin not in hash of plugins"
    return false
  end
end

def delete_plugin(name)
  if plugin_installed?(name)
    puts "[Removing] #{name}"
    FileUtils.rm_rf("bundle/#{name}")
    puts "[Removed] #{name}"
  end
end
```

There is a lot you can do with a rake file to help make your Vim setup easier to add and remove to. You don't have to limit rake files to your Ruby apps or Vim setups either, there are many problems where a nice rake file is a viable solution.
