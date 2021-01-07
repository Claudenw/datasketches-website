<!--
    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.
-->

# Apache C++ Release Steps
__NOTES:__

* This process covers major and minor releases only. Bug-fix releases, which increment the third digit, are performed on a A.B.X branch and not on master, but otherwise is similar.
* Some of these operations can be performed either on the Command-Line or in your IDE, whatever you prefer.

## Preparation
* Confirm correctness for
    * LICENSE
    * NOTICE -- check for copyright dates
    * README.md
    * .gitignore
    * DISCLAIMER-WIP
         
* From Command Line or IDE:
    * Run Unit tests
    * Run Code Coverage > 90%
    * Confirm that all __temporary__ branches are checked into master and/or deleted, both local and remote.
    * Confirm any new bug fixes have corresponding tests

* From Command Line at Component root:
  * Confirm GPG is running (check this every time you open a new Terminal):
      * $ env | grep GPG # you should see something like: GPG_TTY=/dev/ttys000
      * To start GPG if GPG Agent is not running:
          * $ eval $(gpg-agent --daemon)
  * Confirm GitHub repository is current and git status is clean:
      * $ git status
          * "nothing to commit, working tree clean"
  * At major version releases, search for deprecated code and remove at __Major Versions__ only.
      * $ find . -name "*.?pp" -type f -print | xargs grep -i -n -s -A0 "deprecated"
      * may need to ignore false positives in the pybind11 directory
 

## Create Permanent Release Branch & Python Version Preparation
* Assume target version = A.B.0
* From IDE or Command Line: 
    * Switch from master to new __Permanent Branch__: "A.B.X"
    * Check setup.py "version=" line to A.B.X (remove -SNAPSHOT, do not change A or B)
    * Commit the change. __DO NOT PUSH!__
    * Create Annotated TAG: A.B.0-RC1 (or RCn)
    * Write down the Git hash : example: 40c6f4f
    * Now Push Branch  "A.B.X" with edited setup.py to origin
    * __DO NOT MERGE THIS PERMANENT BRANCH INTO MASTER__
* From IDE or Command-line: 
    * Do explicit push of tags on branch "A.B.X" to origin:
        * $ git push origin --tags
* From a web browser at origin web site: github.com/apache/datasketches-\<component\>
    * Select the A.B.X branch
    * Confirm that the tag: A.B.0-RC1 exists and that the tag is on the latest commit and with the correct Git hash.
    * __DO NOT CREATE PR OR MERGE THIS PERMANENT BRANCH INTO MASTER__
* From IDE or Command Line:
    * Confirm that the tag A.B.0-RC1 and the branch A.B.X, and HEAD coincide with the correct Git hash.
    * Confirm that there are no unstaged or staged changes.
    * Return to master branch
    * Edit master setup.py to A'.B'.0-SNAPSHOT where A' or B' will be incremented by 1. (Bug fix releases will change the 3rd digit)
    * Commit and Push this change to origin/master with the comment "Release Process: Change setup.py version to A'.B'.0-SNAPSHOT."
    * Return to release branch A.B.X
    * You may minimize your IDE, pointing at the release branch.
    
## Create and/or Checkout Local *dist/dev* directories on your system
* If you have not already, on your system create the two directory structures that mirror the dist.apache.org/repos/ directories:
    * mkdir dist/dev/datasketches/
    * mkdir dist/release/datasketches/
* Checkout both "dev" and "release" directories 
    * Open a terminal in the dist/dev/datasketches directory and do a checkout:
        * svn co https://dist.apache.org/repos/dist/dev/datasketches/ .      #Note the DOT
        * svn status    # make sure it is clean
    * Open a terminal in the dist/release/datasketches directory and do a checkout:
        * svn co https://dist.apache.org/repos/dist/release/datasketches/ .  #Note the DOT
        * svn status    # make sure it is clean

## Create the Candidate Apache Release Distribution on *dist/dev*
### Create primary zip files & signatures
* You will need the following arguments:
  * Absolute path of target project.basedir on your system
  * Artifact name : datasketches-\<component\> where component is e.g., cpp
  * GitHub Tag: A.B.0-RC1 (or RCn)
