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
external_url 'https://my.server.com:8888'
nginx['listen_port'] = 8888
nginx['listen_https'] = false
```

Its also a good idea to store the repositories on a shared drive
Mount the drive to a folder and then edit the section in the gitlab.rb file to point the location
```
git_data_dirs({
  'default' => {
    'path' => '/mnt/hitachi/gitlab'
  },
})
```

* Enabling container registry
Modify the following settings in gitlab.rb file
```
registry_external_url 'https://registry.example.com' -- Set this to your root url ex: http://192.168.33.109
gitlab_rails['registry_path'] = "/var/opt/gitlab/gitlab-rails/shared/registry" -- Set to your path. /mnt/whatever


registry['enable'] = true
registry['debug_addr'] = "localhost:5051" -- doing this is very important since if there are start up errors on registry you can log on to server and run http://localhost:5051/debug/health.  This is only accessible from server. but give good idea about whats wrong


registry_nginx['enable'] = true
registry_nginx['listen_port'] = 5050
registry_nginx['listen_https'] = false  --This particular one has to be added
```

once this is done go ahead and call gitlabs reconfigure

```
sudo gitlab-ctl reconfigure
```

**Very Very important**
Something is not working look at logs
all kind of logs present in subdirectories of `/var/log/gitlab`
Mostly will be filesystem permission related. The below gives rights to all users. *not safe to do this in a public env*
```
sudo chmod -R a+rwX /mnt/hitachi 
```

* For the docker to login to http registry you need to enable insecure registry. Not recommended. But since its home lab with no external access its fine to enable. modify `/etc/docker/daemon.json` file and add your url to the `insecure-registries` collection
```
"insecure-registries" : ["192.168.86.109:5050"]
```

also restart docker engine to take effect
```
 sudo service docker restart 
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

# CI CD
Now you have to configure shared runners for your ci-cd. 
There are many ways thr runners can run. 
Sincw we are using single server I am planning to run everything including ci-cd on that same ubuntu machine. 

```
dpkg --print-architecture 
```
The above command will print the cpu arch.
Mine came as `amd64`

Run the following to download the oackage. 
Substitute the ${arch} with the amd64
```
curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_${arch}.deb"
```

Unpack and install
```
dpkg -i gitlab-runner_<arch>.deb
```

That will install the thing.
Now we need to register it with the gitlab so that gitlab can queue jobs on to this runner.
For this first go to gitlab --> menu --> admin
Under features card look for an option `Shared Runners`

Under *Set up a shared runner manually* section you can see the url and token value. This is important
Go back to shell and type
```
sudo gitlab-runner register
```

It will ask you to enter url and token. Copy paste it from gitlab screen

Give approriate name and tags

For runner executor type `shell` . I chose this because I have installed docker and dotnet cli on that machine. 
So it can run these programs from the shell

Complete the config , go back to gitlab and refresh.
You can see the new shared runner added in there.
Go to the runner and check *Run untagged jobs* options 

Now you have a runner which any of your project can use.


Now another thing you have to do is regarding user permission
gitlab runs the ci-cd under the user context gitlab-runner
Give this user necessaryt permissions to your folders where you want artifacts to be stored etc
Also add this user to docker group so that it can run docker command

The below prints all groups which the user belongs to
```
groups gitlab-runner
```

Run the following command to add the user to docker group
```
sudo usermod -aG docker gitlab-runner
```


Setting up the pipeline

When you create project you can create .gitlab-ci.yml file where all your steps are stored
Below is a sample

```
stages:
  - build
  - test
  - release

build:
  stage: build
  script:
    - dotnet build 

test:
  stage: test
  script:
    - dotnet test

release:
  stage: release
  only:
    - master
  artifacts:
    paths:
      - publish/
  script:
    - echo $USER
    - docker build -t aspnetapp:v5 App/.
    - docker save -o /home/git/docker/aspnetapp.tar aspnetapp:v5
```


# Installing Nuget 
* First Install Mono
```
sudo apt install gnupg ca-certificates
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
echo "deb https://download.mono-project.com/repo/ubuntu stable-focal main" | sudo tee /etc/apt/sources.list.d/mono-official-stable.list
sudo apt update
sudo apt install mono-devel
```

* Download the latest stable `nuget.exe` to `/usr/local/bin`
```
sudo curl -o /usr/local/bin/nuget.exe https://dist.nuget.org/win-x86-commandline/latest/nuget.exe
```

* Add symlink to /bin so it can be called nuget instead of /usr/local/bin/nuget.exe
```
sudo ln /usr/local/bin/nuget.exe /bin/nuget
```

* Now we could use the below in the cicd pipeline to push into the repository]
```
deploy:
  stage: deploy
  script:
    - dotnet pack -c Release
    - dotnet nuget add source "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/nuget/index.json" --name gitlab --username gitlab-ci-token --password $CI_JOB_TOKEN --store-password-in-clear-text
    - dotnet nuget push "bin/Release/*.nupkg" --source gitlab
  only:
    - main
```

*Might be a good idea to create a new project just to store all your nuget packages. That ways the packages are stored separate from project and there will be only one nuget source to be configured*

