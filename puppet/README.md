Puppet
======

Setup
-----
```
rvm install ruby-2.5.8
gem install r10k
gem install puppet --version 6.3.0
```

Management
----------

Initial setup:
```
r10k deploy environment -v
cd /opt/puppet/code/environments/production
git checkout production
git config receive.denyCurrentBranch warn
```

Then add the following post-update hook to `.git/hooks/post-update` (`chmod +x`):
```
#!/usr/bin/env bash

# http://debuggable.com/posts/git-tip-auto-update-working-tree-via-post-receive-hook:49551efe-6414-4e86-aec6-544f4834cda3

# pwd == The .git directory of this repository.
cd ..
env -i git reset --hard
```

Developing
----------

1. Clone the [TurboPuppet](https://github.com/CFCC/TurboPuppet) repo locally.
2. Add a remote of the puppetmaster.
   ```
   git remote add puppetmaster grant@puppet.lab.campcomputer.com:/opt/puppet/code/environments/production
   ```
3. Push! `git push puppetmaster production`
