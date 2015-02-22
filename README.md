[![Build Status](https://travis-ci.org/mirakui/ec2ssh.png?branch=master)](https://travis-ci.org/mirakui/ec2ssh)

# Introduction
ec2ssh is a ssh_config manager for Amazon EC2.

`ec2ssh` command adds `Host` descriptions to ssh_config (~/.ssh/config default). 'Name' tag of instances are used as `Host` descriptions.

# How to use
### 1. Set 'Name' tag to your instances
eg. Tag 'app-server-1' as 'Name' to an instance i-xxxxx in us-west-1 region.

### 2. Write ~/.aws/credentials
```
# ~/.aws/credentials

[default]
aws_access_key_id=...
aws_secret_access_key=...

[myprofile]
aws_access_key_id=...
aws_secret_access_key=...
```

If you need more details about `~/.aws/credentials`, check [A New and Standardized Way to Manage Credentials in the AWS SDKs](http://blogs.aws.amazon.com/security/post/Tx3D6U6WSFGOK2H/A-New-and-Standardized-Way-to-Manage-Credentials-in-the-AWS-SDKs)

### 3. Install ec2ssh

```
$ gem install ec2ssh
```

### 4. Execute `ec2ssh init`

```
$ ec2ssh init
```

### 5. Edit `.ec2ssh`

```
$ vi ~/.ec2ssh
---
profiles 'default', 'myprofile'
regions 'us-east-1'

# Ignore unnamed instances
reject {|instance| !instance.tags['Name'] }

# You can use methods of AWS::EC2::Instance.
# See http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/EC2/Instance.html
host_line <<END
Host <%= tags['Name'] %>.<%= availability_zone %>
  HostName <%= dns_name || private_ip_address %>
END
```

### 6. Execute `ec2ssh update`

```
$ ec2ssh update
```
Then host-names of your instances are generated and wrote to .ssh/config

### 7. And you can ssh to your instances with your tagged name.

```
$ ssh app-server-1.us-east-1a
```

# Commands
```
$ ec2ssh help [TASK]  # Describe available tasks or one specific task
$ ec2ssh init         # Add ec2ssh mark to ssh_config
$ ec2ssh update       # Update ec2 hosts list in ssh_config
$ ec2ssh remove       # Remove ec2ssh mark from ssh_config
```

## Options
### --dotfile
Each command can use `--dotfile` option to set dotfile (.ec2ssh) path. `~/.ec2ssh` is default.

```
$ ec2ssh init --dotfile /path/to/ssh_config
```

# ssh_config and mark lines
`ec2ssh init` command inserts mark lines your `.ssh/config` such as:

```
### EC2SSH BEGIN ###
# Generated by ec2ssh http://github.com/mirakui/ec2ssh
# DO NOT edit this block!
# Updated Sun Dec 05 00:00:14 +0900 2010
### EC2SSH END ###
```

`ec2ssh update` command inserts 'Host' descriptions between 'BEGIN' line and 'END' line.

```
### EC2SSH BEGIN ###
# Generated by ec2ssh http://github.com/mirakui/ec2ssh
# DO NOT edit this block!
# Updated Sun Dec 05 00:00:14 +0900 2010

# section: default
Host app-server-1.us-west-1
  HostName ec2-xxx-xxx-xxx-xxx.us-west-1.compute.amazonaws.com
Host db-server-1.ap-southeast-1
  HostName ec2-xxx-xxx-xxx-xxx.ap-southeast-1.compute.amazonaws.com
    :
    :
### EC2SSH END ###
```

`ec2ssh remove` command removes the mark lines.

# How to upgrade from 1.x to 2.x
If you have used ec2ssh-1.x, it seems that you may not have '~/.ec2ssh'.
So you need execute `ec2ssh init` once to create `~/.ec2ssh`, and edit it as you like.

```
$ ec2ssh init
$ vi ~/.ec2ssh
```

# How to upgrade from 2.x to 3.x
Dotfile (`.ec2ssh`) format has been changed from YAML to Ruby DSL.

Don't panic and run `ec2ssh migrate` if you have ec2ssh-2.x styled dotfile.

```
$ ec2ssh migrate
```

This command converts your existing `.ec2ssh` file into 3.x style.

# Notice
`ec2ssh` command updates your `.ssh/config` file default. You should make a backup of it.

# Zsh completion support
Use `zsh/_ec2ssh`.

# License
Copyright (c) 2014 Issei Naruta. ec2ssh is released under the MIT license.
