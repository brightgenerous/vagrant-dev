# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = '2'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define :'vagrant-dev' do |config|
    config.vm.box = 'CentOS-6.5-x86_64-Minimal'
    config.vm.box_url = 'http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.5-x86_64-v20140504.box'

    config.vm.hostname = 'vagrant-dev'
    config.vm.network :private_network, ip: '33.33.33.100'
    { 3000 => 3000, 8081 => 8081, 8983 => 8983 }.each do |h, g|
      config.vm.network :forwarded_port, host: h, guest: g
    end
    config.ssh.forward_agent = true
    config.vm.provider :virtualbox do |vb|
      vb.customize [ 'modifyvm', :id, '--natdnsproxy1', 'off' ]
      vb.customize [ 'modifyvm', :id, '--natdnshostresolver1', 'off' ]
      vb.customize [ 'modifyvm', :id, '--memory', '2048', '--cpus', '2' ]
    end

    user, group = 'vagrant', 'vagrant'
    config.vm.provision :chef_solo do |chef|
      chef.custom_config_path = './Vagrantfile.chef'
      chef.cookbooks_path = [ './cookbooks', './vendor/cookbooks' ]
      chef.data_bags_path = 'data_bags'
      [
        'yum-repoforge',
        'git',
        'ntp',
        'curl::libcurl', 'imagemagick', 'libffi::devel',
        'rbenv-install-rubies',
        'mysql::server', 'mysql::client',
        'redisio', 'redisio::enable', 'chef-redis-commander',
        'java',
        'solr',
        'elasticsearch', 'elasticsearch::plugins',
        'td-agent-config',
        'zsh::chsh', 'screen', 'vim::source', 'neobundle', 'lv',
        'homesick::data_bag',
        'iptables::disabled'
      ].each do |recipe|
        chef.add_recipe recipe
      end
      chef.json = {
        users: [ user ], # for homesick.
        rbenv: {
          user: user,
          group: group
        },
        zsh: {
          users: [ user ]
        },
        neobundle: {
          repository: { protocol: 'https' },
          skip: { vimrc: true },
          home_dir: true,
          vim_home: '/home/vagrant/.vim',
          user: user,
          group: group
        },
        git: {
          source: true
        },
        rbenv_install_rubies: {
          global_version: '2.2.0',
          gems: [ :bundler, :homesick, { rails: { version: '4.2.0' } } ]
        },
        mysql: {
          version: '5.6',
          port: '3306',
          server_root_password: '',
          enable_utf8: 'utf8mb4'
        },
        redisio: {
          version: '2.8.19',
          servers: [ { port: '6379' } ]
        },
        chef_redis_commander: {
          nodejs: true
        },
        java: {
          jdk_version: '7'
        },
        solr: {
          version: '4.9.1',
          port: '8983',
          java_opts: {
            xms: '128M',
            xmx: '128M'
          }
        },
        elasticsearch: {
          version: '1.4.4',
          http: { port: '9200' },
          allocated_memory: '128m',
          plugins: {
           :'elasticsearch/elasticsearch-analysis-kuromoji' => {
             version: '2.4.2'
           }
          }
        },
        td_agent: {
          version: '2.1.5',
          includes: true,
          default_config: false
        },
        td_agent_config: {
          sources: [ {
            default_source: {
                type: 'forward',
              tag: '**',
              params: {
                port: '24224'
              }
            }
          } ],
          matches: [ {
            default_match: {
              type: 'file',
              tag: '**',
              params: {
                path: '/var/log/td-agent/forward.log'
              }
            }
          } ]
        }
      }
    end
  end
end
