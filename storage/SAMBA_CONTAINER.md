# Samba Container Runbook

This stack runs Samba using the [dockurr/samba](https://hub.docker.com/r/dockurr/samba)
image from `storage/docker-compose.yaml` with `storage/smb.conf` as the configuration.

## What gets mounted

- `./smb.conf -> /etc/samba/smb.conf:ro` (share definitions)
- `./samba/users.conf -> /etc/samba/users.conf:ro` (user bootstrap, see below)
- `samba-data -> /var/lib/samba` (persistent credential database)
- `/mnt -> /mnt:rw` (share roots used by your config)
- `/etc/passwd -> /etc/passwd:ro` (host user resolution)
- `/etc/group -> /etc/group:ro` (host group resolution)

Mounting `/etc/passwd` and `/etc/group` allows Samba in the container to
resolve host groups such as `camp-ro`, `camp-rw`, and `wheel` referenced in `smb.conf`.

The `samba-data` volume persists the Samba password database, so credentials
survive container restarts and recreations.

## User management

Credentials are stored as hashes in the persistent `samba-data` volume. The
`users.conf` file is only used for initial bootstrap or adding new users.

### Adding users via users.conf (bootstrap)

Add users to `storage/samba/users.conf` with the format:

```
username:UID:groupname:GID:password
```

Example:
```
alice:1001:camp-rw:1002:secretpass
```

To find existing UIDs/GIDs on the host:
```bash
getent passwd username
getent group groupname
```

After adding users, restart the container:
```bash
docker compose -f storage/docker-compose.yaml restart samba
```

Once users are created, **remove the plaintext passwords** from `users.conf`
to avoid storing credentials in plain text. Keep just the user:UID:group:GID
portion (the container won't re-create existing users):

```
alice:1001:camp-rw:1002:
```

### Adding users interactively (preferred)

To avoid plaintext passwords entirely, add users directly in the container:

```bash
docker compose -f storage/docker-compose.yaml exec samba smbpasswd -a username
```

This prompts for the password interactively and stores only the hash.

### Changing passwords

```bash
docker compose -f storage/docker-compose.yaml exec samba smbpasswd username
```

### Listing users

```bash
docker compose -f storage/docker-compose.yaml exec samba pdbedit -L
```

## Migration from native Fedora Samba

1. Stop and disable native Samba services:
   ```bash
   sudo systemctl disable --now smb nmb
   ```

2. List existing Samba users on the host:
   ```bash
   sudo pdbedit -L -v
   ```

3. Start the containerized Samba:
   ```bash
   docker compose -f storage/docker-compose.yaml up -d samba
   ```

4. Recreate each user interactively (avoids plaintext passwords):
   ```bash
   docker compose -f storage/docker-compose.yaml exec samba smbpasswd -a username
   ```

5. Verify users were created:
   ```bash
   docker compose -f storage/docker-compose.yaml exec samba pdbedit -L
   ```

## Start/stop containerized Samba

- Start: `docker compose -f storage/docker-compose.yaml up -d samba`
- Follow logs: `docker compose -f storage/docker-compose.yaml logs -f samba`
- Stop: `docker compose -f storage/docker-compose.yaml stop samba`

## Verification checklist

1. Confirm ports are bound on host network:
   ```bash
   sudo ss -tulpen | grep -E ':(137|138|139|445)\b'
   ```

2. Confirm config parses in-container:
   ```bash
   docker compose -f storage/docker-compose.yaml exec samba testparm -s /etc/samba/smb.conf
   ```

3. Confirm share visibility/auth from a client:
   - `CampFitch`: authenticated read/write for `@camp-ro` and `@camp-rw`
   - `Photos`: authenticated read/write for `@camp-rw`
   - `Deploy`: `write list = @wheel` behavior is enforced

4. Restart and re-test auth persistence:
   ```bash
   docker compose -f storage/docker-compose.yaml restart samba
   ```

## Rollback to native Samba

1. Stop containerized Samba:
   ```bash
   docker compose -f storage/docker-compose.yaml stop samba
   ```

2. Re-enable native services:
   ```bash
   sudo systemctl enable --now nmb smb
   ```

3. Validate native service health:
   ```bash
   sudo systemctl status smb nmb
   ```
