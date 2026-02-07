<!---
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

# Hadoop Release Support

This project helps create validate hadoop release candidates

https://github.com/apache/hadoop-release-support

It has an Apache Ant `build.xml` file to help with preparing the release,
validating gpg signatures, creating release messages and other things.

There is a maven `pom.xml` file. This is used to validate the dependencies
from staging repositories as well as run some basic tests to validate
the classpath.

# Prerequisites

Installed applications/platforms

* Branch 3.4: Java 8. Later Java releases are valid for validation too (and required for some projects)
* trunk: Java 17
* Apache Ant.
* Apache Maven 3.9.12
* gpg
* git
* subversion (for staging artifacts; not needed for validation)

### Ant setup

To use the scp/ssh commands we need the jsch jar on the classpath.
```
ant -diagnostics
```

BAD
```
-------------------------------------------
 Tasks availability
-------------------------------------------
sshsession : Missing dependency com.jcraft.jsch.Logger
scp : Missing dependency com.jcraft.jsch.Logger
sshexec : Missing dependency com.jcraft.jsch.Logger
```

Here are the apt-get commands to set up a raspberry pi for the arm validation
```bash
apt-get install openjdk-17-jdk
apt-get install ant libjsch-java
apt-get install gpgv
apt-get install maven
apt-get install subversion
```


# Files

###  `build.xml`

It has an Apache Ant `build.xml` file to help with preparing the release,
validating gpg signatures, creating release messages and other things.

###  `pom.xml`

There is a maven `pom.xml` file. This is used to validate the dependencies
from staging repositories as well as run some basic tests to validate
the classpath.


###  `build.properties`

This is an optional property file which contains all user-specific customizations
and options to assist in the release process.

This file is *not* SCM-managed (it is explicitly ignored).

It is read before all other property files are read/ant properties
set, so can override any subsequent declarations.


### Release index file: `release.properties`

This is a single-entry property file which provides a relative
path to the latest release being worked on in this branch.

1. It is SCM-managed.
2. It is read after `build.properties`

```properties
release.version=3.4.1
```

Ant uses this to to set the property `release.info.file` to the path
`src/releases/release-info-${release.version}.properties`

```properties
release.info.file=src/releases/release-info-3.4.1.properties
```

This is then loaded, with the build failing if it is not found.

### Release info files `src/releases/release-info-*.properties`

Definition files of base properties for the active RC.

* SCM-managed
* Defines properties which are common to everyone building/validating
  an RC.

As an example, here is the value `src/releases/release-info-3.4.0.properties` for the RC2
release candidate

```properties
hadoop.version=3.4.0
rc=RC2
previous.version=3.3.6
release.branch=3.4
git.commit.id=88fbe62f27e

jira.id=HADOOP-19018
jira.title=Release 3.4.0

amd.src.dir=https://dist.apache.org/repos/dist/dev/hadoop/hadoop-3.4.0-RC2
arm.src.dir=${amd.src.dir}
http.source=${amd.src.dir}
asf.staging.url=https://repository.apache.org/content/repositories/orgapachehadoop-1402

cloudstore.profile=sdk2
```



# workflow for preparing an RC

## Build the RC

Build the RC using the docker process on whichever host is set to do it
using following doc https://cwiki.apache.org/confluence/display/HADOOP2/HowToRelease

Start EC2 Ubuntu 22 instance

Create a local user before anything else

```bash
groupadd -g 1024 mthakur
useradd -g 1024 -u 1024 -m mthakur
newgrp docker
usermod -aG docker mthakur
service docker restart
su - mthakur
```

Install java
```bash
sudo apt install openjdk-8-jre-headless
java -version
```

Install maven (only needed on branches without ./mvnw to do this .)
```bash
wget https://dlcdn.apache.org/maven/maven-3/3.9.11/binaries/apache-maven-3.9.11-bin.tar.gz
tar -xvf apache-maven-3.9.11-bin.tar.gz
mv apache-maven-3.9.11 /opt/
```

Setup maven home in .profile 
```bash
export M2_HOME="/opt/apache-maven-3.9.11/"
PATH="$M2_HOME/bin:$PATH"
export PATH
source .profile
```

