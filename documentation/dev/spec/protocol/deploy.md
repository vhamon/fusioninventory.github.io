---
layout: page
title: API-REST deploy
---

#  API-REST-deploy

This document covers the agent/server protocol used by the "deploy"
task.

This module is used to do remote software deployments and command
executions.

##  Jobs request

###  Request

    /deploy?action=getJobs&machineid=$machineid

This method returns the jobs to process for a given machine.


In this example, one job depends on 2 files. The agent will download
the two files and then will process the order.  The order can be
started until maxValidityDate (GMT).  The files unique ID
are their sha512 signature.

Everytime the agent request this URL, the server has to send it.
Otherwise (empty page) the agent won't continue. This page will be
requested everytime the task is started.

If a job has been processed, the server must remove it from the job
list.

###  Answer

Jobs are processed in the order given by the server.

~~~~
{
   "jobs" : [
      {
         "checks" : [
            {
               "path" : "HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows NT/CurrentVersion/ProductName",
               "type" : "winkeyExists",
               "return" : "ignore"
            },
            {
               "value" : "SYSTEM",
               "match" : "winkeyEquals",
               "path" : "HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows NT/CurrentVersion/SoftwareType",
               "return" : "error"
            },
            {
               "match" : "winkeyMissing",
               "path" : "HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows NT/fooBar",
               "return" : "ignore"
            },
            {
               "path" : "c:/tmp.log",
               "type" : "fileExists",
               "return" : "ignore"
            },
            {
               "value" : 345,
               "path" : "c:/tmp2.log",
               "type" : "fileSizeEquals",
               "return" : "ignore"
            },
            {
               "value" : 45,
               "path" : "c:/tmp2.log",
               "type" : "fileSizeGreater",
               "return" : "error"
            },
            {
               "path" : "c:/tmp.log",
               "type" : "fileMissing",
               "return" : "ignore"
            },
            {
               "value" : 500000,
               "path" : "c:/",
               "type" : "freespaceGreater",
               "return" : "ignore"
            }
         ],
         "actions" : [
            {
               "messageBox" : {
               "checks" : [
                  {
                     "path" : "c:/no-popup.txt",
                     "type" : "fileExists",
                     "return" : "ignore"
                  }
               ],
                  "msg" : {
                     "fr" : "Une mise a jour de votre poste va etre realisee.",
                     "default" : "An upgrade of your post will be done. Please wait."
                  },
                  "timeout" : 600,
                  "type" : "info",
                  "title" : {
                     "fr" : "Boite a message",
                     "default" : "Message box"
                  }
               }
            },
            {
               "messageBox" : {
               "checks" : [
                  {
                     "path" : "c:/no-popup.txt",
                     "type" : "fileExists",
                     "return" : "ignore"
                  }
               ],
                  "msg" : {
                     "fr" : "Voulez-vous lancer la mise a jour maintenant",
                     "default" : "Do you want to launch the upgrade now?"
                  },
                  "timeout" : 600,
                  "type" : "postpone"
               }
            },
            {
               "cmd" : {
               "checks" : [
                  {
                     "path" : "c:/no-popup.txt",
                     "type" : "fileExists",
                     "return" : "ignore"
                  }
               ],
                  "retChecks" : [
                     {
                        "values" : [
                           "ok"
                        ],
                        "type" : "okPattern"
                     },
                     {
                        "values" : [
                           "error",
                           "warning"
                        ],
                        "type" : "errorPattern"
                     },
                     {
                        "values" : [
                           1,
                           54
                        ],
                        "type" : "okCode"
                     },
                     {
                        "values" : [
                           3,
                           53
                        ],
                        "type" : "errorCode"
                     }
                  ],
                  "exec" : "install.exe arg1 arg2",
                  "envs" : {
                     "LANGUAGE" : "de",
                     "OS" : "win",
                     "HOSTNAME" : "babel",
                     "ARCH" : "x64",
                     "OS_VERSION" : "5.1"
                  }
               }
            },
            {
               "move" : {
                  "from" : "install.ini",
                  "to" : "c:/myApp/toto"
               }
            },
            {
               "delete" : {
                  "list" : [
                  "c:/install.log",
                  "c:/foo.bar" ]
               }
            }
         ],
         "maxValidityDate" : 12334546,
         "associatedFiles" : [
            "8e9392b884d631728f917bd1231256f4c44618fd39afdd7385cc654e82affe747d8ec4f65ef9b55c6b1c517c340bf9af6c53291815dbab4e8f6176be2a511b10",
            "cd98fb514b2803d496a0f5c10f4fc8c069998fa4556c0dfbe27b95d742e0d218ec61d69ee42b092d008a542190fc78988b53ee43998d9688c978dac96864ff1a"
         ],
         "uuid" : "0fae2958-24d5-0651-c49c-d1fec1766af650"
      }
   ],
   "associatedFiles" : {
      "cd98fb514b2803d496a0f5c10f4fc8c069998fa4556c0dfbe27b95d742e0d218ec61d69ee42b092d008a542190fc78988b53ee43998d9688c978dac96864ff1a" : {
         "uncompress" : 0,
         "mirrors" : [
            "http://central.server/getConfig?d="
         ],
         "multiparts" : [
            "cd98fb514b2803d496a0f5c10f4fc8c069998fa4556c0dfbe27b95d742e0d218ec61d69ee42b092d008a542190fc78988b53ee43998d9688c978dac96864ff1a"
         ],
         "p2p" : 1,
         "p2p-retention-duration" : 36000,
         "name" : "install.ini"
      },
      "8e9392b884d631728f917bd1231256f4c44618fd39afdd7385cc654e82affe747d8ec4f65ef9b55c6b1c517c340bf9af6c53291815dbab4e8f6176be2a511b10" : {
         "uncompress" : 1,
         "mirrors" : [
            "http://mirror.local/temp/",
            "https://mirror.local/temp/",
            "//mirror.win/temp/",
            "ftp://mirror.win/temp/",
            "http://central.server/download?file="
         ],
         "multiparts" : [
            "34324234545435645645645645674567456745674567456745674567456745674567456745674567456745674567456745674567456745674576457645674567",
            "34345670987654320987654309876543209876789098765434567898765445678987654567898765456789875456789876545678987656789876545678765670",
            "98765434567890987654567890987654567890987654567890987654345678909876543456789098765456789098765456789098765456789876567898765660",
            "09874323456765434567876545678765445678765456898765456898765445678909876543456789876543456789098765456789987654567887654567887610"
         ],
         "p2p" : 1,
         "name" : "winInst.zip"
      },
      "fa939923b3a7502ca0b5fe02f006ac95eed54579595eb4b6a6dd6e97d11a18566835f76f0cd87d29646e333b396d69afff07a538f0cfd2d1d67a29aed2752b16" : {
         "uncompress" : 1,
         "mirrors" : [
            "https://mirror.local/temp/",
            "http://deploy/ocsinventory/deploy/prepare/",
            "//mirror.win/temp/",
            "ftp://mirror.win/temp/",
            "http://central.server/download?file="
         ],
         "name" : "glpi-0.78.2.tar.gz",
         "multiparts" : [
            "7dc573cc9d1eb905259861b42fa065199ddeab18e9ef8ffc548d1bd95b92c688d32122d083dba101c349e8247cc52b1d866083d9a23be0b6ce2bed8a95d3bb1a",
            "fef0ccfdeb9fbdead221ee9e7cba5188a0e672b7dd5b0ec64361c4c2f3bf443344d13076041faa71236f0c5555882a83d6c7013399a345e696682f475a3d780b",
            "fde60d2ceff15b67a80e30e861b665fcb553cda4e6e78d9ea3c1983937a1076fc390889991b23b8e2633e1d69fbd65eda2818eb5af48e9ecc458cff889d38ec0",
            "87efdb41d680695fc3f08dc34c7d86e859eee71ff6429b98c92d16fb68000630b0a67d45cf2f77f6031632e6c2641bf55681c5ad7c58c2a2039e722ceda37bbf",
            "51d7d9fad06ec3a2022487b6560d1f37c59c0114e5825a576d2a33075b087979c34d384ee1dca6c59c537947a8eb45911a87b7a9e9e0c38c5ddcab4c0bbaa937",
            "aff3d39921dce089875891c4e730560d6c651dd8b217314a78829303bf41614f04f52d7a978b177d7cea18e87a069528eaeaee489b83b33705fb67296787121a",
            "c3464fbe1c79c4f925070f150ee3231fb8cac199b1773d85176cc50d088baeebd49acf0f6c1be929dce28dfce7038cf58dce50675fb8df010f7f61df05ceae96",
            "210f240e08643fb46d512f1c53392ead8a981582cabcf3094706bc9e37d45e0e5d80fe5aec4ce67a1e04413261571009c1a395cdc05fd46d356888b247735a08",
            "6a69c79a8fc4484be8bf425c2dc7c18da6c537d7fd7bd1a62cbae3c5102bfb484319d24f3a789522af06e80a49b9464e76d1c21586a6c15d25af2de614dbf8b6",
            "db49d203092ba6d8cd8fd7c1d45a524bdaf8e9f245397b3b6c1256f1d716509292e494d0589fdfe56ee29a59e4266ad63e9f7fdae80c92860f2b84676fd9ce9d",
            "307ee18ea1fc6540e4c84d583ab10f9074654a456466273b0efc7d419a86360d1ec2db3ef0976995e15db68a765a6171f217dcf3de7253bfb679d8c9f45f9a2b",
            "a36263c1cedc8d2e1c0b9e6c17768e418ba8e721c717fe795ba4c883f9bfec5d10cce540efe75b4c3b0a1f22e6e5d2e3557f28f6e6cefecb416dcaaf35fda3be",
            "048a0a51496cf47784bd503f0436eb5ef79a68aeeb807100b0e276203e62e016f03210232490f2f4004822b8d287bb7d7ec778deff6d0f528c46fae7ed332902",
            "e363b07938ed5213d81f9d8f0139e9059e7e6b1fe96285d2a122041a73093868c8578a1e0082e875d9f3262b5a158f58330801536fee72aeeda1f82752e3f2a2",
            "6075de7f966c70f0b0713cb59d092f60a2fba1fb0ff08cb71e2881207f66e2e081669ed3308d8510e3a0f4bc2eb4840f9a96a10c8917166019bdee6010f6d3dc"
         ],
         "p2p" : 0
      }
   }
}
~~~~

