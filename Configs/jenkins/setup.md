Installing jenkins on Ubuntu

* Add jenkins debian repository
  *  first add the key to your system
```
  curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```
  * Then add a Jenkins apt repository entry
  ```
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
  
  ```
  * Install JENKINS NOW
  ```
  sudo apt-get update
  sudo apt-get install jenkins
  ```