---
title: Create & Manage Users
summary: To create and manage your cluster's users (which lets you control SQL-level privileges), use the cockroach user command with appropriate flags.
toc: false
---

To create and manage your cluster's users (which lets you control SQL-level [privileges](privileges.html)), use the `cockroach user` [command](cockroach-commands.html) with appropriate flags.

When creating users, it's also important to note:

- After creating users, you must [grant them privileges to databases and tables](grant.html).
- On secure clusters, users must [authenticate their access to the cluster](#user-authentication).

{{site.data.alerts.callout_info}}You can also create users through the <a href="create-user.html"><code>CREATE USER</code></a> statement.{{site.data.alerts.end}}

<div id="toc"></div>

## Subcommands

Subcommand | Usage 
-----------|------
`get` | Retrieve a table containing a user and their hashed password.
`ls` | List all users.
`rm` | Remove a user.
`set` | Create or update a user.

## Synopsis

~~~ shell
# Create a user:
$ cockroach user set <username> <flags>

# List all users:
$ cockroach user ls <flags>

# Display a specific user:
$ cockroach user get <username> <flags>

# View help:
$ cockroach user --help
$ cockroach user get --help
$ cockroach user ls --help
$ cockroach user rm --help
$ cockroach user set --help
~~~

## Flags

The `user` command and subcommands support the following flags, as well as [logging flags](cockroach-commands.html#logging-flags). 

Flag | Description
-----|------------
`--ca-cert` | The path to the [CA certificate](create-security-certificates.html). This flag is required if the cluster is secure.<br><br>**Env Variable:** `COCKROACH_CA_CERT`
`--cert` | The path to the [client certificate](create-security-certificates.html) of the user *issuing the command* (not the user you're creating). This flag is required if the cluster is secure. <br><br>**Env Variable:** `COCKROACH_CERT`
`-d`, `--database` | _Deprecated_: Users are created for the entire cluster. However, you can control a user's privileges per database when [granting them privileges](grant.html#grant-privileges-on-databases). <br/><br/>**Env Variable:** `COCKROACH_DATABASE`
`--host` | Database server host to connect to.<br/><br/>**Env Variable:** `COCKROACH_HOST`
`--insecure` | Set this only if the cluster is insecure and running on multiple machines. <br><br>If the cluster is insecure and local, leave this out. If the cluster is secure, leave this out and set the `--ca-cert`, `--cert`, and `--key` flags. <br><br>**Env Variable:** `COCKROACH_INSECURE`
`--key` | Path to the [client key](create-security-certificates.html) protecting the client certificate of the user *issuing the command* (not the user you're creating). This flag is required if the cluster is secure. <br/><br/>**Env Variable:** `COCKROACH_KEY`
`--password` | Enable password authentication for the user; you will be prompted to enter the password on the command line.<br/><br/>[Find more detail about how CockroachDB handles passwords](#user-authentication).
`-p`, `--port` | Connect to the cluster on the specified port.<br/><br/>**Env Variable:** `COCKROACH_PORT` <br/>**Default**: `26257`
`--pretty` | Format tables using ASCII. When not specified, table rows are printed as tab-separated values (TSV). <br/><br/>**Default**: `true`
`--url` | Connect to the cluster on the provided URL, e.g., `postgresql://myuser@localhost:26257/mydb`. If left blank, the connection flags are used (`host`, `port`, `user`, `database`, `insecure`, `certs`). <br/><br/>**Env Variable:** `COCKROACH_URL`
`-u`, `--user` | _Deprecated_: Only the `root` user can create users, so you cannot pass any other usernames into this flag. <br/><br/>**Env Variable:** `COCKROACH_USER` <br/>**Default**: `root`

## User Authentication

Secure clusters require users to authenticate their access to databases and tables. CockroachDB offers two methods for this:

- [Client certificate and key authentication](#secure-clusters-with-client-certificates), which is available to all users. To ensure the highest level of security, we recommend only using client certificate and key authentication.
- [Password authentication](#secure-clusters-with-passwords), which is available only to users with passwords. To set a password for a user, include the `--password` flag in the `cockroach user set` command. <br/><br/>You can use this password to authenticate users without supplying their client certificate and key; however, we recommend instead using client certificate and key authentication whenever possible.

{{site.data.alerts.callout_info}}Insecure clusters do not support user authentication, but you can still create passwords for users through the <code>--password</code> flag.{{site.data.alerts.end}}

## Examples

### Create a User

#### Insecure Cluster

~~~ shell
$ cockroach user set jpointsman
~~~

After creating users, you must [grant them privileges to databases](grant.html).

#### Secure Cluster

~~~ shell
$ cockroach user set jpointsman \
--ca-cert=certs/ca.cert --cert=certs/root.cert --key=certs/root.key --password
~~~

{{site.data.alerts.callout_success}}If you want to allow password authentication for the user, include the <code>--password</code> flag and then enter and confirm the password at the command prompt.{{site.data.alerts.end}}

After creating users, you must [grant them privileges to databases](grant.html).

### Authenticate as a Specific User

#### Insecure Clusters

~~~ shell
$ cockroach sql --user=jpointsman
~~~

#### Secure Clusters with Client Certificates

All users can authenticate their access to a secure cluster using [a client certificate](create-security-certificates.html#create-the-certificate-and-key-for-a-client) issued to their username.

~~~ shell
$ cockroach sql --user=jpointsman --ca-cert=certs/ca.cert --cert=jpointsman.cert --key=jpointsman.key
~~~

#### Secure Clusters with Passwords

[Users with passwords](create-and-manage-users.html#secure-cluster) can authenticate their access by entering their password at the command prompt instead of using their client certificate and key.

~~~ shell
$ cockroach sql --user=jpointsman --ca-cert=certs/ca.cert
~~~

After issuing this command, you must enter the password for `jpointsman` twice.

### Update a User's Password

~~~ shell
$ cockroach user set jpointsman \
--password \
--ca-cert=certs/ca.cert --cert=certs/root.cert --key=certs/root.key
~~~

After issuing this command, enter and confirm the user's new password at the command prompt.

### List All Users

~~~ shell
$ cockroach user ls
~~~
~~~
+------------+
|  username  |
+------------+
| jpointsman |
+------------+
~~~

### Find a Specific User

~~~ shell
$ cockroach user get jpointsman
~~~
~~~
+------------+--------------------------------------------------------------+
|  username  |                        hashedPassword                        |
+------------+--------------------------------------------------------------+
| jpointsman | $2a$108tm5lYjES9RSXSKtQFLhNO.e/ysTXCBIRe7XeTgBrR6ubXfp6dDczS |
+------------+--------------------------------------------------------------+
~~~

### Remove a User

~~~ shell
$ cockroach user rm jpointsman
~~~

## See Also

- [Create Security Certificates](create-security-certificates.html)
- [`GRANT`](grant.html)
- [`SHOW GRANTS`](show-grants.html)
- [`CREATE USER`](create-user.html)
- [Other Cockroach Commands](cockroach-commands.html)
