## Installation

Install as a global package to enable commandline execution of `cpm`, `cgulp`, `cgitsh`, and `oauth` from any location.

```cmd
npm install -g {name}
```

### Pre-requisites

  * Windows 7/8
    * Unix has not been tested and for this reason is unsupported.
  * [GITWin32 OpenSSL v1.0.1g](http://slproweb.com/products/Win32OpenSSL.html).
    * When installing OpenSSL, you must tell it to put DLLs in the Windows system directory
  * Node packages (global):
    * node-gyp
        * Python ([windows-python-v2.7.3](http://www.python.org/download/releases/2.7.3#download) recommended, `v3.x.x` is __*not*__ supported)
        * Microsoft Visual Studio 2013 for Windows Desktop (Express version works well)
    * gulp
  * Git
    * cygwin is recommended for Windows and use with Atlassian Bamboo.  Install packages git-completion, git-gui, gitk, and openssh.
  * Atlassian suite - for build tasks integrating with Atlassian JIRA or Atlassian Bamboo.

Only 32bit versions of the following pre-requisites have been tested.

### Oauth config

The `oauth` command runs the `oauth-rest-atlassian` package -an OAuth wrapper to authenticate and use the Atlassian REST API. The initial authorisation dance is managed through a local web page.  Follow the instructions from the [package readme](https://www.npmjs.org/package/oauth-rest-atlassian) to create a config.json and private key.  Save these files to the root of this package.



## Usage

### From the command line

The `cpm` command accepts two arguments - the name of the cpm function to be executed and a parameter to pass to the function.

```cmd
cpm install myModule
```

The `cgulp` command accepts one argument - the name of the gulp task to be executed.

```cmd
cgulp test
```

The `cgitsh` command accepts one argument - the name of the bash script to be executed.

```cmd
git-bamboo Git-clone
```

The `oauth` command does not require any arguments.  However, a config.json and a private key must be available in the root of this package.

```cmd
oauth
```

Each bash script uses Atlassian Bamboo build variables.  These variables can be set manually if calling the script outside of Atlassian Bamboo or if you want to override the variable.  For example, to override the build repository:

```cmd
set bamboo_planRepository_repositoryUrl="git@github.com:Cellarise/script-git-bamboo.git"
git-bamboo Git-clone
```


# cpm functions

cpm provides a wrapper around npm commands.

## gulp

Execute the `cgulp` command. First remove any build folders created by a previous execution of `cgulp`. The second argument past to `cpm` is used as the task to be executed.

```bat
REM delete Build, Reports, and Temp folder contents
rmdir .\Build /s /q
rmdir .\Reports /s /q
rmdir .\Temp /s /q
call cgulp %2
```

## clean

Remove build folders created by `cgulp`.

```bat
rmdir .\Build /s /q
rmdir .\Reports /s /q
rmdir .\Temp /s /q
```

## install

Install modules (production dependencies only). First prune any modules no longer in package.json and after installation deduplicate. This function uses the second argument passed to `cpm` as the module name to install.

```bat
REM set path to programs required by installers
set path = c:\OpenSSL-Win32\bin;%windir%\Microsoft.NET\Framework\v4.0.30319; %path%
call npm prune
call npm install --msvs_version=2013 --production %2
call npm dedupe --msvs_version=2013
```

## publish

Publish to npm

```bat
REM set path to programs required by installers
set path = c:\OpenSSL-Win32\bin;%windir%\Microsoft.NET\Framework\v4.0.30319; %path%
call npm publish
```

## update

Update modules (production dependencies only). First prune any modules no longer in package.json and after installation deduplicate. This function uses the second argument passed to `cpm` as the module name to update.

```bat
REM set path to programs required by installers
set path = c:\OpenSSL-Win32\bin;%windir%\Microsoft.NET\Framework\v4.0.30319; %path%
call npm prune
call npm update --msvs_version=2013 --production %2
call npm dedupe --msvs_version=2013
```


# cgulp functions

cgulp enables execution of global tasks defined in this package. See the API for a definition of the avaialble tasks.



# cgitsh functions

## Git-clone

Clone the first repository linked to the bamboo build plan. Clone into folder Git.

```sh
git clone "$bamboo_planRepository_repositoryUrl" Git
exit
```

## Git-release-version

Clone the first repository linked to the bamboo build plan. Clone into folder Git. Then create a new branch to track the change and tag it with the Atlassian Bamboo Jira version. Finally merge the release branch into the production branch.

```sh
git clone "$bamboo_planRepository_repositoryUrl" Git
cd Git
git checkout master
git checkout -b "release/$bamboo_jira_version"
git tag -a "v$bamboo_jira_version" -m "Release v$bamboo_jira_version"
git push origin "release/$bamboo_jira_version" --tags
git checkout production
git merge "release/$bamboo_jira_version"
git tag -a "v$bamboo_jira_version" -m "Release v$bamboo_jira_version" production
git push origin production --tags
exit
```

## Github-clone

Clone repository from github using the Atlassian Bamboo Jira project name. Clone into folder Build.

```sh
git clone git@github.com:"$bamboo_jira_projectName".git Build
exit
```

## Github-release-version

Stage all changes, commit, and push to the master branch.  Then create a new branch to track the change and tag it with the Atlassian Bamboo Jira version.

```sh
git rm . -r --cached
git add .
git status
git commit -m "Deploy release $bamboo_jira_version"
git push origin master
git checkout -b "release/$bamboo_jira_version"
git tag -a "v$bamboo_jira_version" -m "Release v$bamboo_jira_version"
git push -u origin "release/$bamboo_jira_version" --tags
exit
```
