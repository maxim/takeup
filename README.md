takeup
======

Starts and stops shit for you. See this [screencast](http://www.screenr.com/BzJ) (duration: 1min, no sound).

Example: You're working on bunch of projects, each with different things you need to run. One requires Redis/Resque, another Postgres and DJ, third uses MySQL and Sphinx. Plus, you don't want all of these to run in background continuously, because it's a waste of resources. Solution is as follows.

1. write a takeup manifest
2. partey

##### WHY U NO SHELL SCRIPT?

Me no good with shell script.

##### WHY U NO GEM?

Don't want to have to install it for each project running on different ruby/gemset (in rvm).

## Install

1. git clone git://github.com/maxim/takeup.git ~/somewhere
2. ln -s ~/somewhere/takeup ~/bin/takeup (or whatever is in your path)
3. mkdir ~/.takeup (this is where manifests go)

## Commands

takeup understands very few commands. Obviously, you have to be in your project directory for takeup to know which manifest to use.

- takeup — start everything in manifest
- takeup status — see what's running or not (shorter: takeup st)
- takeup stop — stop everything

- takeup unicorn — start only the thing named unicorn
- takeup stop unicorn — stop only the thing named unicorn

- takeup minimal — start only the things marked as `required`

## Manifests

Manifests are yaml files which tell takeup what to run to start/stop things. They go into `~/.takeup/project_name/manifest.yml`.

Here's an example manifest I keep in `~/.takeup/printio/manifest.yml`.

		- name:       "jaxer"
		  pid_file:   ":project_root/tmp/pids/jaxer.pid"
		  start:      ":support_root/jaxer_start"
		  stop:       ":support_root/jaxer_stop"

		- name:       "postgres"
		  pid_file:   "/usr/local/var/postgres/postmaster.pid"
		  start:      "pg_ctl -D /usr/local/var/postgres -l :project_root/log/postgres.log start"
		  stop:       "pg_ctl -D /usr/local/var/postgres stop -s -m fast"
		  required:   true

		- start:      "sleep 2"

		- name:       "unicorn"
		  pid_file:   ":project_root/tmp/pids/unicorn.pid"
		  start:      "unicorn_rails -D -c :support_root/unicorn.conf"
		  stop:       "kill -QUIT `cat :pid_file`"
		  required:   true

		- name:       "sphinx"
		  pid_file:   ":project_root/tmp/pids/sphinx.pid"
		  start:      "rake ts:start"
		  stop:       "rake ts:stop"

		- name:       "delayed_job"
		  pid_file:   ":project_root/tmp/pids/delayed_job.pid"
		  start:      ":support_root/delayed_job start"
		  stop:       ":support_root/delayed_job stop"

takeup is really stupid. It simply runs each `start` in the order you listed it. When you ask it to `stop` — it runs each `stop` in reverse order. That's it.

### YAML keys

None of them are required really. As you see above, I have an entry that only does `start: 'sleep 2'` between postgres and unicorn.

- name — pretty display name for service being run
- pid\_file — location of pid_file which tells takeup if something is running
- start — the actual command to start something (daemonized)
- stop — the actual command to stop something
- required — if true, this dependency will launch when you do `takeup minimal`

### Interpolated values

takeup knows how to interpolate these colon-prefixed vars when you use them in your manifests. Here's what it will insert in their place.

- :project\_name — the name of directory in which you run takeup
- :project\_root — full path to directory in which you run takeup
- :pid\_file	— the exact thing you specified in `pid_file` yml entry, in case you want to reuse it in your `start`  or stop command (see how I stop unicorn in the example above)
- :support\_root — points to `~/.takeup/project_name` so you can throw config files in there and reference them in start/stop (see example above)

## Debugging

You can add `--debug` to any of the commands (listed above) to prevent takeup from eating your STDOUT/STDERR (which it does by default, because I like pretty.)

## More examples

I have more manifests in my [Dotfiles](https://github.com/maxim/dotfiles/tree/master/takeup).

## License

Copyright © 2011 Maxim Chernyak

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.