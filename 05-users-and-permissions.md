# 5. Users and Permissions

This section documents the identity and permission information I can inspect when a Linux operation fails because of access rather than because the file, process, or service is missing.

## What I know

I understand the basic relationship:

```text
process or command
      ↓ runs as
user + groups
      ↓ attempts access to
file or directory
      ↓ kernel checks
owner + group + permission bits
```

The main inspection commands I have used are:

```bash
id
ls -l
```

They answer different questions:

```text
id    → current UID, GID, and group memberships
ls -l → file type, permission bits, owner, group, size, and name
```

## How I read file permissions

A mode such as:

```text
-rw-r--r--
```

breaks down as:

```text
-    → regular file
rw-  → owner permissions
r--  → group permissions
r--  → permissions for everyone else
```

The common numeric values are:

```text
4 = read
2 = write
1 = execute
```

so:

```text
6 = read + write
5 = read + execute
7 = read + write + execute
```

For example:

```bash
chmod 0644 file
```

means:

```text
owner  → read + write
group  → read
others → read
```

## What I can do with this information

### Investigate `permission denied`

A `permission denied` error gives me a reason to inspect identity and access before changing unrelated service or network configuration.

The basic path is:

```text
operation fails with permission denied
→ identify the executing user/group context
→ inspect owner/group/mode on the target object
→ compare actual access with required access
→ make only the necessary permission/ownership change
→ retry the original operation
```

Useful first checks are:

```bash
id
ls -l <path>
```

If the operation is being performed by a service, the executing identity may come from the service configuration or process information rather than from my interactive shell.

### Distinguish ownership from permission bits

I do not treat these as the same thing:

```text
owner/group     → which identity class applies?
permission bits → what that class is allowed to do?
```

A file can have restrictive mode bits even when the expected user owns it, or permissive group bits that do not help if the process is not actually a member of that group.

### Connect permissions to application behavior

A downloaded or generated file may need to be readable by another process after it is created.

For example, a synchronization script ended with:

```bash
chmod 0644 "$DEST"
```

That ensured the resulting file was readable beyond only the owner while keeping write permission limited to the owner.

The important reasoning is not the number `0644` by itself. It is:

```text
who creates the file?
→ who needs to read/write it afterward?
→ what owner/group/mode provides that access?
```

## How symptoms narrow my first inspection

```text
"permission denied"               → executing identity + target permissions
"works for one user, not another" → UID/GID/group membership + owner/group/mode
"can read but cannot modify"      → inspect write permission and ownership/group context
"service can see file but cannot change it" → inspect service identity + file/directory permissions
```

These cues do not tell me the exact fix. They tell me which state to inspect before touching unrelated configuration.

## What this lets me do

I can include identity as part of a troubleshooting path instead of treating permissions as a separate topic:

```text
failed operation
→ executing identity
→ target ownership/mode
→ required access
→ smallest justified correction
→ retry original operation
```
