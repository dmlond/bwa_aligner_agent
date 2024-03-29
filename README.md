bwa_aligner_agent
==================

This is composed of both a dmlond/google_agent_base wrapper agent for
dmlond/bwa_aligner and a SpreadsheetAgent::Runner
script which monitors the configured spreadsheet worksheet
for new subsets that are ready to align, and runs the bwa_aligner_agent
on them.

Configuring dmlond/bwa_aligner_agent
-

Before you can build dmlond/bwa_aligner_agent (see next for
instructions), you must first create a file called
agent.conf.yml in the bwa_aligner_agent build directory.
For reference, you can use the agent.conf.yml.example
file included. Copy agent.conf.yml.example to agent.conf.yml.
This is a YAML file with the account information
required to connect to the Google Drive system,
and a specific Spreadsheet to interact with.

You must change the following fields:

guser: your google account userid
gpass: your password.

It is highly recommended
that you activate [Google 2-Step Verificaiton](https://www.google.com/landing/2step/)
in google, and then create an [App Password](https://support.google.com/accounts/answer/185833?hl=en)
for your agent to use in this field.  Then, if your App Password gets stolen or compromised,
you can delete it and create a new one.

spreadsheet_name: the name of the spreadsheet in google drive that your agent will use.

This must have a worksheet in it called 'alignment'.   It should have the following column names in the first row:

subset, build, reference, ready, bwa_aligner, complete

send_to: this cannot be null, but it is not used.  The agent runs in foreground mode, and prints errors to
STDOUT and STDERR which can be accessed by the docker logs command on a running agent container.

Note, if you are wanting to dmlond/bwa_aligner_agent in conjunction with
dmlond/split_agent, you should make sure that the agent.conf.yml
file is the same for both before building them.

building dmlond/bwa_aligner_agent
-

To build dmlond/bwa_aligner_agent, you have to download dmlond/bwa_aligner from Dockerhub, or build it
from scratch.  If you download dmlond/bwa_aligner you can 'tag' it with the tag 'dmlond/google_agent_candidate'.
If you build it from scratch, you can either build it with '-t dmlond/google_agent_candidate'
or build it with '-t dmlond/bwa_aligner', and then tag it with 'dmlond/google_agent_candidate'.
Then build dmlond/google_agent_base to produce an image tagged 'dmlond/google_bwa_aligner_agent_candidate'
Then you can build dmlond/bwa_aligner_agent.

Here are the steps starting from a download of bwa_aligner from the dockerhub.  When you
run dmlond/bwa_aligner, docker will download it from Dockerhub, and then it will run and print
the usage instructions for bwa_aligner.  It will then be available to tag appropriately.

```bash
$ sudo docker run dmlond/bwa_aligner
$ sudo docker tag dmlond/bwa_aligner dmlond/google_agent_candidate
$ sudo docker build -t dmlond/bwa_aligner_agent_candidate google_agent_base
$ sudo docker build -t dmlond/bwa_aligner_agent bwa_aligner_agent
```

You should consult the README for docker_bwa_aligner for instructions on how
to build bwa_aligner from scratch.

running dmlond/bwa_aligner_agent
-

You should run dmlond/bwa_aligner_agent in 'deamon' mode with the -d switch,
and provide the volumes that are required by dmlond/bwa_aligner.  The
runner will run continuously, waiting for new subsets to be made ready in the
configured spreadsheet.  To align a subset against a specific build and reference,
fill in the 'build' and 'reference' fields for the subset(s) based on the arguments
you would pass to dmlond/bwa_aligner.  Enter '1' in the 'ready' field, and the runner will
at some point pick up the entry and attempt to run the bwa_aligner_agent script on it.

You can run more than one dmlond/bwa_aligner_agent, sharing the same volumes, to run multiple
alignments in parallel.

While an agent is running, you will see 'r:containerid' in the 'bwa_aligner' field.  At some
point this will change to '1' if it completes, or 'f:contiainerid' if it fails. Sometimes,
the agent completes or fails, but cannot update the spreadsheet.  You should monitor the
logs of each running agent container, and determine whether a run of an agent completed or
failed, and manually update the spreadsheet appropriately.  You can supply the 'containerid'
in the field to docker logs or docker inspect to find out information about the specific
instance of the agent.

License
-------
Docker containers that demonstrate a proof of concept bwa alignment workflow
Copyright (c) 2014, Duke University
All rights reserved. Darin London

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of the {organization} nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