* Start a new terminal in the above dist/dev/datasketches/scripts directory on your system: 
  * Confirm GPG is running: $ env | grep GPG
      * If not: $ eval $(gpg-agent --daemon)
  * Run something like:
    * $ ./bashDeployToDist.sh /Users/\<name\>/dev/git/Apache/datasketches-\<component\> datasketches-\<component\> A.B.0-RC1
    * Follow the instructions.
    * NOTE: if you get the error "gpg: signing failed: No pinentry":
        * open .gnupg/gpg-agent.conf
        * change to: pinentry-program /usr/local/bin/pinentry-tty
        * reload the gpg agent in the terminal: gpg-connect-agent reloadagent /bye   
        * restart the ./bashDeployToDist script
    * Close the terminal
* Check and grab the web URL ~ https://dist.apache.org/repos/dist/dev/datasketches/\<component\>/A.B.0-RC1/
    * There should be 3 files: \*-src.zip, \*-src.zip.asc, \*-src.zip.sha512 

### Create Copy of External Artifact Distributions
* For Java, we need to place copies of the artifact jars deployed to Nexus under a "maven" directory.
* For external artifacts of Python or Docker it will be something else.
* For example see <https://dist.apache.org/repos/dist/release/datasketches/java/1.3.0-incubating/>
* These must be signed with GPG (.asc) and SHA512 (.sha512)
* I will create a script for these artifacts someday :)

## Prepare & Send [VOTE] Letter to dev@
* See VoteTemplates directory for a recent example
* If vote is not successful, fix the problem and repeat above steps.
* After a successful vote return to __this point__ and continue ...

## Prepare & Send [VOTE-RESULT] Letter to dev@
* See VoteTemplates directory for a recent example
* Declare that the vote is closed.
* Summarize PPMC vote results

## Move files from dev/staging to release
### Move primary zip files from *dist/dev* to *dist/release*
* In local dist/__dev__/datasketches/
    * Open Terminal #1 
        * Perform SVN Checkout:
            * $ svn co https://dist.apache.org/repos/dist/dev/datasketches/ .  #note dot at end
* In local dist/__release__/datasketches/
    * Open Terminal #2
        * Perform SVN Checkout:
            * $ svn co https://dist.apache.org/repos/dist/release/datasketches/ . #note dot at end
        * Create new version directory under appropriate component directory:
            * $ mkdir -p \<component\>/A.B.0
    * Using local file system copy files 
        * From  ... /dist/dev/datasketches/\<component\>/version-RCnn/*
        * To    ... /dist/release/datasketches/\<component\>/version (no RCnn)/*
    * Using Terminal #2 at ... /dist/release/datasketches directory:
        * svn add . --force
        * svn ci -m "Release A.B.0"
        * Remove the prior release...
        * svn remove \<component\>/X.Y.0
        * svn ci -m "Remove Prior release"
        * svn status # should be empty
    * Using local file system
        * Delete the prior X.Y.0 directory  

### Move External Artifact Distributions *dist/dev* to *dist/release*

## Create & Document Release Tag on GitHub
* Open your IDE and switch to the recently created Release Branch A.B.X
* Find the recently created A.B.0-RCn tag in that branch
* At that same GitHub ID hash, create a new tag A.B.0 (without the RCn).
* From the Command Line: Push the new tag to origin:
    * $ git push origin --tags
* On the GitHub component site document the release 

## Update Website Downloads.md "Latest Source Zip Files" Table
* This script assumes that the remote .../dist/release/datasketches/... directories are up-to-date with no old releases.  
* Start a new terminal in the ../dist/dev/datasketches/scripts directory on your system:
* Make sure you local website directory is pointing to master and up-to-date. 
* Run the following with the argument specifying the location of your local website directory:
    * $ ./createDownloadsInclude.sh /Users/\<name\>/ ... /datasketches-website
* When this is done, be sure to commit the changes to the website.

## Update Javadocs (or Equivalent) on Website

## Update Website Documentation (if new functionality)

## Prepare Announce Letter to dev@
* ASF requests that you wait 24 hours to publish Announce letter to allow the propagation to mirrors.
* Use recent template
* Summarize vote results

## Update These Instructions
* If you have updated this file or any of the scripts, please check it in using SVN using your local dist/dev directory copy:
    * $ svn status
    * $ svn add . --force  # if adding a file for the first time
    * $ svn ci -m "update Release Steps" 