Create ~/.m2/settings.xml file as mentioned in https://cwiki.apache.org/confluence/display/HADOOP2/HowToRelease

Install docker

https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce
sudo systemctl status docker
```

Download hadoop git module.
```bash
git clone https://github.com/apache/hadoop.git
```

Setup gpg signing in remote EC2 host.

On local Mac, export PGP key:

```bash

gpg --export --armor > gpg_public

gpg --export-secret-keys "Mukund Thakur" > private.key
```

Upload the file gpg_public to the Linux VM

On the Linux VM, import PGP key:

```bash
gpg --import gpg_public
gpg --import private.key
````

Follow rest of process as mentioned in above HowToRelease doc.

```sh
git clone https://github.com/apache/hadoop.git
cd hadoop
git checkout --track origin/branch-3.4.3

# for the arm buld: dev-support/bin/create-release --docker --dockercache
dev-support/bin/create-release --asfrelease --docker --dockercache
```


### Create a `src/releases/release-X.Y.X.properties` file

Create a new release properties file, using an existing one as a template.
Update as a appropriate.

### Update `release.properties`

Update the value of `release.version` in `release.properties` to
declare the release version. This is used to determine the specific release properties
file for that version.

```properties
release.version=X.Y.Z
```

#### Switching to a new release on the command line

You can switch to a new release on the command line; this is needed when
validating PRs.

 ```bash
ant -Drelease.version=3.4.1
 ```

### create a `build.properties` file for your local build setup

```properties

# info for copying down the RC from the build host
scp.hostname=stevel-ubuntu
scp.user=stevel
scp.hadoop.dir=hadoop

# SVN managed staging dir
staging.dir=/Users/stevel/hadoop/release/staging

# where various modules live for build and test
spark.dir=/Users/stevel/dev/spark
cloud-examples.dir=/Users/stevel/dev/sparkwork/cloud-integration/cloud-examples
cloud.test.configuration.file=/Users/stevel/dev/config/test-configs/s3a.xml
bigdata-interop.dir=/Users/stevel/dev/gcs/bigdata-interop
hboss.dir=/Users/stevel/dev/hbasework/hbase-filesystem
cloudstore.dir=/Users/stevel/dev/cloudstore
fs-api-shim.dir=/Users/stevel/dev/Formats/fs-api-shim/
```

### Clean up first

Clean up all the build files, including any remote downloads in the `downloads/`
dir.
And then purge all artifacts of that release from maven.
This is critical when validating downstream project builds.

```bash
ant clean mvn-purge
```

Tip: look at the output to make sure it is cleaning the artifacts from the release you intend to validate.


### SCP the RC down to `target/incoming`

This will take a while! look in target/incoming for progress

```bash
ant scp-artifacts
```

### Copy to the release dir

Copies the files from `downloads/incoming/artifacts` to `downloads/hadoop-$version-$rc`'

```bash
ant copy-scp-artifacts release.dir.check
```

The `release.dir.check` target just lists the directory.

### Sidenote: lean binary tarballs

The normal `binary tar.gz` file was historically huge because they contained a version of the AWS v2 SDK `bundle.jar`
file which has been validated with the hadoop-aws module and the S3A connector which was built against it.

This is a really big file because it includes all the "shaded" dependencies as well as client libraries
to talk with many unused AWS services up to and including scheduling satellite downlink time.

We shipped the full bundle jar as it allows Hadoop and its downstream applications to be isolated from
the choice of JAR dependencies in the AWS SDK. That is: it ensures a classpath that works out the box
and stop having to upgrade on a schedule determined by maintains the AWS SDK pom files.


It does make for big images and that has some negative consequences.
* More data to download when installing Hadoop.
* More space is required in the fileystem of any host into which it is installed.
* Slower times to launch docker containers if installing the binary tar as part of the container launch
  process.
* Larger container images if preinstalled.

A "lean" tar.gz was built by stripping out the AWS SDK jar and signing the new tarball.

The "lean" `binary tar.gz` files eliminated these negative issues by being
a variant of the normal x86 binary distribution with the relevant AWS SDK JAR removed.


