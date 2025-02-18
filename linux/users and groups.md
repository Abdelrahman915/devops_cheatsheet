# User and Group Management in Linux

## 1. Creating Users with `useradd`

### a. Create a User with Default Settings

```bash
sudo useradd username
```

- This creates a user with:
  - A default home directory (`/home/username`)
  - A default shell (`/bin/bash`)

### b. Create a User with a Specific UID

```bash
sudo useradd -u 1727 yousuf
```

- Assigns the UID `1727` to the user `yousuf`.

### c. Create a User with a Custom Home Directory

```bash
sudo useradd -d /var/www/yousuf -m yousuf
```

- `-d` sets the home directory to `/var/www/yousuf`.
- `-m` ensures the home directory is created.

### d. Create a User Without a Home Directory

```bash
sudo useradd -M kareem
```

- `-M` prevents the creation of a home directory.

### e. Create a User with a Non-Interactive Shell

```bash
sudo useradd -M -s /sbin/nologin kareem
```

- `-s /sbin/nologin` prevents the user from logging in interactively.

Alternative:

```bash
sudo useradd -M -s /bin/false kareem
```

**Difference between **``** and **``**:**

- `/sbin/nologin`: Displays a message like "This account is currently not available."
- `/bin/false`: Simply denies login without any message.

### f. Create a Temporary User (With Expiry Date)

```bash
sudo useradd -e 2024-01-28 john
```

- `-e 2024-01-28` sets the expiry date to **January 28, 2024**.

## 2. Managing Groups with `groupadd`

### a. Create a New Group

```bash
sudo groupadd nautilus_admin_users
```

### b. Add a User to a Group

```bash
sudo usermod -aG nautilus_admin_users stark
```

- Adds `stark` to the group `nautilus_admin_users`.
- `-aG` appends the user to the group without removing existing group memberships.

### c. Verify Group Membership

```bash
groups stark
```

## 3. Verifying User Details

### a. Check User Information

```bash
id username
```

- Displays UID, GID, and group memberships.

### b. Check User's Home Directory and Shell

```bash
grep username /etc/passwd
```

- Expected output (default setup):
  ```
  username:x:1001:1001::/home/username:/bin/bash
  ```

### c. Verify User Expiry Date

```bash
chage -l username
```

- Expected output:
  ```
  Last password change      : <date>
  Password expires          : <date>
  Account expires           : Jan 28, 2024
  ```

### d. List All Groups

```bash
getent group
```

- Displays all groups available on the system.

### e. List All Users

```bash
getent passwd
```

### f. Switch to Another User

```bash
su - username
```

- Switches to the specified user session.

## 4. Summary Table

| Command                                       | Purpose                                     |
| --------------------------------------------- | ------------------------------------------- |
| `sudo useradd username`                       | Create a user with default settings         |
| `sudo useradd -u 1727 yousuf`                 | Create a user with a specific UID           |
| `sudo useradd -d /var/www/yousuf -m yousuf`   | Create a user with a custom home directory  |
| `sudo useradd -M kareem`                      | Create a user without a home directory      |
| `sudo useradd -M -s /sbin/nologin kareem`     | Create a user with a non-interactive shell  |
| `sudo useradd -e 2024-01-28 john`             | Create a temporary user with an expiry date |
| `sudo groupadd nautilus_admin_users`          | Create a new group                          |
| `sudo usermod -aG nautilus_admin_users stark` | Add a user to a group                       |
| `id username`                                 | Check user details                          |
| `chage -l username`                           | Check user expiry date                      |
| `getent group`                                | List all groups                             |
| `getent passwd`                               | List all users                              |
| `su - username`                               | Switch to another user                      |

---

This documentation covers essential commands for user and group management, including interactive vs. non-interactive shells, home directory options, and verification commands. 🚀

