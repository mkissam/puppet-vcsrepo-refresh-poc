VCSRepo Workaround with conditional exec implementation
-------------------------------------------------------

# Test manifest with refresh event

refresh-test.pp:

    $askbot_revision = '06be25e5d1aa013a9a201a92db4b35b1ee1f3d32'
    # $askbot_revision = 'master'

    vcsrepo { '/srv/askbot':
      ensure   => latest,
      provider => git,
      revision => $askbot_revision,
      source   => 'https://github.com/ASKBOT/askbot-devel.git',
    }

    exec { 'askbot-install':
      path        => [ '/bin', '/sbin' , '/usr/bin', '/usr/sbin', '/usr/local/bin' ],
      command     => '/bin/echo "Exec triggered."',
      logoutput   => on_failure,
      subscribe   => Vcsrepo["/srv/askbot"],
      unless      => 'cmp /srv/askbot/.git/HEAD /srv/HEAD.latest',
      refreshonly => true,
    }

    exec { 'save-git-run-state':
      path        => [ '/bin', '/sbin' , '/usr/bin', '/usr/sbin', '/usr/local/bin' ],
      command     => 'cp /srv/askbot/.git/HEAD /srv/HEAD.latest',
      logoutput   => on_failure,
      subscribe   => Vcsrepo["/srv/askbot"],
      unless      => 'cmp /srv/askbot/.git/HEAD /srv/HEAD.latest',
      refreshonly => true,
      require     => Exec['askbot-install'],
    }

# Result of Puppet runs

## first run

    $ puppet apply refresh-test.pp
    Notice: Compiled catalog for vcsrepo-poc.openstacklocal in environment production in 0.04 seconds
    Warning: Found multiple default providers for vcsrepo: svn, bzr, cvs, hg, git; using svn
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: Creating repository from latest
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: created
    Notice: /Stage[main]/Main/Exec[askbot-install]: Triggered 'refresh' from 1 events
    Notice: /Stage[main]/Main/Exec[save-git-run-state]: Triggered 'refresh' from 1 events
    Notice: Finished catalog run in 10.79 seconds

## second run

    $ puppet apply refresh-test.pp
    Notice: Compiled catalog for vcsrepo-poc.openstacklocal in environment production in 0.04 seconds
    Warning: Found multiple default providers for vcsrepo: svn, bzr, cvs, hg, git; using svn
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: 06be25e5d1aa013a9a201a92db4b35b1ee1f3d32
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: Updating to latest '06be25e5d1aa013a9a201a92db4b35b1ee1f3d32' revision
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: ensure changed 'present' to 'latest'
    Notice: /Stage[main]/Main/Exec[askbot-install]: Triggered 'refresh' from 1 events
    Notice: /Stage[main]/Main/Exec[save-git-run-state]: Triggered 'refresh' from 1 events
    Notice: Finished catalog run in 2.09 seconds

`Notice` the second run silently skips the askbot-install command execution, meanwhile
the report still include the refresh event related logs.