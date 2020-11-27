### Configuration Files

Git’s configuration files are all simple text files in the style of .ini files. They record various choices and settings used by many Git commands. Some settings represent purely personal preferences (should a `color.pager` be used?); others are vital to a repository functioning correctly (`core.repositoryformatversion`); and still others tweak command behavior a bit (`gc.auto`). Like many tools, Git supports a hierarchy of configuration files. In decreasing precedence they are (in Linux):

**.git/config**
		Repository-specific configuration settings manipulated with the `--local` option or by default. These settings have the highest precedence.

**~/.gitconfig**
		User-specific configuration settings manipulated with the `--global` option.

**/etc/gitconfig**
		System-wide configuration settings manipulated with the `--system` option if you have proper Unix file write permissions on it. These settings have the lowest precedence. Depending on your actual installation, the system settings file might be somewhere else (perhaps in `/usr/local/etc/gitconfig`), or may be entirely absent.

For example, to establish an author name and email address that will be used on all the commits you make for all of your repositories, configure values for `user.name` and `user.email` in your `$HOME/.gitconfig` file using `git config --global`:

```
$ git config --global user.name "Jon Loeliger"
$ git config --global user.email "jdl@example.com"
```

Or, to set a repository-specific name and email address that would override a `--global` setting, simply omit the `--global` flag (you can also use `git config --local` explicitly):

```
$ git config user.name "Jon Loeliger"
$ git config user.email "jdl@special-project.example.org"
```

Instead of using `--local`, `--global` and `--system`, you can use `--file` to explicitly the configuration file using the file path:

```
$ git config --file ./.git/config user.name "Jon Loeliger"
```

`-l` parameter can be used to display all configuration variables:

```
$ git config -l
$ git config --local -l
$ git config --global -l
$ git config --system -l
```

Or if you want to check the value of a particular variable, you can use `--get`:

```
$ git config --get user.name
$ git config --local --get user.name
$ git config --global --get user.name
$ git config --system --get user.name
```

References:
[1] Version Control with Git Powerful tools and techniques for collaborative software development