Puppet vcsrepo git refresh commit ref POC
-----------------------------------------

# Install prerequisites

    $ apt-get install -y git puppet
    $ cd /etc/puppet/modules
    $ git clone https://github.com/openstack-infra/puppet-vcsrepo vcsrepo

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
      refreshonly => true,
    }

# Result of puppet runs

## ensure => latest, revision = 'master'

    $ puppet apply refresh-test.pp
    Notice: Compiled catalog for vcsrepo-test.openstacklocal in environment production in 0.04 seconds
    Warning: Found multiple default providers for vcsrepo: svn, bzr, cvs, hg, git; using svn
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: Creating repository from latest
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: created
    Notice: /Stage[main]/Main/Exec[askbot-install]: Triggered 'refresh' from 1 events
    Notice: Finished catalog run in 9.96 seconds

    $ puppet apply refresh-test.pp
    Notice: Compiled catalog for vcsrepo-test.openstacklocal in environment production in 0.04 seconds
    Warning: Found multiple default providers for vcsrepo: svn, bzr, cvs, hg, git; using svn
    Notice: Finished catalog run in 0.66 seconds

## ensure => latest, revision = '06be25e5d1aa013a9a201a92db4b35b1ee1f3d32'

    $ puppet apply refresh-test.pp
    Notice: Compiled catalog for vcsrepo-test.openstacklocal in environment production in 0.03 seconds
    Warning: Found multiple default providers for vcsrepo: svn, bzr, cvs, hg, git; using svn
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: Creating repository from latest
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: created
    Notice: /Stage[main]/Main/Exec[askbot-install]: Triggered 'refresh' from 1 events
    Notice: Finished catalog run in 10.36 seconds

    $ puppet apply refresh-test.pp
    Notice: Compiled catalog for vcsrepo-test.openstacklocal in environment production in 0.03 seconds
    Warning: Found multiple default providers for vcsrepo: svn, bzr, cvs, hg, git; using svn
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: 06be25e5d1aa013a9a201a92db4b35b1ee1f3d32
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: Updating to latest '06be25e5d1aa013a9a201a92db4b35b1ee1f3d32' revision
    Notice: /Stage[main]/Main/Vcsrepo[/srv/askbot]/ensure: ensure changed 'present' to 'latest'
    Notice: /Stage[main]/Main/Exec[askbot-install]: Triggered 'refresh' from 1 events
    Notice: Finished catalog run in 1.83 seconds
