takeup
======

Starts and stops shit for you. See this [screencast](http://www.screenr.com/BzJ) (duration: 1min, no sound).

Example: You're working on bunch of projects, each with different things you need to run. One requires Redis/Resque, another Postgres and DJ, third uses MySQL and Sphinx. Plus, you don't want all of these to run in background continuously, because it's a waste of resources. Solution is as follows.

1. write a takeup manifest
2. partey

## Install

1. git clone git://github.com/maxim/takeup.git ~/somewhere
2. ln -s ~/somewhere/takeup ~/bin/takeup (or whatever is in your path)

## Commands

takeup understands very few commands. Obviously, you have to be in your project directory for takeup to know which manifest to use.

- `takeup` — start everything in manifest
- `takeup minimal` — start only the things marked as `required`
- `takeup status` — see what's running or not (shorter: `takeup st`)
- `takeup stop` — stop everything
- `takeup unicorn` — start only the thing named unicorn
- `takeup stop unicorn` — stop only the thing named unicorn
- `takeup restart unicorn` — attempt to gracefully stop and start unicorn

## Storing manifests

Manifests can either be stored in your home directory (`~/.takeup`), or under your project's directory (`~/dev/printio/.takeup`), depending on your use case.

### Storing manifests in your home dir 

Useful when you just want to write a manifest for yourself without bothering anyone else. Simply throw one into your home dir.

    ~/.takeup/project_name/manifest.yml

The project\_name should be the dirname of your project. For example, if my project is in `~/dev/foobar` then the project_name is foobar.

### Storing manifests in the project's dir

You can also keep your manifests in the project's dir, versioned. In this case takeup looks up your hostname using `hostname -s` command to scope manifests per developer.

    /path/to/project/.takeup/hostname/manifest.yml

This is also helpful if you're changing your project's architecture (e.g. moving from delayed\_job to resque) and would like to modify takeup manifest in a particular git branch.

However, if a manifest for this project is also found in your home dir, it will take precedence over the one versioned with the project.

## Writing manifests

Manifests are yaml files which tell takeup what to run to start/stop things.

Here's an example manifest I have for printio.

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
- :pid\_file	— the exact thing you specified in `pid_file` yml entry, in case you want to reuse it in your `start` or stop command (see how I stop unicorn in the example above)
- :support\_root — points to the takeup dir for this project (e.g. `~/.takeup/project_name` or `~/dev/project/.takeup/hostname/`, depending on where your takeup manifest is found) so that you can throw config files in there and reference them in start/stop (see example above)

## Options

### Wait mode

You can add `--wait` (or `-w`) flag to any of the start/stop commands. In this mode, takeup will not move on until it ensured that a process has indeed started or stopped (by polling for pid file's existence).

### Debug mode

You can add `--debug` (or `-d`) to any of the commands (listed above) to prevent takeup from eating your STDOUT/STDERR (which it does by default, because I like pretty.)


## More examples

I have more manifests in my [Dotfiles](https://github.com/maxim/dotfiles/tree/master/takeup).

## License

Copyright © 2011 Maxim Chernyak

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.