# How to build Running Sushi for Pipedrive Chef servers.

### Dependencies
* [chef_diff](https://github.com/One-com/chef_diff)
* [slack-notifier](https://github.com/stevenosloan/slack-notifier)

## 1) Build and install tools

On a Chef server build and install needed tools:

```
# Create a directory for building
mkdir -p build; cd build
# Clone needed repos
git clone https://github.com/One-com/chef_diff.git
git clone git@github.com:pipedrive/running_sushi.git
# Build and install needed gems
cd chef_diff/ && /opt/chef/embedded/bin/gem build chef_diff.gemspec && /opt/chef/embedded/bin/gem install chef_diff-${VERSION_OF_GEM_BUILT}.gem && cd ..
cd running_sushi/ && /opt/chef/embedded/bin/gem build chef_deliver.gemspec && /opt/chef/embedded/bin/gem install chef_delivery-${VERSION_OF_GEM_BUILT}.gem && cd ..
/opt/chef/embedded/bin/gem install slack-notifier
```

## 2) Create config file

Create a config file for `running_sushi` (`chef-delivery`) in `/etc/chef/chef_delivery_config.rb` like:

```
# master_path - The top-level path for Running Sushi's work. Most other paths are relative to this. Default: /var/chef/chef_delivery_work
repo_url 'git@github.com:pipedrive/chef-repo.git'
reponame 'chef-repo'
pod_name 'eu-central-1'
user 'admin'
pem '/root/.chef/admin.pem'
chef_server_url "https://chef-fr.eu-central-1.pipedrive.net/organizations/pipedrive"
# client_path A directory to find clients in relative to reponame. Default: clients
# cookbook_paths - An array of directories that contain cookbooks relative to reponame. Default: ['cookbooks']
# databag_path - A directory to find databags in relative to reponame. Default: data_bags
# environment_path - A directory to find environments in relative to reponame. Default: environments
# node_path - A directory to find nodes in relative to reponame. Default: nodes
role_path 'roles_json'
role_local_path 'roles_local/json'
# user_path - A directory to find users in relative to reponame. Default: users
# rev_checkpoint - Name of the file to store the last-uploaded revision, relative to reponame. Default: chef_delivery_revision
# plugin_path - Path to plugin file. Default: /etc/chef_delivery_config_plugin.rb
slack_url 'https://hooks.slack.com/services/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'
```

Config parameters that need to be adjusted for each separate chef server are:
- pod_name - the name of the region Chef server is in
- chef_server_url - Full URL (with Chef organization) to the Chef server used

## 3) Schedule the execution

Running Sushi is triggered using `cron`.
Add the following line to Chef server `cron`:

```
# Running sushi
*/5 * * * * bash -c 'source /root/.bash_profile; chef-delivery -T;'
```

Cron execution logs are stored in `/var/log/syslog`
