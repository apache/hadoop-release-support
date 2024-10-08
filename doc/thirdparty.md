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

# Releasing Hadoop Third party

See wiki page [How To Release Hadoop-Thirdparty](https://cwiki.apache.org/confluence/display/HADOOP2/How+To+Release+Hadoop-Thirdparty)


Support for this release workflow is pretty minimal, but releasing it is simpler
than a manual build.

1. Update the branches and maven artifact versions
2. build and test. This can be done with the help of a (draft) PR to upgrade hadoop from the RC.
3. Create the vote message.

## Configuration options

All options are prefixed `3p.`; the rest of their name matches that
of the core release.

The core property is the version to release
```properties
3p.version=1.3.0
```
This will trigger the loading of the relevant file from
`src/releases/3p/`:

```
src/releases/3p/3p-release-1.3.0.properties
```
It contains the options needed to create the vote message and execute the other
targets in the build to validate the third party release

```properties
3p.rc=RC1
3p.branch=https://github.com/apache/hadoop-thirdparty/commits/release-1.3.0-RC1
3p.git.commit.id=0fd62903b071b5186f31b7030ce42e1c00f6bb6a 
3p.jira.id=HADOOP-19252
3p.nexus.staging.url=https://repository.apache.org/content/repositories/orgapachehadoop-1420
3p.src.dir=https://dist.apache.org/repos/dist/dev/hadoop/hadoop-thirdparty-1.3.0-RC1
3p.staging.url=https://dist.apache.org/repos/dist/dev/hadoop/hadoop-thirdparty-1.3.0-RC1
3p.tag.name=release-1.3.0-RC1
```

## Targets:

All targets are prefixed `3p.`

| target            | function                                             |
|-------------------|------------------------------------------------------|
| `3p.mvn-purge`    | remove all third party artifacts from the local repo |
| `3p.vote-message` | generate a vote message in target/3p.vote.txt        |
| `3p.print-tag-command` | Print all the tag commands for a release |

```
 3p.mvn-purge                     purge all local hadoop-thirdparty 
 3p.print-tag-command             print the git command to tag the rc
 3p.stage-move-to-production      promote the staged the thirdparty RC into dist
 3p.stage-svn-rollback            rollback a thirdparty version staged to RC
 3p.vote-message                  build the vote message
```

Third party artifacts must be staged to the same svn repository as for
staging full hadoop releases, as set in `staging.dir`