# Users and Groups

When shadowing is enabled, the `password` field in the password and group file will be marked as `x`. All of the encrypted passwords are stored in the shadow password file.

| Description          | Location        | Format                                                                         |
|----------------------|-----------------|--------------------------------------------------------------------------------|
| Password file        | `/etc/passwd`   | `<username>:<password>:<uid>:<gid>:<comment>:<home>:<shell>`                   |
| Group file           | `/etc/group`    | `<groupname>:<password>:<gid>:<users>`                                         |
| Shadow password file | `/etc/shadow`   | `<username>:<password>:<last-changed>:<min>:<max>:<warn>:<inactive>:<expired>` |

The `password` field in the shadow password file contains an encrypted passoword. Its format can be:

- `$id$salt$encrypted`
- `$id$rounds=yyy$salt$encrypted` (since glibc 2.7)

Here is an example, see [`crypt`](https://man7.org/linux/man-pages/man3/crypt.3.html) in detail:

```c
// MD5 with a salt 'mysalt'
printf("%s\n", crypt("P@ssw0rd", "$1$mysalt$"));  // $1$mysalt$LSxioYd1q6Rl2LG.3IXwm/

// SHA-256 with a salt 'mysalt'
printf("%s\n", crypt("P@ssw0rd", "$5$mysalt$"));  // $5$mysalt$lQvd3eETPGyCrL3GjXojml9LqHjaW8FHyw3Yfoimej9

// SHA-512 with a salt 'mysalt'
printf("%s\n", crypt("P@ssw0rd", "$6$mysalt$"));  // $6$mysalt$45ntUCBpDXhrwoe9WXVnehWwJlop2aG2j1IWdm5hW06OBiAyP/dpu4eMlgAq7EFHzSy7OUCathAiDUOPgjtuA/
```
