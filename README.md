[comment]: <> (what cli does, treat it as the brochure of my car)

# powermole/cli

This program will let you perform port forwarding, redirect internet traffic, and transfer files to, and issue commands on,
a host without making a direct connection (ie. via one or more intermediate hosts), which would undoubtedly compromise your privacy.
This solution can only work when you or your peers own one or more hosts as this program communicates with SSH servers.
This program can be viewed as a multi-versatile wrapper around SSH with the ProxyJump directive enabled.


## Demo

Running program with TOR mode enabled.
In this mode, the target destination host acts as an exit node (in TOR terminology).
A program of choice can be started automatically (when the program states "READY") by setting the application directive in the configuration file
For example, Mozilla Firefox (and with FOR mode, Mozilla Thunderbird).
There are with 2 gateways (intermediate hosts) involved.

![alt text](img/executing_powermole_tor_mode_2_gateway.gif)


Running program with INTERACTIVE mode enabled.
In this mode, a rudimentary terminal interface is provided which enables the user to interact with the target destination host.
There is 1 gateway (intermediate host) involved.

![alt text](img/executing_powermole_interactive_mode_1_gateway.gif)


Running program with FILE mode enabled.
In this mode, files are transferred from localhost (client) to the target destination host.
There are 1 gateway (intermediate host) involved.

![alt text](img/executing_powermole_file_mode_1_gateway.gif)


## How it works

### Terminology
* **Tunnel** is an established connection from localhost to target destination host through intermediate hosts (called Gateways).
* **Agent** is a python module running on the target destination host. It performs various functions.
* **Instructor** sends data and instructions to the *Agent* by utilizing a forwarded connection provided by *Tunnel*.

This cli package uses the lib package to create a Tunnel and models the specific Instructor to communicate with the Agent (on the target destination host).
The Agent communicates directly with the operating system of the host on which it resides.
The Agent is responsible to redirect internet traffic (TOR mode), put files (FILE mode), and issue commands (INTERACTIVE mode).
For port forwarding (FOR mode), the program simply relies on SSH itself. The Agent also responds to heartbeats send by localhost to check if connection is still intact.

![alt text](img/illustration_how_it_works.png)

