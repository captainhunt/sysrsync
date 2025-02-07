# sysrsync
Simple and safe native rsync wrapper for Python 3

[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=gchamon_sysrsync&metric=alert_status)](https://sonarcloud.io/dashboard?id=gchamon_sysrsync)

## Requirements

* rsync
* python 3.6+

**development**:

* poetry (be sure to have both poetry and pip upgraded to the latest version)

## Installation

`pip install sysrsync`

## Basic rules

- Syncs source contents by default, so it adds a trailing slash to the end of source, unless `sync_source_contents=False` is specified
- Removes trailing slash from destination
- Extra arguments are put right after `rsync`
- Breaks if `source_ssh` and `destination_ssh` are both set

## Usage

* Basic file sync

```python
import sysrsync

sysrsync.run(source='/home/user/foo.txt',
             destination='/home/server/bar')
# runs 'rsync /home/users/foo.txt /home/server/files'
```

* sync whole folder

```python
import sysrsync

sysrsync.run(source='/home/user/files',
             destination='/home/server/',
             sync_source_contents=False)
# runs 'rsync /home/user/files /home/server'
```

* sync folder contents

```python
import sysrsync

sysrsync.run(source='/home/user/files',
             destination='/home/server/',
             sync_source_contents=True)
# runs 'rsync /home/user/files/ /home/server'
```

* ssh with options

```python
import sysrsync

sysrsync.run(source='/home/user/files',
             destination='/home/server/files',
             destination_ssh='myserver',
             options=['-a'])
# runs 'rsync -a /home/users/files/ myserver:/home/server/files'
```

* exclusions

```python
import sysrsync

sysrsync.run(source='/home/user/files',
             destination='/home/server/files',
             destination_ssh='myserver',
             options=['-a'],
             exclusions=['file_to_exclude', 'unwanted_file'])
# runs 'rsync -a /home/user/files/ myserver:/home/server/files --exclude file_to_exclude --exclude unwanted_file'
```
* Private key

```python
import sysrsync

sysrsync.run(source='/home/user/files',
             destination='/home/server/files',
             destination_ssh='myserver',
             private_key="totally_secure_key")
# runs 'rsync --rsh='ssh -i totally_secure_key' /home/user/files/ myserver:/home/server/files'
```

## API

`sysrsync.run`

| argument  | type | default | description |
| --------- | ---- | ------- | ----------- |
| cwd  | str  | `os.getcwd()` | working directory in which subprocess will run the rsync command |
| strict  | bool | `True` | raises `RsyncError` when rsync return code is different than 0  |
| verbose | bool | `False` | verbose mode: currently prints rsync command before executing |
| **kwargs | dict | Not Applicable | arguments that will be forwarded to call to `sysrsync.get_rsync_command` |

**returns**: `subprocess.CompletedProcess`

**raises**:
- `RsyncError` when `strict = True` and rsync return code is different than 0 ([Success](https://lxadm.com/Rsync_exit_codes#List_of_standard_rsync_exit_codes))

`sysrsync.get_rsync_command`

| argument  | type | default | description |
| --------- | ---- | ------- | ----------- |
| source | str | - | Source folder or file |
| destination | str | - | Destination folder |
| source_ssh | Optional[str] | None | Remote ssh client where source is located |
| destination_ssh | Optional[str] | None | Remote ssh client where destination is located |
| exclusions | Optional[Iterable[str]] | None | List of excluded patterns as in rsync's `--exclude` |
| sync_source_contents | bool | True | Abstracts the elusive trailing slash behaviour that `source` normally has when using rsync directly, i.e. when a trailing slash is present in `source`, the folder's content is synchronized with destination. When no trailing slash is present, the folder itself is synchronized with destination. |
| options | Optional[Iterable[str]] | None | List of options to be used right after rsync call, e.g. `['-a', '-v']` translates to `rsync -a -v` |
| private_key | Optional[str] | None | Configures an explicit key to be used with rsync --rsh command |
| rsync_binary | Optional[str] | rsync | The rsync binary to call |
| **kwargs | dict | Not Applicable | arguments that will be forwarded to call to `subprocess.Popen` |

**returns**: `List[str]` -> the compiled list of commands to be used directly in `subprocess.run`

**raises**:
- `RemotesError` when both `source_ssh` and `destination_ssh` are set. Normally linux rsync distribution disallows source and destination to be both remotes.
- `PrivateKeyError` when `private_key` doesn't exist

# Contributing

- Fork project
- Install dependencies with `poetry install`
- Make changes
- Lint with `poetry run pylint ./sysrsync`
- Test with `poetry run python -m nose tests/*`
- Submit changes with a pull request
