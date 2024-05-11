# remote-ssh-access

NAME
    remote-ssh-access - An application for creating handy SSH client
    shortcuts.

SYNOPSIS
        remote-ssh-access [-a|--add] [-h] [-s|--silent] [-N|--no-defkey] [cmds...]

DESCRIPTION
    This script replaces the use of aliases or other small scripts for
    automating and managing SSH client commands.

README
    This small script creates and uses shortcuts for launching SSH sessions.
    It's a useful tool when you have a lot of systems to manage.

    This program is meant to be executed through a symlink to a hard link.
    The hard link file is called a "parameter file"; the symlink is referred
    to as the "shortcut". If you wish, the hard link may also serve as the
    shortcut, obviating the step of creating a symlink.

    The "parameter file" is created with the standard ln(1) command. Its
    name forms the arguments that the shorcut file uses to launch SSH
    sessions. The syntax for a parameter file is defined given the following
    syntax:

    host[:user[:port[:key[:version[:cmd]]]]]

    Where:

    All parameters are delimited with a colon ":" character unless all
    right-most parameters are permitted to default, in which case they may
    be omitted.

    host is a fully qualified hostname or IP address
        This parameter is the first and the only required argument.

    user is the remote login username
        The default is the invoking user if not supplied in the parameter
        filename.

    port is the destination SSH server port number
        This parameter file argument may be a number or an /etc/services
        name. The default is whatever the current hosts' services(5) entry
        for 'ssh/tcp' has configured.

    key is the name of a secret private key (rsa1, rsa2 or dsa)
    authentication file in your ~/.ssh directory
        A default key will be selected unless the command line switch -N is
        used. The default secret key file selection process uses a
        prioritized selection criteria ( if the key file exists ) in ~/.ssh,
        which is:

        id_rsa
                ~/.ssh/id_rsa

        id_dsa
                ~/.ssh/id_dsa

        identity
                ~/.ssh/identity

        Any others
            Other keys are discovered by looking for key filenames ending
            with a .pub extension, in which a secret keyfile with the same
            name (sans the .pub extension) exists.

    version is the protocol version of SSH that the key argument uses (1 or
    2).
        It's usually best to just leave this empty unless you're sure the
        SSH key is protocol version 1. If this is specified in the parameter
        filename just remember that supplying this means that you intend to
        force SSH to require this protocol version (see "-1" in ssh).

    cmd is an optional command argument list to run on the remote host
    (default is a login session)

    When forming the "parameter filename" all right-justified parameters and
    any delimiters may be omitted if the default values are wanted. If a
    right-most parameter needs to be supplied then embed all left-to-right
    intermediary parameters with empty "::" delimiters. In other words
    parameters are identified using a positional argument list delimited
    with colons. Supplying empty colons will use their default values.

  COMMAND LINE SWITCHES
    -a | --add
        An input loop is used to gather all of the input necessary to
        automatically build the "parameter file" and "shortcut" links. No
        extra charge.

    -s | --silent
        The SSH command being spawned is normally echoed to the terminal.
        The echo is suppressed if this command line switch is given, or the
        remote session has command line arguments given (either through the
        parameter file, or if passed to the shortcut). Since shortcuts might
        exist to launch automated processes the echo suppression for command
        arguments makes parsing command output easier.

    -h | --help
        Help!

    -N | --no-defkey
        Normally, if a private authentication key is not specified in the
        parameter filename, or in the ~/.remote-ssh-access file, a default
        private key is selected. Using this switch prevents the default
        identity file from being selected.

        A couple things are worth noting about this option:

        This switch does not prevent a secret key file from being used if it
        is given in the parameter file, or the host/user specific override
        -- it only prevents default key selection of a key.

        The SSH client itself may or may not decide to automatically use a
        key anyway.

    cmd...
        Multiple commands may be passed to a shortcut command.

  INSTALLATION AND USE
   INSTALL THE SCRIPT
    Install and use this script using the following steps

    *   Create a subdirectory under your home named ~/.hosts, or something
        similar.

    *   Add it to your PATH environment variable (and ideally, to your login
        profile)

    *   Install this script inside ~/.hosts, or whatever directory you used
        above.

    *   The script must be named "remote-ssh-access".

        If another name is desired then you must modify the source code:
        change the constant "REAL_PROCESS_NAME" in the source code to
        reflect the new script name.

    *   Do not install the script in a shared system-wide location.

        Do not install this script in a directory such as /usr/local/bin.
        This is because:

        1. Non-privileged users need write access to the same directory.
        2. This script makes use of hard links.
            Often system installation directories live on their own file
            systems. Since this script makes use of hard links, and hard
            links do not span disparate file systems, it would not make
            sense to install the script in a system binary directory.

   CREATE SHORTCUTS THE EASY WAY
    Run this script with the "--add" switch, you will be prompted for all of
    the necessary data. The hard and soft links will be created by this
    script.

   CREATE HARD AND SOFT LINKS TO FORM SHORTCUTS THE HARD WAY
    Hardlink the parameter file to this script given the syntax described
    earlier -- then create a symlink to the hard link and invoke the SSH
    session with the shortcut.

    *   Example 1

        You want a shortcut to remote to the host plethora as the user joe.
        On plethora, sshd runs on port 1234. In addition you would like to
        use the public key associated with id_dsa. Furthermore you would
        like the 'uptime' command to be executed. Allow the SSH protocol
        version to default.

            % cd ~/.hosts
            % ln remote-ssh-access plethora:joe:1234:id_dsa::uptime

        Symlink the parameter file to a shortcut named duptime

            % ln -s plethora:joe:1234:id_dsa::uptime duptime

        Login and run uptime against plethora by invoking the shortcut

            % duptime

    *   Example 2

        You want to connect to the host pinyata as your default user,
        default ssh port, using the SSH v1 public key file named 'identity'
        and have an interactive shell.

            % ln remote-ssh-access pinyata:::identity:1
            % ln -s pinyata:::identity:1 pinyata
            % pinyata

    *   Example 3

        Using all defaults create a shortcut to the host domino. Since the
        command is already sufficiently short you can use it as the shortcut
        (no symlink is required).

            % ln remote-ssh-access domino
            % domino

  OPTIONAL SSH GOODNESS
    The standard SSH suite includes tools for managing sessions. You can
    load your key via ssh-agent under a sub-process (e.g., shell or X11)
    then add the key via ssh-add. Subsequent invocations of the shortcut
    will have the passphrase fed by the agent.

        % ssh-agent bash
        % ssh-add ~/.ssh/identity
        % pinyata

  OVERRIDING PARAMETER-CONFIGURED REMOTE COMMANDS
    If any command argument list is passed to the shortcut it is passed as
    commands to run against the target host -- overriding any command
    argument in the "parameter filename".

        % ln remote-ssh-access dilbert:root:1234:id_dsa:2:who
        % ln -s dilbert:root:1234:id_dsa:2:who dwho
        % dwho      # runs who(1) on dilbert
        % dwho w    # runs the w(1) command on dilbert instead

  OVERRIDING HOST AND USER SPECIFIC KEYS
    You can override keys on a per host/user basis.

    Create a file named ~/.remote-ssh-access. Other than comments
    (introduced with the standard "#" sigil) the file takes the following
    syntax:

    host:user:key:version

    Either host or user may be the wild card (*) character, which means any
    host or user. Note that the wild card does not "glob" identifiers, for
    example "jo*" will not pattern match all users prefixed with "jo". The
    wild card is either '*' or a specific label.

    The key argument can be the name of a secret key file in your ~/.ssh
    directory, or it may be a fully qualified path to the file.

    Note that when an authentication key is overridden you are given a hint
    -- the command echo will prefix the SSH command with a [*] noting that
    the key had been overridden (unless, as explained earlier, echoes are
    suppressed).

    Example
        foo.bar.com:*:id_dsa:2

        This would force any shortcut, which, if symlinked to a file that
        has a host parameter of foo.bar.com to have its secret key
        overridden with ~/.ssh/id_dsa, despite the key specified in the
        parameter file.

    Another nifty example
        *:uploads:identity:1

        Would force all shortcuts that result in remote SSH sessions
        targeted to the "uploads" user to automatically resort to using
        ~/.ssh/identity and SSH protocol version 1.

PREREQUISITES
    This script requires

    "Cwd 3.12",

    "File::Spec::Functions 1.3",

    "File::Basename 2.74",

    "Getopt::Long 2.35", and

    "Pod::Usage 1.33"

    It should be easy to obtain all of these since they are all standard
    Perl core modules.

OSNAMES
    "Linux", "UNIX", "BSD"

SCRIPT CATEGORIES
    Networking UNIX/System_administration

AUTHOR
    Lane Davis <cpan@upt.org>

