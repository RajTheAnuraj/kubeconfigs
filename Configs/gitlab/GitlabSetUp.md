You can go to [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-gitlab-on-ubuntu-18-04]() or gitlabs own site for instructions


```
sudo apt update
sudo apt install ca-certificates curl openssh-server postfix
```

```
cd /tmp
curl -LO https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh
```

```
sudo bash /tmp/script.deb.sh
```

```
sudo apt install gitlab-ce
```

The above step is gonna take an eternity.  
once that is done we need to modify the config for us to enable the thing to be accessible from our network

```
sudo nano /etc/gitlab/gitlab.rb
```

Look for `external_url` section.  
Once you find it add the following.  This will expose the thing on given port and url
```
external_url 'https://my.server.com/'
nginx['listen_port'] = 8888
nginx['listen_https'] = false
```

once this is done go ahead and call gitlabs reconfigure

```
sudo gitlab-ctl reconfigure
```

The above step again takes an eternity.  

After this you can look at the output of the first gitlab-ee install for the root pwd
Or you can reset the password running below

```
sudo gitlab-rake "gitlab:password:reset"
```
This will ask for the user name and Password and Confirm password.  

Now you can use the new pwd and log in to the url:port you configured earlier.


If you need the repositories to be on a separate drive or something edit the above config file and search for git_data_dir. Uncomment the section and set your path and restart config