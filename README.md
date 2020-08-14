# Challenge 5 (Stake_Wars) CI/CD Pipline
----------------------------------------
Published on: August 15 2020
## 

This document provides one way to automatically deploy nearcore using a CI / CD pipeline using Jenkins (testnet):

* Install curl:

 ``` 
 sudo apt install curl 
 ```


* Install build dependencies:
 ``` 
 apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm
 ``` 
* Install  Jenkins (https://www.jenkins.io/doc/book/installing/#debianubuntu) :
 ``` 
   wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
     /etc/apt/sources.list.d/jenkins.list'
   sudo apt-get update
   sudo apt-get install jenkins
 ``` 

* Go to jenkins user:
 ```
 sudo su jenkins
```

* Install rustup and cargo (rust compiler):
```
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   during installation it will ask which version to install. you need to press 1 (default)
```
* Set up auto build in jenkins:

   * create item -> task with free configuration
   
   * in the section "Source Code Management":
   
     register the repository https://github.com/nearprotocol/nearcore.git
     
     ```set refspec + refs / tags / *: refs / remotes / origin / tags / * ``` so that the git only downloads tags
     
   * set the value in Branches to build:. ```* / tags /.* rc. * ``` - this is a regex that matches only tags and only those containing the word "rc" (fot betanet - "beta"), which was required in the task
     
   * in the "Build triggers" block:   
      check the box to poll the SCM about changes: Jenkins will check the git repository at configured intervals and start building if it sees that something new has appeared in the repository
      set in the schedule ```H / 15 * * * * ```(i.e. poll every 15 minutes)
   * in the "Assembly" block:
   
   
     * add the step "execute shell command": 
     ```
     cargo build -p neard --release (compile the binary, everything is compiled, you can find it in the / var / lib / jenkins / workspace / nearcore_beta / target / release / folder on the server)
     ```
     * add the step "execute shell command": 
     ```
     python3 ./scripts/parallel_run_tests.py (run tests, crashed several times with the error memory exhausted)
     ```
     * add "execute shell" step: 
     ```
     nearup testnet --nodocker --binary-path ./target/release
     ```
* You can check the changes in the build steps through manual activation of the build: you open the nearcore_beta project and click "Build now". After a couple of seconds, a new assembly will appear on the left in the build history, click directly on it and open the console output. All command logs will be there. 
