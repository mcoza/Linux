# 5. Users and Permissions

These exercises introduced Linux identity and access as another troubleshooting layer. The main lesson was that a command can be correct and a file or service can exist, while the operation still fails because the current user does not have the required access.

## Questions I learned to separate

```text
Who am I running as?
Who owns this file or directory?
Which group is involved?
What permissions are set?
Does the required user or group actually exist?
```

Useful inspection commands included:

```bash
id
ls -l
```

`id` provides the current user's UID, GID, and group membership. `ls -l` exposes file ownership and permission bits.

## Applied work

I completed exercises involving:

- creating a required user
- troubleshooting access between users who needed to work with the same files
- correcting file or directory permissions when access did not match the requirement
- distinguishing ownership/group membership from the read, write, and execute permissions applied to the object

The exact command trail for every permissions exercise was not retained, so I am not reconstructing specific `chmod`, `chown`, or user-management commands that I cannot verify from my notes.

## How this connected to later troubleshooting

Permissions became more useful once they stopped being a standalone topic. They can appear inside other failure paths:

```text
service or script fails
      ↓
path exists
      ↓
process is running as a particular user
      ↓
ownership / group / mode does not permit the operation
```

That means a `permission denied` error should push the investigation toward identity and object permissions rather than immediately toward networking, systemd, or application logic.

## What changed in my troubleshooting approach

I learned to treat the executing identity as part of the environment. When something works for one user but not another, or a service can see a file but cannot modify it, checking the user, group, ownership, and permission bits is a more direct next step than changing unrelated configuration.