###  Answer structure format (jobs)

#### actions

a list of operation to process

`messageBox`: Display an alert to the currently logged user.

* msg: a hash table. keys are language code and the value are the
  strings. "default" is the default language.
* type: the type of box to display to the user. info is an info box.
  postpone lets the user decide if the installation should be
  postponed.

`cmd`: Execute a command

* retChecks: an array of rule to eval to know if the command
  successed.  Order matter. (types are: okPattern, errorPattern,
  okCode, errorCode)
* exec: a string with the command the launch. 2>&1 is added as suffix
  by the agent at the run time.
* envs: a hash of environment variable to set before the cmd execution
* logLineLimit: the number of line to keep in the log (optional,
  default 3)

`move`: Move a file or directory

* from: source file or directory (* are accepted)
* to: destination file or directory

`copy`: Copy a file

* from: source file or directory (* are accepted)
* to: destination file or directory

`delete`: Delete a list of file or directory. The command works like
rm -Rf $arg

* list: a list of file or directory to remove

`mkdir`: Create a list of directory

* list: a list of directory to create

`installPkg` _(draft/not implemented)_

* name: the package name.
* method (optional): the method to use to install the package,
  (apt,yum).
* repo (optional): the repository to use, e.g: backport-wheezy or
  epel.
* version (optional): the version to use, e.g: ">= 3", "= 3.0-2".