Since Hadoop 3.4.3 the release binaries are automatically lean, an explicit build option of `-Dhadoop-aws-package` is needed
to bundle the AWS JAR.
The bundle.jar must now be added to `share/hadoop/common/lib`, and any version later than that of the release
can be added for automatic inclusion in the classpath.

The specific AWS SDK version qualified with is still the only "safe" version -its version number must be
included in the release announcement.

Instructions on this are included in the release announcement


### Building Arm64 binaries

Arm64 binaries must be created on an arm docker image.
They can be built locally or remotely (cloud server, raspberry pi5, etc.)

Do not use the `--asfrelease` option as this stages the JARs.
Instead use the explicit `--deploy --native --sign` options.

The arm process is one of
1. Create the full set of artifacts on an arm machine (macbook, cloud vm, ...).
   Based on our experience, doing this on a clean EC2 Ubuntu VM is more reliable than any local laptop
2. Use the ant build to copy and rename the `.tar.gz` with the native binaries only
3. Create a new `.asc `file.
4. Generate new `.sha512` checksum file containing the new name.
   Renaming the old file is insufficient.
5. Move these files into the `downloads/release/$RC` dir

To perform these stages, you need a clean directory of the same
hadoop commit ID as for the x86 release.

#### Local Arm build

In `build.properties` declare its location

```properties
arm.hadoop.dir=/Users/stevel/hadoop/release/hadoop
```

In that dir, create the release using command below. If signing fails in your ARM docker
container, you can skip signing by removing `--sign` option. The signing happens in the
next step if `ant arm.release` process after this.

create the release.

```bash
time dev-support/bin/create-release --docker --dockercache --native --sign
```

Leaving out the `-deploy` option keeps the artifacts out of nexus

*Important* make sure there is no duplicate staged hadoop repo in nexus.
If there is: drop and restart the x86 release process to make sure it is the one published


```bash

# copy the artifacts to this project's target/ dir, renaming
ant arm.copy.artifacts
# sign artifacts then move to the shared RC dir alongside the x86 artifacts
ant arm.sign.artifacts release.dir.check
```

#### Arm remote build and scp download

Create the remote build on an arm server.

```bash
time dev-support/bin/create-release --docker --dockercache --native --sign
```

| name                 | value                           |
|----------------------|---------------------------------|
| `arm.scp.hostname`   | hostname of arm server          |
| `arm.scp.user`       | username of arm server          |
| `arm.scp.hadoop.dir` | path under user homedir         |

Download the artifacts

```bash
ant arm.scp-artifacts
```
This downloads the artifacts to `downloads/arm/incoming`. 

Copy and rename the binary tar file.

```bash
ant arm.scp.copy.artifacts
```

#### arm signing

```bash
ant arm.sign.artifacts
```

Make sure that the log shows the GPG key used and that it matches that used for the rest
of the build.

```
[gpg] gpg: using "38237EE425050285077DB57AD22CF846DBB162A0" as default secret key for signing
```
### Publishing the RC

Publish the RC by copying it to a staging location in the hadoop SVN repository.

When committed to subversion it will be uploaded and accessible via a
https://svn.apache.org URL.


This makes it visible to others via the apache svn site, but it
is not mirrored yet.

When the RC is released, an `svn move` operation can promote it
directly to the release directory, from where it will be served at mirror locations.


*do this after preparing the arm64 binaries*

Final review the release files to make sure the -aarch64.tar.gz is present along with the rest,
and that everything is signed and checksummed.

```bash
ant release.dir.check
```

