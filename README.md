[![build status](https://github.com/alecunsolo/ansible-role-chezmoi/actions/workflows/ci.yml/badge.svg)](https://github.com/alecunsolo/ansible-role-chezmoi/actions/workflows/ci.yml)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit)](https://github.com/pre-commit/pre-commit)

Ansible Role: chezmoi
=========

## DISCLAIMER
After [this](https://www.redhat.com/en/blog/furthering-evolution-centos-stream) announcement I will not test on RHEL anymore.

---------

An ansible role role that installs and initializes [chezmoi](https://github.com/twpayne/chezmoi).
In addition to the main application, the module creates a systemd service and timer to be used to setup auto updates of dotfiles for the users.

Requirements
------------

`git` should be installed before.

Role Variables
--------------

Available variables and their defaults values are listed in [defaults/main.yml](defaults/main.yml).

```yaml
chezmoi_repo: twpayne/chezmoi
chezmoi_version: latest
```
The repository and the version of `chezmoi` that will be installed.

```yaml
chezmoi_users: []
# chezmoi_users:
#   - user: XXX
#     repo: YYY [optional: dotfiles repository]
#     start_timer: bool [optional, default 'false']
```
The list of the local users and their dotfiles repositories. For each user in the list, if not already initialized, will be executed:
```sh
chezmoi init --apply <REPO>
```

`start_timer` controls whether the systemd timer that keeps the dotfiles repo
in sync is started and enabled for each user. When omitted it falls back to
`chezmoi_update_timer_default`.

```yaml
chezmoi_update_cron: hourly
```
Set the `OnCalendar` parameter for the systemd timer unit. This value is the same for all the users.

```yaml
chezmoi_update_timer_default: true
```
Timer state for users that don't set `start_timer`. `true` enables and starts
the per-user timer (dotfiles self-heal on the `chezmoi_update_cron` schedule);
`false` stops and disables it. The timer is configured idempotently either way,
so flipping this brings existing hosts into the chosen state on the next run.

```yaml
chezmoi_update_now: true
```
When true, the role forces an immediate `chezmoi update` for each user at the
end of the run, so a re-provisioned host converges to the current dotfiles
right away rather than waiting for the next timer tick.

### Forcing a sync by hand

The role installs `/usr/local/bin/chezmoi-sync`:

```sh
chezmoi-sync [user]   # defaults to the invoking user
```

It triggers the same systemd unit the timer uses
(`chezmoi-update@<user>.service`). Because systemd never runs two instances of
one unit at once, a manual sync can never collide with a scheduled one — if a
timed run is in flight, the manual run joins it. The service additionally wraps
`chezmoi update` in `flock`, so even a raw `chezmoi update` typed in a shell is
serialized against the scheduled run.

Dependencies
------------

None.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: all
  vars:
    chezmoi_users:
    - user: my_user
      repo: my_dotfile_repo
      start_timer: true
  roles:
  - alecunsolo.chezmoi
```
License
-------

MIT

Notes
-----

Testing with molecule (including the docker images used) is ~~stolen from~~ heavily inspired by [Jeff Geerling](https://www.jeffgeerling.com/). Watch [his video](https://youtu.be/FaXVZ60o8L8) (and the other ones as well).