#### associatedFiles

The files (sha512 reference) to be downloaded.
Those sha512 must be listed in the `associatedFiles`
section (array) of the answer.

#### checks

A array of check to evaluate. The code returned to the server and their behaviors a describe below.

* `winkeyExists`: true if $path is a valid key in the Windows
  registry.
* `winkeyEquals`: true if the content of $path in the registry is
  equal with $value
* `winkeyMissing`: true if $path doesn't exist in the Windows
  registry.
* `fileExists`: true if $path is a valid plain file.
* `fileSizeEquals`: true if $path is a file and if its size equals
  $value (Byte).
* `fileSizeGreater`: true if $path is a file and if its size is
  greater than $value (Byte).
* `fileSizeLess`: true if $path is a file and if its size is less than
  $value (Byte).
* `fileMissing`: true is $path is not a file.
* `freespaceGreater`: true if $path freespace is greater than $value
  (MByte).

{% include warning.html param="Registry path must:<br /><br />* start with the complete hive name (HKEY_something)<br />* use the `/` delimiter (not `\ ` )" %}

A check failure will stop a job and the type of failure can be
interpreted like the following:

* `ignore`: this failure is normal and the job can be
  treated sucessful.
  <br/>
  > For example, the package is already installed or uninstalled

* `error`: this failure is unexpected and the job must be treated like
  a critical error.
  <br/>
  > For example, there is no more space left on the harddrive to
    process installation.