```
release.dir.check:
     [echo] release.dir=/home/stevel/Projects/client-validator/downloads/hadoop-3.4.3-RC0
        [x] total 2179608
        [x] -rw-r--r--@ 1 stevel  staff      10667 Jan 27 19:54 CHANGELOG.md
        [x] -rw-r--r--@ 1 stevel  staff        833 Jan 27 19:54 CHANGELOG.md.asc
        [x] -rw-r--r--@ 1 stevel  staff        153 Jan 27 19:54 CHANGELOG.md.sha512
        [x] -rw-r--r--@ 1 stevel  staff  511380916 Jan 27 20:02 hadoop-3.4.3-aarch64.tar.gz
        [x] -rw-r--r--@ 1 stevel  staff        833 Jan 27 20:02 hadoop-3.4.3-aarch64.tar.gz.asc
        [x] -rw-r--r--@ 1 stevel  staff        168 Jan 27 20:02 hadoop-3.4.3-aarch64.tar.gz.sha512
        [x] -rw-r--r--@ 1 stevel  staff    2302464 Jan 27 19:54 hadoop-3.4.3-rat.txt
        [x] -rw-r--r--@ 1 stevel  staff        833 Jan 27 19:54 hadoop-3.4.3-rat.txt.asc
        [x] -rw-r--r--@ 1 stevel  staff        161 Jan 27 19:54 hadoop-3.4.3-rat.txt.sha512
        [x] -rw-r--r--@ 1 stevel  staff   42682959 Jan 27 19:54 hadoop-3.4.3-site.tar.gz
        [x] -rw-r--r--@ 1 stevel  staff        833 Jan 27 19:54 hadoop-3.4.3-site.tar.gz.asc
        [x] -rw-r--r--@ 1 stevel  staff        165 Jan 27 19:54 hadoop-3.4.3-site.tar.gz.sha512
        [x] -rw-r--r--@ 1 stevel  staff   39418511 Jan 27 19:54 hadoop-3.4.3-src.tar.gz
        [x] -rw-r--r--@ 1 stevel  staff        833 Jan 27 19:54 hadoop-3.4.3-src.tar.gz.asc
        [x] -rw-r--r--@ 1 stevel  staff        164 Jan 27 19:54 hadoop-3.4.3-src.tar.gz.sha512
        [x] -rw-r--r--@ 1 stevel  staff  509684174 Jan 27 19:54 hadoop-3.4.3.tar.gz
        [x] -rw-r--r--@ 1 stevel  staff        833 Jan 27 19:54 hadoop-3.4.3.tar.gz.asc
        [x] -rw-r--r--@ 1 stevel  staff        160 Jan 27 19:54 hadoop-3.4.3.tar.gz.sha512
        [x] -rw-r--r--@ 1 stevel  staff       3495 Jan 27 19:54 RELEASENOTES.md
        [x] -rw-r--r--@ 1 stevel  staff        833 Jan 27 19:54 RELEASENOTES.md.asc
        [x] -rw-r--r--@ 1 stevel  staff        156 Jan 27 19:54 RELEASENOTES.md.sha512
```

Now stage the files, first by copying the dir of release artifacts
into the svn-mananaged location
```bash
ant stage
```

### In the staging svn repo, update, add and commit the work

This can take a while...exit any VPN for extra speed.


```bash
ant stage-to-svn
```

Manual
```bash
cd $stagingdir
svn update
svn add <RC directory name>
svn commit
```


### Tag the rc and push to github

This isn't automated as it needs to be done in the source tree.

The ant `print-tag-command` prints the command needed to create and sign
a tag.

```bash
ant print-tag-command
```
Which lists commands like
```
print-tag-command:
[echo] git.commit.id=56b832dfd5
[echo] 
[echo]       # command to tag the commit
[echo]       git tag -s release-3.4.3-RC0 -m "Release candidate 3.4.3-RC0" 56b832dfd5
[echo] 
[echo]       # how to verify it
[echo]       git tag -v release-3.4.3-RC0
[echo] 
[echo]       # how to view the log to make sure it really is the right commit
[echo]       git log tags/release-3.4.3-RC0
[echo] 
[echo]       # how to push to apache
[echo]       git push apache release-3.4.3-RC0
[echo] 
[echo]       # if needed, how to delete locally
[echo]       git tag -d release-3.4.3-RC0
[echo] 
[echo]       # if needed, how to delete it from apache
[echo]       git push --delete apache release-3.4.3-RC0
[echo] 
[echo]       # tagging the final release
[echo]       git tag -s rel/release-3.4.3 -m "HADOOP-19770. Hadoop 3.4.3 release"
[echo]       git push origin rel/release-3.4.3
[echo]   
```
From the output, go through the steps to tag, verify, view and then push

### Prepare the maven repository

