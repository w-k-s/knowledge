# Setting-up Jenkins

## 1. Configuring Git

In order for Jenkins to checkout code using git, git needs to be installed on the Jenkins server.

```sh
sudo apt-get install git
```

The above command will install Git in `/usr/bin/git`. This path needs to be set on Jenkins. Go to `Manage Jenkins > Global Tool Configuration"`. In the section titled "Git", provide the path at which git is installed in the field titled "Path to Git Executable". Then save these configurations.

[Reference](https://stackoverflow.com/a/11124264/821110)

## Troubleshooting

### 1. Found an incorrect Java version

> admin@ip-172-26-11-125:~$ systemctl status jenkins.service
> ● jenkins.service - LSB: Start Jenkins at boot time
>    Loaded: loaded (/etc/init.d/jenkins; generated)
>   Active: failed (Result: exit-code) since Thu 2021-01-07 17:10:30 UTC; 4s ago
>     Docs: man:systemd-sysv-generator(8)
>  Process: 1143 ExecStart=/etc/init.d/jenkins start (code=exited, status=1/FAILURE)
>
> Jan 07 17:10:30 ip-172-26-11-125 systemd[1]: Starting LSB: Start Jenkins at boot time...
> Jan 07 17:10:30 ip-172-26-11-125 jenkins[1143]: Found an incorrect Java version
> Jan 07 17:10:30 ip-172-26-11-125 jenkins[1143]: Java version found:
> Jan 07 17:10:30 ip-172-26-11-125 jenkins[1143]: openjdk version "11.0.9.1" 2020-11-04
> Jan 07 17:10:30 ip-172-26-11-125 jenkins[1143]: OpenJDK Runtime Environment (build 11.0.9.1+1-post-Debian-1deb10u2)
> Jan 07 17:10:30 ip-172-26-11-125 jenkins[1143]: OpenJDK 64-Bit Server VM (build 11.0.9.1+1-post-Debian-1deb10u2, mixed mode, sharing)
> Jan 07 17:10:30 ip-172-26-11-125 jenkins[1143]: Aborting
> Jan 07 17:10:30 ip-172-26-11-125 systemd[1]: jenkins.service: Control process exited, code=exited, status=1/FAILURE
> Jan 07 17:10:30 ip-172-26-11-125 systemd[1]: jenkins.service: Failed with result 'exit-code'.
> Jan 07 17:10:30 ip-172-26-11-125 systemd[1]: Failed to start LSB: Start Jenkins at boot time.



This error happens when trying to run Jenkins with Java 11. The solution to this problem is to fix the regular expression in the file `/etc/init.d/jenkins`:

```sh
sudo vim /etc/init.d/jenkins
```

Search for the line that looks like:
```sh
JAVA_ALLOWED_VERSIONS=( "18" "110" )
# Work out the JAVA version we are working with:
JAVA_VERSION=$($JAVA -version 2>&1 | sed -n ';s/.* version "\(.*\)\.\(.*\)\..*".*/\1\2/p;')
```

Change the last line to:
```sh
JAVA_VERSION=$($JAVA -version 2>&1 | sed -n ';s/.* version "\([0-9]*\)\.\([0-9]*\)\..*".*/\1\2/p;')
```

and then restart Jenkins:

```sh
sudo systemctl start jenkins
```

[Reference](https://askubuntu.com/questions/1139046/jenkins-error-incorrect-java-version-for-java11-after-removing-java11-and-ins)