###  Answer structure format (associatedFiles)

Only the files used by the jobs must be in this list.

* ``name`` : the local name of the file

* ``uncompress`` : should the archive be extracted

* ``mirror``: an array of base URL <br/>
  The mirror path must ends with a `/` if needed. The agent
  will join the mirror path directly with the multipart file name. The
  sha512 checksum is done after the optional gzip.

* ``p2p`` : does the file be shared with other agents ?

* ``p2p-retention-duration``: how long an archive must be kept for
  sharing before its deletion from cache (minutes)

## Update jobs' status

###  Request

    /deploy?action=setStatus&machineid=$machineid&part=$part&uuid=$uuid&currentStep=$currentStep&status=$status&msg=$msg

The agent uses this method to update the status of the deployment.

* `action=setStatus`: the method name
* `machinedid=$machineid`: the machine unique id (mandatory)
* `part=$part`: either 'file' or 'job' (optional, default=job)
* `uuid=$uuid`: the job UUID (mandatory)
* `sha512=$sha512`: the sha512 of the file (mandatory if part=file)
* `status=$status`: 'ko', 'ok'
* `currentStep=$currentStep`:
  received/checking/downloading/extracting/processing (mandatory)
* `msg=$msg`: a 120 character length string, this string must be
  static in order to reduce the transalation effort. (optional)
* `log=$log`: log file sent through in UNIX format (\n). (looks below)
* `checknum=$checknum`: the check number (optional), start at 0
* `actionnum=$actionnum`: the action number (optional), start at 0



Current Step:

* received (job): the agent received the job request _(not
  implemented)_
* downloading (file) the agent started to check the mirror to download
  the file
* extracting (job): preparing the working directory
* processing (job): the agent is processing the job

The status:

* ko (job, file): the job processing or the file downloading had been
  abort
* ok (job, file): the job processing or the file downloading had been
  successful

The log:

The log length shouldn't be greater than 1000 caracters. If the value
is above, the begining may be silently truncated by the agent.

###  Answer

The expected answer is a JSON encoded empty hash ref {}.


##  Multipart

The multipart is the list of the file to download (from one to
unlimited). This can file name can include a relative directory path
too. Once the downloads are down, they will be joined together in the
list order. If the subfile has a .gz suffix, it will be extracted
first.

The agent will use the .gz extension of the part file to identify an
optional compression. So be careful if the initial file has already
this extension, you need to rename it to something like
file.tar.gz-01.

### Request

POST a log file for a given job.

~~~~
#!/bin/sh

MIRROR="http://deploy/ocsinventory/deploy/prepare/"
FILE=$1

if [ -z $FILE ]; then
echo "exit"
exit 1
fi
split $FILE $FILE

sha512=`sha512sum $FILE|awk '{print $1}'`
echo "

\"$sha512\" : {
\"uncompress\" : 1,
    \"mirror\" : [
    \"https://mirror.local/temp/\",
    \"$MIRROR\",
    \"//mirror.win/temp/\",
    \"ftp://mirror.win/temp/\",
    \"http://central.server/download?file=\"
    ],
    \"p2p\" : 1,
    \"multipart\" : ["
request
first=""
for part in $FILE?*; do
    [ ! -z $first ] && echo ','
    echo {\"$part\" : \"`sha512sum $part|awk '{print $1}'`\"}
    first="no"
done

echo "
    ],
    \"name\" : \"$FILE\"
    }
"
~~~~

##  Communication examples