1. Go to [https://repository.apache.org/#stagingRepositories](https://repository.apache.org/#stagingRepositories)
2. Find the hadoop repo for the RC
3. "close" it and wait for that to go through.
   (note: this is a little icon above the lists of repos)

### Generate the RC vote email

Review/update template message in `src/text/vote.txt`.
All ant properties referenced will be expanded if set.

```bash
ant vote-message
```

The message is printed and saved to the file `target/vote.txt`
    
*do not send it until you have validated the URLs resolve*

Now wait for the votes to come in. This is a good time to
repeat all the testing of downstream projects, this time
validating the staged artifacts, rather than any build
locally.

# How to download and build a staged release candidate

This project can be used to download and validated a release created by other people,
downloading the staged artifacts and validating their signatures before
executing some (minimal) commands.

This relies on the relevant `release-info-` file declaring the URL to download the artifacts from, and the maven staging repository.


```properties
amd.src.dir=https://dist.apache.org/repos/dist/dev/hadoop/hadoop-${hadoop.version}-RC${rc}/
```

### (Obsolete) Choose full versus lean downloads

The property `category` controls what suffix to use when downloading artifacts.
The default value, "", pulls in the full binaries.
If set to `-lean` then lean artifacts are downloaded and validated.
(_note: this is obsolete but retained in case it is needed for arm64 validation_)

```
category=-lean
```

### Targets of Relevance

| target                  | action                                                     |
|-------------------------|------------------------------------------------------------|
| `release.fetch.http`    | fetch artifacts                                            |
| `gpg.keys`              | import the hadoop KEYS                                     |
| `gpg.verify`            | verify the signature of the retrieved artifacts            |
| `release.dir.check`     | verify release dir exists                                  |
| `release.src.untar`     | untar retrieved artifacts                                  |
| `release.src.build`     | build the source; call `release.src.untar` first           |
| `release.src.test`      | build and test the source; call `release.src.untar` first  |
| `release.bin.untar`     | untar the binary file                                      |
| `release.bin.commands`  | execute a series of commands against the untarred binaries |
| `release.site.untar`    | untar the downloaded site artifact                         |
| `release.site.validate` | perform minimal validation of the site.                    |


Set `check.native.binaries` to false to skip native binary checks on platforms without them

### Download the Staged RC files from the Apache http servers

Downloads under `downloads/incoming`
```bash
ant release.fetch.http
```


### Verify GPG signatures

```bash
ant gpg.keys gpg.verify
```
This will import all the KEYS from 
[https://downloads.apache.org/hadoop/common/KEYS](https://downloads.apache.org/hadoop/common/KEYS),
then verify the signature of each downloaded file.

If you don't yet trust the key of whoever signed the release then
1. Refresh the keys from the OpenPGP server, to see
   if they've been signed by others.

        gpg --refresh-keys        

2. Perform whatever key verification you can and sign the key that
   level -ideally push up the signature to the servers.

### Untar source and build

This puts the built artifacts into the local maven repo so
do not do this while building/testing downstream projects
*and call `ant mvn-purge` after*

```bash
ant release.src.untar release.src.build
```

This build does not attempt to build the native binaries.
The `Pnative` profile can be enabled (or any other maven arguments)
in by declaring them in the property `source.compile.maven.args`

```properties
source.compile.maven.args=-Pnative
```
These are added at the end of the hard-coded arguments (`ant clean install -DskipTests`)

Testing is also possible through the target `release.src.test`

```bash
ant release.src.test
```
Again, the options set in `source.compile.maven.args` are passed down.

These targets are simply invoking maven in the source subdirectory
of `downloads/untar/source`, for example `downloads/untar/source/hadoop-3.4.1-src`


Do remember to purge the locally generated artifacts from your maven repository
```bash
ant mvn-purge
```

### Untar site and validate.


```bash
ant release.site.untar release.site.validate
```
Validation is pretty minimal; it just looks for the existence
of index.html files in the site root and under api/.

### Untar binary release

Untar the (already downloaded) binary tar to `bin/hadoop fs -ls $BUCKET/
`

```bash
ant release.bin.untar
```

Once expanded, the binary commands can be tested


```bash
ant release.bin.commands
```

This will fail on a platform where the native binaries don't load,
unless the `hadoop checknative` command has been disabled.

This can be done in `build.properties`

```properties
check.native.binaries=false
```

```bash
ant release.bin.commands -Dcheck.native.binaries=false
```

If `check.native.binaries` is false, the `bin/hadoop checknative`
is still executed, with the outcome printed (reporting a failure if
the binaries are not present).

The ant build itself will succeed, even if the `checknative` command reports a failure.

## Cloud connector integration tests

To test cloud connectors you need the relevant credentials copied into place into their
`src/test/resources` subdirectory, as covered in the appropriate documentation for each component.

The location of this file must be defined in the property `auth-keys.xml`.

```properties
auth-keys.xml=/home/alice/private/xml/auth-keys.xml
```


## Testing ARM binaries

There are ARM variants of the commands to fetch and validate the ARM binaries.

| target                  | action                                                     |
|-------------------------|------------------------------------------------------------|
| `release.fetch.arm`     | fetch ARM artifacts                                        |
| `gpg.arm.verify`        | verify ARM artifacts                                       |
| `release.arm.untar`     | untar the ARM binary file                                  |
| `release.arm.commands`  | execute commands against the ARM binaries                  |

```bash
# untars the `-aarch64.tar.gz` binary
ant release.fetch.arm gpg.arm.verify
ant release.arm.untar
ant release.arm.commands
```

# Testing on a remote server

The way to do this is to clone this `hadoop-release-support`
repository to the remote server and run the validation
commands there.

```sh
git clone https://github.com/apache/hadoop-release-support.git
```

# Building and testing projects from the staged maven artifacts

A lot of the targets build maven projects from the staged maven artifacts.

For this to work:

1. Check out the relevant projects in your local system.
2. Set their location in the `build.properties` file
3. Make sure that the branch checked out is the one you want to build.
   This matters for anyone who works on those other projects
   on their own branches.
4. Some projects need java11 or later. 

Some of these builds/tests are slow, but they can all be executed in parallel unless
you are actually trying to transitively build components, such as run spark tests with
the parquet artifact you build with the RC.
If you find yourself doing this: you've just become a CI system without the automation.

## Purge any existing artifacts from the maven repository

First, purge your maven repository of all `hadoop-` JAR files of the
pending release version

```bash
ant mvn-purge
```

## Execute a maven test run

Download the artifacts from maven staging repositories and compile/test a minimal application

```bash
mvn clean
ant mvn-test
```

Note: setting up on linux needs a number of dependencies installed (at least on an arm64 ubuntu system):

```bash
sudo apt-get install libssl-dev openssl subversion maven gpgv openjdk-17-jdk libjsch-java ant ant-optional ant-contrib cmake

# add to .bashrc
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-arm64/
```

## Validating maven dependencies

To see the dependencies of the maven project:
```bash
ant mvn-validate-dependencies
```

This saves the output to the file `target/mvndeps.txt` and explicitly
checks for some known "forbidden" artifacts that must not be exported
as transitive dependencies.

Review this to make sure there are no unexpected artifacts coming in.

## Build and test Cloudstore diagnostics

[cloudstore](https://github.com/steveloughran/cloudstore).

```bash
ant cloudstore.build
```


## Build and test Google GCS

[Big Data Interop](https://github.com/GoogleCloudPlatform/bigdata-interop).

* This is java 11+ only.

Ideally, you should run the tests, or even better, run them before the RC is up for review, so as to identify which failures are actually regressions.

### Building the GCS library
Do this only if you aren't running the tests.

```bash
ant gcs.build
```

### Testing the GCS library

Requires the source tree to be set up for test runs, including login credentials.

```bash
ant gcs.test
```

## Build Apache Spark

Validates hadoop client artifacts; the cloud tests cover hadoop cloud storage clients.

```bash
ant spark.build
```

To view the `hadoop-cloud` dependencies

```bash
ant spark.hadoop-cloud.dependencies
```

Review this to look for conflict.


To run the `hadoop-cloud` tests

```bash
ant spark.hadoop-cloud.test
```

A full spark test run takes so long that CI infrastructure should be used.

### Spark cloud integration tests

Then followup cloud integration tests if you are set up to build.
Spark itself does not include any integration tests of the object store connectors.
This independent module tests the s3a, gcs and abfs connectors,
and associated committers, through the spark RDD and SQL APIs.

## Parquet build and test

To clean build Apache Parquet:

```bash
ant parquet.build
```

There's no profile for using ASF staging as a source for artifacts.
Run this after the spark build so the files are already present.


To clean build Apache Parquet and then run the tests in the `parquet-hadoop` module:
```bash
ant parquet.test
```



# After the Vote Succeeds: publishing the release

## Update the announcement and create site/email notifications

Edit `src/text/announcement.txt` to have an up-to-date
description of the release.

The `release.site.announcement` target will generate these
annoucements. Execute the target and then review
the generated files in `target/`

```bash
ant release.site.announcement
```

The announcement must be generated before the next stage,
so make sure the common body of the site and email
annoucement is up-to-date: `src/text/core-announcement.txt`

## Build the Hadoop site

Set `hadoop.site.dir` to be the path of the
local clone of the ASF site repository
https://gitbox.apache.org/repos/asf/hadoop-site.git

```properties
hadoop.site.dir=/Users/stevel/hadoop/release/hadoop-site
```

Prepare the site; this also demand-generates the release announcement.

The site .tar.gz distributable is used for the site; this must already
have been downloaded. It must be untarred and copied under the
SCM-managed `${hadoop.site.dir}` repository, linked up
and then committed.

```bash
ant release.site.untar

ant release.site.docs
```

## Review the announcement.

### Manually link the site current/stable symlinks to the new release

In the hadoop site dir content/docs subdir

```bash

# update
git pull

# review current status
ls -l

# symlink current
rm current3
ln -s r.3.3.5 current3

# symlink stable
rm stable3
ln -s r3.3.5 stable3

# review new status
ls -l
```
Run `hugo` to build the site and verify. Follow the doc
https://cwiki.apache.org/confluence/display/HADOOP/How+to+generate+and+push+ASF+web+site 
NOTE: On some setups `hugo server` command to test locally may change the links to localhost.
In that case, create a commit first before running `hugo server` command and verify the links
on https://hadoop.apache.org/ after pushing the changes to the remote asf-site.


Finally, *commit*

```bash
git add .
git status
git commit -S -m "HADOOP-18470. Release Hadoop 3.3.5"
git push
```


## Promoting the RC artifacts to production through `svn move`

```bash

# check that the source and dest URLs are good
ant staging-init
# do the promotion
ant stage-move-to-production
```

This does a sequence of
1. Update local svn repo
2. Move the staged artifacts to the production path (and commit changes there)
3. Commit the staging dir changes.

Both commits use the generated message `production.commit.msg`.

The `svn-init` command prints this out.

## update the `current` ref

TODO: document/automate.
```bash
https://dist.apache.org/repos/dist/release/hadoop/common
```
Check that release URL in your browser.

## Publish Maven artifacts

do this at [https://repository.apache.org/#stagingRepositories](https://repository.apache.org/#stagingRepositories)

to verify this is visible
[search for hadoop-common](https://repository.apache.org/#nexus-search;quick~hadoop-common)
-verify the latest version is in the production repository.

## Declare the projects released in JIRA

Go to JIRA and:

1. Update the release JIRA as done; fix version = the release version.
2. Make sure there is a "next release" entry for that branch in the HADOOP, HDFS and YARN projects 
3. Declare the release version as done in HADOOP, HDFS and YARN projects.
   If there is warning of unresolved JIRAs, view them and see if they need to be closed.
   Otherwise, when declaring the release as published, move them to the next point
   release in that branch.

Release links.
* [HADOOP](https://issues.apache.org/jira/projects/HADOOP?selectedItem=com.atlassian.jira.jira-projects-plugin%3Arelease-page&status=unreleased)
* [HDFS](https://issues.apache.org/jira/projects/HDFS?selectedItem=com.atlassian.jira.jira-projects-plugin%3Arelease-page&status=unreleased)
* [YARN](https://issues.apache.org/jira/projects/YARN?selectedItem=com.atlassian.jira.jira-projects-plugin%3Arelease-page&status=unreleased)

## tag the final release and push that tag

The ant `print-tag-command` target prints the command needed to create and sign
a tag.

```bash
ant print-tag-command
```

Use the "tagging the final release" commands printed


## Send that email announcement once the artifacts have been propagated to the mirror sites.

1. Wait for all the release artifacts to be copied from the apache.org release repository to all the mirror sites.
2. Announce on hadoop-general as well as developer lists.

# update site doap rdf

Update hadoop-site  `hadoop-site/content/doap_Hadoop.rdf` with the new release version and date.

## Clean up your local system

For safety, purge your maven repo of all versions of the release, so
as to guarantee that everything comes from the production store.

```bash
ant mvn-purge
```

# Tips

## Git status prompt issues in fish

There are a lot of files, and if your shell has a prompt which shoes the git repo state, scanning can take a long time.
Disable it, such as for fish:

```fish
set -e __fish_git_prompt_showdirtystate
```

## Adding a global maven staging profile `asf-staging`

Many projects have a profile to use a staging repository, especially the ASF one.

Not all do -these builds are likely to fail.
Here is a profile, `asf-staging` which can be used to enable this.
The paths to the repository can be changed too, if desired.

Some of the maven builds invoked rely on this profile (e.g. avro).
For some unknown reason the parquet build doesn't seem to cope.

```xml
 <profile>
  <id>asf-staging</id>
  <properties>
    <!-- override point for ASF staging/snapshot repos -->
    <asf.staging>https://repository.apache.org/content/groups/staging/</asf.staging>
    <asf.snapshots>https://repository.apache.org/content/repositories/snapshots/</asf.snapshots>
  </properties>

  <pluginRepositories>
    <pluginRepository>
      <id>ASF Staging</id>
      <url>${asf.staging}</url>
    </pluginRepository>
    <pluginRepository>
      <id>ASF Snapshots</id>
      <url>${asf.snapshots}</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
      <releases>
        <enabled>false</enabled>
      </releases>
    </pluginRepository>

  </pluginRepositories>
  <repositories>
    <repository>
      <id>ASF Staging</id>
      <url>${asf.staging}</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
      <releases>
        <enabled>true</enabled>
      </releases>
    </repository>
    <repository>
      <id>ASF Snapshots</id>
      <url>${asf.snapshots}</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
      <releases>
        <enabled>true</enabled>
      </releases>
    </repository>
  </repositories>
</profile>

```

# Aborting an RC

Drop the staged artifacts from nexus
 [https://repository.apache.org/#stagingRepositories](https://repository.apache.org/#stagingRepositories)

Delete the tag. Print out the delete command and then copy/paste it into a terminal in the hadoop repository

```bash
ant print-tag-command
```

Remove downloaded files and maven artifactgs

```bash
ant clean mvn-purge
```


1. Go to the svn staging dir
2. `svn rm` the RC subdir
3. `svn commit -m "rollback RC"`

```bash
ant stage-svn-rollback
# and get the log
ant stage-svn-log
```

# Releasing Hadoop-thirdparty


See [releasing Hadoop-thirdparty](doc/thirdparty.md)

# Contributing to this module

There are lots of opportunities to contribute to the module
* New ant targets for more stages of the process, including automating more release steps
* Extending the maven module dependencies
* Adding more artifacts to the forbidden list
* Adding more validation tests to the maven test suites
* Adding more commands to execute against a distribution
* Adding github actions to help validate the module itself.

During the release phase of a Hadoop release: whatever is needed
to ship!

This repo works on Commit-then-Review; that is: no need to wait for
review by others before committing.
This is critical for rapid evolution during the release process.
Just expect to be required to justify changes after the fact.

* Contributions by non-committers should be submitted as github PRs.
* Contributions by committers MAY be just done as commits to the main branch.
* The repo currently supports forced push to the main branch. We may need to block this


# What can go wrong?

## Disconnection from remote system during build.

Docker should keep going. Use the `tmux` tool to maintain terminal sessions over interruptions.

## Multiple staging repositories in Nexus

If the Arm and x86 builds were running at the same time with `-asfrelease` or `-deploy` then the separate builds will have created their own repo.
Abort the process, drop the repositories and rerun the builds, sequentially, and only one set to create the staging repositories.

If a single host was building, then possibly network access came to the ASF Nexus server by multiple IP Addresses (i.e a VPN was involved).
If this happened then both repositories are incomplete.
Abort the build and retry. It may be that your network setup isn't going to work at all. The only fix there is: build somewhere else.
