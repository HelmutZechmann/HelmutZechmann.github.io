---
layout: post
title: Inspect Docker Containers With JQ
tags: [docker, jq]
---

The [docker inspect](https://docs.docker.com/reference/commandline/inspect/) command provides useful information about docker containers. But the the huge output of this command can be quite confusing. Since the output comes in json format, the [jq-tool](http://stedolan.github.io/jq/) can be used to get an overview of the output and pick interesting parts.

## Show all keys in the output:

 The following command shows all available keys in the ```inspect``` output for the container with id 89db758135a4:

    docker inspect 89db758135a4 | jq .[0] | jq keys

    [
      "AppArmorProfile",
      "Args",
      "Config",
      "Created",
      "Driver",
      "ExecDriver",
      "ExecIDs",
      "HostConfig",
      "HostnamePath",
      "HostsPath",
      "Id",
      "Image",
      "LogPath",
      "MountLabel",
      "Name",
      "NetworkSettings",
      "Path",
      "ProcessLabel",
      "ResolvConfPath",
      "RestartCount",
      "State",
      "Volumes",
      "VolumesRW"
    ]

## Show values for a specific key:

The following command can be used to show information about the state of the container:

    docker inspect 89db758135a4 | jq .[0] | jq .State
    {
    "Running": false,
    "Paused": false,
    "Restarting": false,
    "OOMKilled": false,
    "Dead": false,
    "Pid": 0,
    "ExitCode": 127,
    "Error": "",
    "StartedAt": "2015-07-30T07:46:55.819253566Z",
    "FinishedAt": "2015-07-30T07:46:55.967718855Z"
    }