Get the job, download the file and run the command.

    http://localhost:8080/deploy1?action=getJobs&machineid=fakeid
    http://localhost:8080/deploy1?action=setStatus&currentStep=checking&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy1?action=setStatus&currentStep=downloading&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy1?action=setStatus&sha512=60ace8efe87aa2a6ef3fed5392fef70bd2d733e198d6965135dd44ea3b8b7cf9937dd03cbfe3206fe11e0ed23e9dd489a1c1e6fa71bf912f229917bef30aeafe&currentStep=downloading&part=file&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy1?action=setStatus&sha512=60ace8efe87aa2a6ef3fed5392fef70bd2d733e198d6965135dd44ea3b8b7cf9937dd03cbfe3206fe11e0ed23e9dd489a1c1e6fa71bf912f229917bef30aeafe&status=ok&currentStep=downloading&part=file&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy1?action=setStatus&status=ok&currentStep=downloading&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy1?action=setStatus&status=ok&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650

Failed to download the file 60ace8e(...)

    http://localhost:8080/deploy1.1?action=getJobs&machineid=fakeid
    http://localhost:8080/deploy1.1?action=setStatus&currentStep=checking&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy1.1?action=setStatus&currentStep=downloading&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy1.1?action=setStatus&sha512=60ace8efe87aa2a6ef3fed5392fef70bd2d733e198d6965135dd44ea3b8b7cf9937dd03cbfe3206fe11e0ed23e9dd489a1c1e6fa71bf912f229917bef30aeafe&currentStep=downloading&part=file&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy1.1?action=setStatus&msg=download%20failed&sha512=60ace8efe87aa2a6ef3fed5392fef70bd2d733e198d6965135dd44ea3b8b7cf9937dd03cbfe3206fe11e0ed23e9dd489a1c1e6fa71bf912f229917bef30aeafe&status=ko&currentStep=downloading&part=file&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650

Fails to run a command and set the log file.

    http://localhost:8080/deploy4?action=getJobs&machineid=fakeid
    http://localhost:8080/deploy4?action=setStatus&currentStep=checking&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy4?action=setStatus&currentStep=downloading&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy4?action=setStatus&status=ok&currentStep=downloading&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy4?action=setStatus&log[]=%20%20%20%20osname%3Dlinux%2C%20osvers%3D2.6.32-5-amd64%2C%20archname%3Dx86_64-linux-gnu-thread-multi&log[]=%20%20Platform%3A&log[]=%20%20%20&log[]=Summary%20of%20my%20perl5%20%28revision%205%20version%2012%20subversion%204%29%20configuration%3A&log[]=--------------------------------&log[]=exit%20status%3A%20%600%27&log[]=exit%20status%20is%20not%20ok%3A%20%600%27&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650&actionnum=0
    http://localhost:8080/deploy4?action=setStatus&msg=action%20processing%20failure&status=ko&currentStep=processing&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650&actionnum=0

Run a command without issue.

    http://localhost:8080/deploy5?action=getJobs&machineid=fakeid
    http://localhost:8080/deploy5?action=setStatus&currentStep=checking&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy5?action=setStatus&currentStep=downloading&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy5?action=setStatus&status=ok&currentStep=downloading&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy5?action=setStatus&status=ok&currentStep=processing&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650&actionnum=0
    http://localhost:8080/deploy5?action=setStatus&status=ok&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650

Second action (actionnum=1) failed.

    http://localhost:8080/deploy13?action=getJobs&machineid=fakeid
    http://localhost:8080/deploy13?action=setStatus&currentStep=checking&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy13?action=setStatus&currentStep=downloading&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy13?action=setStatus&status=ok&currentStep=downloading&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650
    http://localhost:8080/deploy13?action=setStatus&status=ok&currentStep=processing&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650&actionnum=0
    http://localhost:8080/deploy13?action=setStatus&log[]=Failed%20to%20move%20file%3A%20%27%2Fhome%2Fgoneri%2Ffusioninventory%2Fagent-task-deploy%2Ft%2F..%2Ftmp%2Fdeploy-test%2Fserver%2FDeploy.totorMissing%27%20to%20%27%2Fhome%2Fgoneri%2Ffusioninventory%2Fagent-task-deploy%2Ft%2F..%2Ftmp%2Fdeploy-test%2Fserver%2FDeploy.titi%27&log[]=Aucun%20fichier%20ou%20dossier%20de%20ce%20type&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650&actionnum=1
    http://localhost:8080/deploy13?action=setStatus&msg=action%20processing%20failure&status=ko&currentStep=processing&part=job&machineid=fakeid&uuid=0fae2958-24d5-0651-c49c-d1fec1766af650&actionnum=1
