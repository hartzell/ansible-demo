# Ansible demo

An example/template Ansible site setup.  Follows in the footsteps of
Will Robinson's [Ansible Playbook Structure][structure] and [The
Anatomy of an Ansible Playbook][anatomy] articles.

The various tasks are stubs that just print debugging output and the
playbooks all avoid collecting facts from any of the "machines".

# The problem definition

There are two groups of machines (**a** and **b**) as well as a
singleton (**solo.alerce.com**).  The machines in the two groups are
*coincidentally* in eponymously named subdomains, but that is not
something to count on as the site grows.

1. Add users to each of the machines.  There is a common set of user
   definitions (e.g. name, ssh public key, etc...) across the site but
   different subsets of users get accounts on various machines or
   groups of machines.

2. Sudo should be configured on each of the machines, but different
   subsets of the users are sudoers on various machines or groups of
   machines.

3. The ssh daemon should be configured on each machine.  The ability
   to connect via ssh as root is configurable, defaults to being
   disallowed.  The ability to authenticate with a password is
   configurable, defaults to being disallowed.

   Only **solo.alerce.com** allows root SSH connections.

   Only the machines in group **b** allow password authentication,
   *except* (oddly enough), **c.b.alerce.com**.

4. The solo machine needs to run solo-specific plays.

# The solution

Here's the file layout:

```
.
├── README.md
├── a.yml
├── b.yml
├── group_vars
│   ├── a.yml
│   ├── all.yml
│   └── b.yml
├── host_vars
│   ├── c.b.alerce.com.yml
│   └── solo.alerce.com.yml
├── hosts.ini
├── roles
│   ├── ssh
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   ├── sudo
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   └── users
│       ├── defaults
│       │   └── main.yml
│       └── tasks
│           └── main.yml
├── site.yml
└── solo.yml

12 directories, 17 files

```

The machines are defined and assigned to groups in the `hosts.ini` file:

```ini
solo.alerce.com

[a]
a.a.alerce.com
b.a.alerce.com

[b]
a.b.alerce.com
b.b.alerce.com
c.b.alerce.com
```

The user definitions, shared across the entire "site", are in
`group_vars/all.yml`.

The lists of `users` and `sudoers` are set in either a `group_vars`
(for machines in **a** and **b**) or a `host_vars` file (for
**solo**).

SSH and sudo configurations are set in either a `group_vars` (for
machines in **a** and **b**) or a `host_vars` file (for **solo**).
Password authentication for group **b** is set in `group_vars/b.yml`
*but* that setting for **c.b.alerce.com** is overridden in
`host_vars/c.b.alerce.com.yml`.

All of the roles define default values for their configuration
variables in `roles/*/defaults/main.yml`

Since *all* of the machines do the basic roles (users, ssh, sudo)
those roles have been gathered into a `basics` role.

There is a separate role, `solo` that the solo machine uses.  Its
unique resource (e.g. files, templates) can live within the role so
that the top-level landscape isn't cluttered.

# Running the playbooks

You can safely run the commands below, they don't actually *do*
anything, just print out various messages.

The entire site can be provisioned via:

```sh
ansible-playbook -i hosts.ini site.yml
```

Because the provisioning for the machines in each group (and the solo)
is defined in distinct playbooks there are two ways to provision by
*group*/*solo*:

- by calling it's playbook explicitly, e.g.:

  ```sh
  # for a
  ansible-playbook -i hosts.ini a.yml
  # for b
  ansible-playbook -i hosts.ini b.yml
  # for solo
  ansible-playbook -i hosts.ini solo.yml
  ```

- by restricting the set of hosts, e.g.:

  ```sh
  # for a
  ansible-playbook -i hosts.ini -l a site.yml
  # for b
  ansible-playbook -i hosts.ini -l b site.yml
  # for solo
  ansible-playbook -i hosts.ini -l solo site.yml
  ```

The second form can also be used to run on an individual machine from
one of the groups:

  ```sh
  # for solo
  ansible-playbook -i hosts.ini -l c.b.alerce.com site.yml
  ```

[anatomy]: http://www.oznetnerd.com/anatomy-ansible-playbook/
[dry]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[structure]: http://www.oznetnerd.com/ansible-playbook-structure/
