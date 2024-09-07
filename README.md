<img src="./icon.png" height="96" />

# jmap-backup

This is a Python program to back up messages from your Fastmail JMAP mailbox.

Based on the amazing [work by Nathan Grigg][1] 🙏

## ⚠️ Breaking Change

Versions prior to 1.1 stored configuration as YAML. This has been changed to JSON because Python can read/write it without requiring the PyYAML module.

If you're not comfortable converting your legacy config file to JSON by hand, I suggest using [`yq`][5]:

```sh
yq -p yaml -o json fastmail.yml >fastmail.json
```

## Prerequisites

- a Fastmail API key (get from https://app.fastmail.com/settings/security/tokens)
- Python 3 (`brew install python3` if you're on macOS)
- Python's `requests` module

To get the required modules, either install them in a virtualenv, or globally with:

```shell
PIP_REQUIRE_VIRTUALENV=false python3 -m pip install --break-system-packages requests
```

## Setup

1. Download the latest [release][4] or clone this repo (if you don't know how to do that, click the green **Code** button above, then **Download ZIP**)

2. Copy the `jmap-backup.py` file to a directory in your `$PATH` (I suggest `/usr/local/bin` if you're unsure) and make sure it's executable (`chmod +x jmap-backup.py`)

3. Create a configuration file (JSON) to store your API key, destination directory where the backup will be kept, and other settings. You can create multiple config files to back up different accounts or to keep copies on different storage (local, SMB/NFS etc).

> If you don't specify a config file with the `-c` option, the program will assume a default path of `~/.jmapbackup/fastmail.json`.

A bare minimum config file must contain at least the `dest_dir` and `token` keys, for example:

```js
{
  "dest_dir": "/Volumes/storage/backups/Fastmail",
  "token": "{your_api_key_here e.g. fmu1-xxxxxx...}"
}
```

### Other optional parameters for the config file

| Key           | Description                                                                                                                                                                                                | Example value |
|:------------- |:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |:------------- |
| `delay_hours` | Back up only messages at least this many hours old                                                                                                                                                         | `24`          |
| `not_before`  | Cut off date before which messages will not be backed up                                                                                                                                                   | `2018-06-01`  |
| `pre_cmd`     | Command (and args) to run prior to execution, most often used to mount some remote storage location such as an SMB or NFS share. It is formatted as an array so you can provide additional args as needed. | (see below)   |
| `post_cmd`    | Command to run post-execution (e.g. unmount the share)                                                                                                                                                     | (see below)   |

Example of pre/post commands in config file (`~` chars will be expanded by Python):

```js
{ 
  "pre_cmd": [
    "/sbin/mount", "-t", "smbfs",
    "//luckman212:hunter2@nas/backups", "/mnt/jmap"
  ],
  "post_cmd": [
    "/sbin/umount", "-t", "smbfs", "/mnt/jmap"
  ]
}
```

## Run

```shell
jmap-backup.py -c ~/.jmapbackup/fastmail.json
```

Progress messages will be printed to the console. When the job is finished, you should see your messages in the destination directory, organized in folders in `YYYY-MM` format. The individual messages are saved as standard `.eml` format files with the filename made up of a datestamp, messageid and subject.

This is designed to run quickly and often, so running it daily is no problem and should complete within a minute or two. It's a good idea to stick it in your crontab or set up a LaunchAgent to trigger it at regular intervals. I suggest [LaunchControl][3] (no affiliation) if you're on a Mac and don't want to fiddle about with XML files.

## Verification

Every so often, it's a good idea to run the script with the `--verify` argument. This will be slower, but will thoroughly check that every message in your mailbox exists on the filesystem, and will "fill in the blanks" if any are missing.

## Environment Variables

- Export `JMAP_DEBUG` to `True` to see additional debugging info printed to the console
- You can export `NOT_BEFORE` to override the default of `2000-01-01` or whatever date is specified in the config file

## Good luck

I've been using this script for a few months with good success, but it has been tested on exactly _one_ system! So you may encounter issues. If you do, please [report them][2].


[1]: https://nathangrigg.com/2021/08/fastmail-backup
[2]: https://github.com/luckman212/jmap-backup/issues
[3]: https://www.soma-zone.com/LaunchControl/
[4]: https://github.com/luckman212/jmap-backup/releases/latest
[5]: https://github.com/mikefarah/yq
