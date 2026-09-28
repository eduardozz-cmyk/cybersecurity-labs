# Linux Lab 01 — User & Permission Management

## Objective

Practice basic Linux user, group, ownership, and permission management by creating users and groups and configuring access to a shared project directory.

## Environment

* OS: Ubuntu Linux
* Environment: Virtual Machine
* Users: `alice`, `bob`, `charlie`
* Groups: `developers`, `sysadmins`, `project`

## Scenario

I was managing a Linux server for a fictional company called **CyberLab**.

The company had three users:

* `alice` — Developer
* `bob` — System Administrator
* `charlie` — Intern

The goal was to create their accounts and configure a project directory so that Alice and Bob could access it, while Charlie could not.

## 1. Creating Users

I created the three users using `adduser`:

```bash
sudo adduser alice
sudo adduser bob
sudo adduser charlie
```

I verified that the users existed by checking `/etc/passwd`:

```bash
cat /etc/passwd
```

## 2. Creating Groups

I created two groups for the users:

```bash
sudo groupadd developers
sudo groupadd sysadmins
```

I then added Alice to the `developers` group:

```bash
sudo usermod -aG developers alice
```

And Bob to the `sysadmins` group:

```bash
sudo usermod -aG sysadmins bob
```

I used `su` and `groups` to switch between users and verify their group memberships.

## 3. Creating the Project Group

I created a separate group specifically for the shared project:

```bash
sudo addgroup project
```

I added Alice and Bob to the `project` group:

```bash
sudo usermod -aG project alice
sudo usermod -aG project bob
```

Charlie was not added to this group.

The purpose of this group was to allow Alice and Bob to share access to the project directory.

## 4. Creating the Directory

I created the project directory structure using:

```bash
sudo mkdir -p /opt/cyberlab/project
```

The `-p` option allows `mkdir` to create the necessary parent directories if they do not already exist.

The resulting structure was:

```text
/opt
└── cyberlab
    └── project
```

## 5. Checking Ownership and Permissions

I checked the directory's current ownership and permissions with:

```bash
ls -ld /opt/cyberlab/project
```

Initially, the directory was owned by:

```text
root root
```

The permissions were:

```text
drwxr-xr-x
```

This meant that the directory belonged to the `root` user and `root` group, while other users still had access.

## 6. Changing Ownership

I changed the directory's group from `root` to `project` while keeping `root` as the owner:

```bash
sudo chown root:project /opt/cyberlab/project
```

The ownership became:

```text
Owner: root
Group: project
```

This was important because Alice and Bob were members of the `project` group.

### Understanding `chown`

`chown` is used to change the ownership of a file or directory.

In this lab:

```text
root:project
│    │
│    └── Group
└─────── Owner
```

So:

**`chown` → controls who owns the file/directory.**

## 7. Configuring Permissions

I then changed the directory permissions:

```bash
sudo chmod 770 /opt/cyberlab/project
```

The `770` permission represents:

```text
Owner   Group   Others
  7       7       0
 rwx     rwx     ---
```

Therefore:

* Owner (`root`) → `rwx`
* Group (`project`) → `rwx`
* Others → no permissions

### Understanding `chmod`

`chmod` changes the permissions of a file or directory.

In this lab:

**`chmod` → controls what the owner, group, and others can do.**

## 8. Testing Access

I tested the configuration by switching between the three users.

### Alice

Alice was able to enter the project directory:

```bash
su - alice
cd /opt/cyberlab/project
```

She was also able to create a file:

```bash
touch test-alice.txt
```

This confirmed that Alice had write access through the `project` group.

### Bob

Bob was also able to enter the directory and create a file:

```bash
su - bob
cd /opt/cyberlab/project
touch test-bob.txt
```

This confirmed that Bob also had write access through the `project` group.

### Charlie

Charlie was not a member of the `project` group.

When I tried:

```bash
su - charlie
cd /opt/cyberlab/project
```

access was denied.

This confirmed that the permission configuration was working as intended.

## Troubleshooting

During the lab, I initially tried to create the `/opt/cyberlab/project` directory while logged in as Alice.

Alice could not use `sudo` because she was a regular user and had not been given administrative privileges.

I exited the Alice session:

```bash
exit
```

and returned to my normal administrative Ubuntu account. I was then able to create the directory using `sudo`.

This helped me understand the difference between a normal Linux user and an administrative user.

## What I Learned

* How to create Linux users with `adduser`.
* How to create groups with `groupadd` and `addgroup`.
* How to add users to supplementary groups using `usermod -aG`.
* How to switch between users using `su`.
* How to create directory structures with `mkdir -p`.
* How to check ownership and permissions with `ls -ld`.
* How `chown` changes ownership and group association.
* How `chmod` controls permissions.
* How Linux permissions are divided into **owner, group, and others**.
* How group membership can be used to control access to shared resources.
* How directory permissions affect whether a user can enter and modify a directory.
* Why administrative operations may require `sudo`.

## Key Takeaway

The main concept I learned from this lab was the relationship between **groups, ownership, and permissions**.

```text
Directory
   │
   ├── Owner → root
   │
   ├── Group → project
   │       ├── alice
   │       └── bob
   │
   └── Others → charlie
```

With:

```text
rwx rwx ---
```

Alice and Bob could access and modify the project, while Charlie was denied access.

