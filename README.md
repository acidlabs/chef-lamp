# Chef-LAMP

Kitchen to setup an Ubuntu Server ready to roll with MySQL and Apache2.

## Requirements

* Ubuntu 10.04+

## Usage

To cook with this kitchen you must follow four easy steps.

### 1. Prepare your local working copy

```bash
git clone git://github.com/acidlabs/chef-lamp.git
cd chef-lamp
bundle install
bundle exec librarian-chef install
```

### 2. Prepare the servers you want to configure

We need to copy chef-solo to any server we’re going to setup. For each server, execute

```bash
bundle exec knife prepare [user]@[host] -p [port]
```

where

* *user* is a user in the server with sudo and an authorized key.
* *host* is the ip or host of the server.
* *port* is the port in which ssh is listening on the server.

### 3. Define the specs for each server

If you take a look at the *nodes* folder, you’re going to see files called [host].json, corresponding to the hosts or ips of the servers we previously prepared, plus a file called *localhost.json.example* which is, as its name suggests, and example.

The specs for each server needs to be defined in those files, and the structure is exactly the same as in the example.

For the very same reason, we’re going to exaplain the example for you to ride on your own pony later on.

```json
{
// This is the list of the recipes that are going to be cooked.
  "run_list": [
    "recipe[sudo]",
    "recipe[apt]",
    "recipe[build-essential]",
    "recipe[ohai]",
    "recipe[runit]",
    "recipe[git]",
    "recipe[apache2]",
    "recipe[mysql::server]",
    "recipe[chef-rails]"
  ],

// You must define who’s going to be the user(s) you’re going to use for deploy.
  "authorization": {
    "sudo": {
      "groups":       ["ubuntu"],
      "users":        ["ubuntu"],
      "passwordless": true
    }
  },

// You must define the username and password for MySQL.
  "mysql": {
    "server_root_password"  : "mysqlrootuserpasswordhere",
    "server_debian_password": "mysqlrootuserdebianwordhere",
    "server_repl_password"  : "mysqlrootuserreplwordhere"
  },

// You must define the server platform. See Apache2 cookbook attributes to know all configuration params.
  "apache": {
    "platform": "ubuntu"
  }

// Finally, declare all the system packages required by the services and gems you’re using in your apps.
 //"chef-rails": {
   //"packages": ["package_name", "package2_name"]
 //}
}
```

### 4. Virtual Hosts configurations

First, for each Apache2 Virtual Host, we have to create a cookbook.

// We create our cookbook with apachev2 license, markdown readme file format and specific path for the cookbook
```bash
bundle exec knife cookbook create my_vhost -c "My Name" -e "my@email.com" -l apachev2 -r md -o site-cookbooks/
```

We will use a template to set a Virtual Host, so we need to include the apache recipe in the previously created site cookbook.

```ruby
# sites-cookbook/example_vhost/recipes/default.rb
#
# Cookbook Name:: example_vhost
# Recipe:: default
#
# Copyright 2013, YOUR_COMPANY_NAME
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

include_recipe "apache2"

# See Apache2 cookbook/templates/default to know all the possible params and template examples.
web_app "example_vhost.com" do
  server_name "example_vhost.com"
  server_aliases ["www.example_vhost.com"] # Is an Array
  docroot "/var/www/example_vhost.com"
end
```

Finally, we must add each Virtual Host recipe to run_list in our node configuration.
In this case, our run list is:
```json
  "run_list": [
    "recipe[sudo]",
    "recipe[apt]",
    "recipe[build-essential]",
    "recipe[ohai]",
    "recipe[runit]",
    "recipe[git]",
    "recipe[apache2]",
    "recipe[mysql::server]",
    "recipe[chef-rails]",
    "recipe[example_vhost]"
    ],
```

### 5. Happy cooking

We’re now ready to cook. For each server you want to setup, execute

```bash
bundle exec knife cook [user]@[host] -p [port]
```