For more details, including illustrations, please consult the [corelib](https://github.com/yutanicorp/powermolelib) on GitHub.


## Requirements (software)

* Every host (except local host, the _client_) needs a running SSH daemon.


## Requirements (functional)

* The solution doesn't work with Windows, only on Linux (tested on Red Hat).
* The client and all servers have Python >3.6 as their default interpreter.
* You need _at least_ 1 server.
* Have the associated SSH identification file (i.e. the private key) for these servers.
* Due to security reasons, SSH password login is not supported.
* This program don't require root privileges on client (*to be confirmed*).


## Installation

If you use the standard packet manager:

```
$ pip install powermolecli
```
or if you use pipx:
```
$ pipx install powermolecli
```


## Usage

Issue this command for help:

```
$ powermolecli --help

usage: powermolecli [-h] [--config-file CONFIG_FILE]
           [--log-level {debug,info,warning,error,critical}]

powermole allows you to connect to a target destination host via one or more intermediaries, offering a variety of modes (FOR, TOR, FILE, and INTERACTIVE) to perform a variety of tasks

optional arguments:
  -h, --help       show this help message and exit
  --config-file, -c CONFIG_FILE
                   The location of the config file
  --log-level, -L {debug,info,warning,error,critical}
                   Provide the log level. Defaults to info.
```

Issue this command to actually execute the program.

```
$ powermolecli --config-file ~/powermole.json
```


Use option "--log-level debug" to print every activity in the program.

```
$ powermolecli -c ~/powermole.json -l debug
```

The JSON file contains directives to enter one of the modes listed below:

* TOR mode
* FOR(warding) mode
* INTERACTIVE mode
* FILE mode


In TOR mode, the target destination host acts as an exit node (in TOR terminology).

![alt text](img/illustration_tor.png)

In FOR(warding) mode, connections are forwarded to the target destination host, on which, for example, an email server (e.g. Postfix) is running and a local email client want to connect to its listening ports.

![alt text](img/illustration_forwarding.png)

In INTERACTIVE mode, a rudimentary terminal interface is provided which enables the user to interact with the target destination host.

![alt text](img/illustration_interactive.png)

In FILE mode, files are transferred from client to the target destination host.

![alt text](img/illustration_file.png)


## Configuration

#### To enable TOR mode

Edit the JSON document in the configuration file to incorporate the keywords **mode**, **gateways**, **destination**, and optionally **application**.
When **application** is specified, the program will start the application of choice once the Tunnel is ready.
In the example below, the program drills through 3 hosts and ends at host #4.
Hitting ctrl + c in terminal will dismantle Tunnel (and stop application)

```
    {
    "mode":         "TOR",
    "gateways":    [{"host_ip": "10.10.1.72",
                     "user": "root",
                     "identity_file": "/home/vincent/.ssh/id_rsa_be_vm"},
                    {"host_ip": "10.10.2.92",
                     "user": "root",
                     "identity_file": "/home/vincent/.ssh/id_rsa_it_vm"},
                    {"host_ip": "10.10.3.52",
                     "user": "root",
                     "identity_file": "/home/vincent/.ssh/id_rsa_es_vm"}],
    "destination": {"host_ip": "10.10.4.31",
                    "user": "root",
                    "identity_file": "/home/vincent/.ssh/id_rsa_gr_vm"},
    "application": {"binary_name": "firefox",
                    "binary_location": "/usr/bin/firefox"}
    }
```

#### To enable FOR(warding) mode

Edit the JSON document to incorporate the keywords **mode**, **gateways**, **destination**, **forwarders**, and optionally **application**.
When application is specified, then the program will start this application once the Tunnel is ready.
Hitting ctrl + c in terminal will dismantle Tunnel (and stop application)

```
    {
    "mode":         "FOR",
    "gateways":    [{"host_ip": "10.10.1.72",
                     "user": "root",
                     "identity_file": "/home/vincent/.ssh/id_rsa_be_vm"}],
    "forwarders": [{"local_port": 1587,
                    "remote_interface": "localhost",
                    "remote_port": 587},
                   {"local_port": 1995,
                    "remote_interface": "localhost",
                    "remote_port": 995}],
    "destination": {"host_ip": "10.10.2.92",
                    "user": "root",
                    "identity_file": "/home/vincent/.ssh/id_rsa_it_vm"}
    }
```

#### To enable INTERACTIVE mode

Edit the JSON document to incorporate the keywords **mode**, **gateways**, and **destination**.
Hitting ctrl + c in terminal will dismantle Tunnel.

```
    {
    "mode":         "INTERACTIVE",
    "gateways":    [{"host_ip": "10.10.1.72",
                     "user": "root",
                     "identity_file": "/home/vincent/.ssh/id_rsa_be_vm"],
    "destination": {"host_ip": "10.10.2.92",
                    "user": "root",
                    "identity_file": "/home/vincent/.ssh/id_rsa_it_vm"}
    }
```

#### To enable FILE mode
Edit the JSON document to incorporate the keywords **mode**, **gateways**, **destination** and **files**.

```
    {
    "mode":         "FILE",
    "gateways":    [{"host_ip": "10.10.1.72",
                     "user": "root",
                     "identity_file": "/home/vincent/.ssh/id_rsa_be_vm"}],
    "destination": {"host_ip": "10.10.2.92",
                    "user": "root",
                    "identity_file": "/home/vincent/.ssh/id_rsa_it_vm"},
    "files":      [{"source": "/home/vincent/amsterdam_de_jordaan.jpg",
                    "destination": "/tmp"},
                   {"source": "/home/vincent/amsterdam_nine_streets.jpg",
                    "destination": "/tmp"}]
    }
```



## Planning

The format of the configuration file will be modified to include:
  - the MACHINE_DEPLOY_PATH (default: /tmp/)
  - the LOCAL_PROXY_PORT (default: 8080)
  - for mode FILE "direction", which can have value UPLOAD|DOWNLOAD (note: only UPLOAD is available)

The program lacks the ability to choose a correct Python interpreter. The program will
start the Agent module on destination host with the Python 3.6 interpreter. This is
hardcoded, but a future release should also include a mechanism to use a higher version
of Python if available.


## Development Workflow

The workflow supports the following steps

 * lint
 * test
 * build
 * document
 * upload
 * graph

These actions are supported out of the box by the corresponding scripts under _CI/scripts directory with sane defaults based on best practices.
Sourcing setup_aliases.ps1 for windows powershell or setup_aliases.sh in bash on Mac or Linux will provide with handy aliases for the shell of all those commands prepended with an underscore.

The bootstrap script creates a .venv directory inside the project directory hosting the virtual environment. It uses pipenv for that.
It is called by all other scripts before they do anything. So one could simple start by calling _lint and that would set up everything before it tried to actually lint the project

Once the code is ready to be delivered the _tag script should be called accepting one of three arguments, patch, minor, major following the semantic versioning scheme.
So for the initial delivery one would call

    $ _tag --minor

which would bump the version of the project to 0.1.0 tag it in git and do a push and also ask for the change and automagically update HISTORY.rst with the version and the change provided.


So the full workflow after git is initialized is:

 * repeat as necessary (of course it could be test - code - lint)
   * code
   * lint
   * test
 * commit and push
 * develop more through the code-lint-test cycle
 * tag (with the appropriate argument)
 * build
 * upload (if you want to host your package in pypi)
 * document (of course this could be run at any point)


## Important Information

This template is based on pipenv. In order to be compatible with requirements.txt so the actual created package can be used by any part of the existing python ecosystem some hacks were needed.
So when building a package out of this **do not** simple call

    $ python setup.py sdist bdist_egg

## Documentation

* Documentation: https://powermolecli.readthedocs.org/en/latest


## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.


## Authors

* **Vincent Schouten** - *Initial work* - [LINK](https://github.com/yutanicorp/powermolecli)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details


## Acknowledgments

* Costas Tyfoxylos
* MisterDaneel (developer of pysoxy)

