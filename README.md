# Logfile::Tail and logfile-tail

## Logfile::Tail - read log files

### SYNOPSIS

        use Logfile::Tail ();
        my $file = new Logfile::Tail('/var/log/messages');
        while (<$file>) {
                # process the line
        }

and later in different process

        my $file = new Logfile::Tail('/var/log/messages');

and continue reading where we've left out the last time. Also possible
is to explicitly save the current position:

        my $file = new Logfile::Tail('/var/log/messages',
                { autocommit => 0 });
        my $line = $file->getline();
        $file->commit();

### DESCRIPTION

Log files are files that are generated by various running programs.
They are generally only appended to. When parsing information from
log files, it is important to only read each record / line once,
both for performance and for accounting and statistics reasons.

The `Logfile::Tail` provides an easy way to achieve the
read-just-once processing of log files.

The module remembers for each file the position where it left
out the last time, in external status file, and upon next invocation
it seeks to the remembered position. It also stores checksum
of 512 bytes before that position, and if the checksum does not
match the file content the next time it is read, it will try to
find the rotated file and read the end of it before advancing to
newer rotated file or to the current log file.

Both .num and -date suffixed rotated files are supported.

### METHODS

- new()
- new( FILENAME \[,MODE \[,PERMS\]\], \[ { attributes } \] )
- new( FILENAME, IOLAYERS, \[ { attributes } \] )

    Constructor, creates new `Logfile::Tail` object. Like `IO::File`,
    it passes any parameters to method `open`; it actually creates
    an `IO::File` handle internally.

    Returns new object, or undef upon error.

- open( FILENAME \[,MODE \[,PERMS\]\], \[ { attributes } \] )
- open( FILENAME, IOLAYERS, \[ { attributes } \] )

    Opens the file using `IO::File`. If the file was read before, the
    offset where the reading left out the last time is read from an
    external file in the ./.logfile-tail-status directory and seek is
    made to that offset, to continue reading at the last remembered
    position.

    If however checksum, which is also stored with the offset, does not
    match the current content of the file (512 bytes before the offset
    are checked), the module assumes that the file was rotated / reused
    / truncated in the mean time since the last read. It will try to
    find the checksum among the rotated files. If no match is found,
    it will reset the offset to zero and start from the beginning of
    the file.

    Returns true, or undef upon error.

    The attributes are passed as an optional hashref of key => value
    pairs. The supported attribute is

    - autocommit

        Value 0 means that no saving takes place; you need to save explicitly
        using the commit() method.

        Value 1 (the default) means that position is saved when the object is
        closed via explicit close() call, or when it is destroyed. The value
        is also saved upon the first open.

        Value 2 causes the position to be save in all cases as value 1,
        plus after each successful read.

    - status\_dir

        The attribute specifies the directory (or subdirectory of current
        directory) which is used to hold status files. By default,
        ./.logfile-tail-status directory is used. To store the status
        files in the current directory, pass empty string or dot (.).

    - status\_file

        The attribute specifies the name of the status file which is used to
        hold the offset and SHA256 checksum of 512 bytes before the offset.
        By default, SHA256 of the full (absolute) logfile filename is used
        as the status file name.

- commit()

    Explicitly save the current position and checksum in the status file.

    Returns true, or undef upon error.

- close()

    Closes the internal filehandle. It stores the current position
    and checksum in an external file in the ./.logfile-tail-status
    directory.

    Returns true, or undef upon error.

- getline()

    Line &lt;$fh> in scalar context.

- getlines()

    Line &lt;$fh> in list context.

### AUTHOR AND LICENSE

Copyright (c) 2010--2024 Jan Pazdziora.

Logfile::Tail is free software. You can redistribute it and/or modify
it under the terms of either:

a) the GNU General Public License, version 2 or 3;

b) the Artistic License, either the original or version 2.0.

***

## logfile-tail - output (cat) file from the last saved position

### SYNOPSIS

    logfile-tail [ --status=status-directory | --status=status-file ] logfile
    logfile-tail --help

### DESCRIPTION

When processing log files, you want to continue reading where you
left out the last time. The **logfile-tail** program uses the
**Logfile::Tail** module internally to store the position last
seen for the log file and retrieve it upon the subsequent
invocation.

The program also handles rotated files -- if the log file was
rotated since the last read, it is detected and the rest of the
rotated file is read first, before proceeding to the newer
rotate file or to the current log file.

The content is printed to the standard output.

### OPTIONS

- --status=STATUS DIRECTORY | --status=STATUS FILE

    The parameter specifies either the status file which is used
    to store the position, or directory which will hold the status file.
    The file has to already exist (albeit empty) for the path to be
    recognized as status file, otherwise it is considered to be
    a status directory path.

- --help

    Short usage is printed.

### EXAMPLES

    # output data from Apache's access_log
    logfile-tail /var/log/httpd/access_log

    logfile-tail --status /var/run/apache/logfile-tail error_log

### AUTHOR

Copyright (C) 2011--2024 by Jan Pazdziora

### LICENSE

This module is free software.  You can redistribute it and/or
modify it under the terms of the Artistic License 2.0.

This program is distributed in the hope that it will be useful,
but without any warranty; without even the implied warranty of
merchantability or fitness for a particular purpose.
