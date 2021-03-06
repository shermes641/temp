# Oyssey API with JWT Authorization and Swagger

Based on the code from https://medium.com/@maison.moa/using-jwt-json-web-tokens-to-authorize-users-and-protect-api-routes-3e04a1453c3e

## Run locally
On a win 10 Dev box

    git clone https://shermes641@bitbucket.org/powerhive/odyssey.git
    cd odyssey
    npm install
    npm audit fix
    node server\server
    
The server will be running at  

    localhost:2222/api-docs
    
This should display the Swagger UI:

![Swagger UI](./docs/swag.png?raw=true "Title")

From here you can Login and then get your user data.


## Postman

If you do not want to use Swagger, you can load this Postman collect to test the API

(**NOTE:** the collection does not have a valid username, password or token, see [Valid User](models/user_model.js) )

[Postman collection](./Odyssey.postman_collection_v2.1.json)  


## Deploy

SSH into the Timescale AWS server @ 52.37.205.144

(**NOTE:** you will need the swarm.pem file, the same key used on the SWARM AWS server)

    git clone https://shermes641@bitbucket.org/powerhive/odyssey.git
    cd odyssey
    chmod +x *.sh
    dos2unix *.sh
    npm install
    node server/server

##NOTES

The JWT token is only good for 10 minutes

----

Postgrest 6.0.1 is installed in 

    /usr/local/bin
    
To install:

    sudo su
    cd /usr/local/bin
    wget https://github.com/PostgREST/postgrest/releases/download/v6.0.1/postgrest-v6.0.1-ubuntu.tar.xz
    tar xf postgrest-v6.0.1-ubuntu.tar.xz
    
    
To run:

    cd /usr/local/bin
    postgrest jwt.conf
    
Output:
    
    Attempting to connect to the database...
    Listening on port 3333
    Connection successful
    
 ###CRON
 
    crontab -l
    
 Output:
 
    .............................................
    */1 * * * * /home/ubuntu/odyssey/pgr.sh start > /dev/null
    */1 * * * * /home/ubuntu/odyssey/api.sh start > /dev/null
    
 pgr.sh keeps postgrest running
 api.sh keeps the API app running   
   
### TimescaleDb

    user: tsuser      
    pw:   eun4MejkZYGY8VZ98bkVeFg3G9wsEBBd

Connect as root
    
    sudo -u postgres psql -p 5444 -d odyssey
    
Change the password    
    
    \password tsuser    # to change password
    
Misc

    SELECT create_hypertable('vehicle_traffic', 'time',
        chunk_time_interval => interval '1 day');
        
https://medium.com/@aiven_io/timescaledb-101-the-why-what-and-how-9c0eb08a7c0b
    
    



Design

For meter data Odyssey already has a Spark meter integration, so no work should be required there. For Hexing or Asali meters we will need a new DB and the associated code to populate the data and provide API access.

Postgrest is well suited to quickly build API support to any PostgreSQL DB. It also is integrated with  Timescale DB, which provides faster querying of time series data. 

AWS setup

Launch Timescale DB AMI  

us-west-2 (Oregon)

ami-0140204580a0d2481

Configure AWS server  . 

Note this might change as requirements change. We will probably also use this DB for reporting and querying, and potentially as our full API gateway.

Update Linux packages. 

SSH into the AWS timescaledbserver EC2 Server

sudo su

apt update

apt upgrade

apt autoremove

Install GoLang

dd

Run Timescale Tune  . PostgreSQL config file after tuning    

Edit PostgreSQL config files https://stackoverflow.com/questions/14545298/connect-to-remote-postgresql-server-on-amazon-ec2

Restart PostgreSQL sudo service postgresql restart

Create a user  Create User (actual user not shown here)  
















    
    
    
    
    
    
    
    
###Postgrest    


ubuntu@ip-172-31-47-137:~/odyssey$ cat /usr/local/bin/jwt.conf

      db-uri = "postgres://tsuser:eun4MejkZYGY8VZ98bkVeFg3G9wsEBBd@localhost:5444/odyssey"
        db-schema = "public" # this schema gets added to the search_path of every request
        jwt-secret = "BB838C3E31F4A7C82BEB9FD2D7231D82C4E07559565412420F2DE26968EA35FE3MBGOLD0li6Vex7p9NvcAw72lMppiSVeMqAEEGVspza1GJZYqftFutI7nkvrbE2"
        db-anon-role = "tsuser"
        db-pool = 10
        db-pool-timeout = 10
        max-rows = 10000
      
        server-host = "!4"
        server-port = 3333
      
        ## unix socket location
        ## if specified it takes precedence over server-port
        # server-unix-socket = "/tmp/pgrst.sock"
      
        ## base url for swagger output
        # server-proxy-uri = ""
      
        ## choose a secret, JSON Web Key (or set) to enable JWT auth
        ## (use "@filename" to load from separate file)
        # jwt-secret = "foo"
        # secret-is-base64 = false
        # jwt-aud = "your_audience_claim"
      
        ## limit rows in response
        # max-rows = 1000
      
        ## stored proc to exec immediately after auth
        # pre-request = "stored_proc_name"
      
        ## jspath to the role claim key
        # role-claim-key = ".role"
      
        ## extra schemas to add to the search_path of every request
        # db-extra-search-path = "extensions, util"
      
        ## stored proc that overrides the root "/" spec
        ## it must be inside the db-schema
        # root-spec = "stored_proc_name"
      
        ## content types to produce raw output
        # raw-media-types=["image/png","image/jpg"]



##TODO

Remove valid username and password from Swagger

Set cron job to keep the app running (Postrest also)

Add real endpoints that get data from Postgres directly or Postrest

Use TimescalDb or not (probably will to test out production readiness for new API)

Document in Confuence

Filtering Rows

You can filter result rows by adding conditions on columns, each condition a query string parameter. For instance, to return people aged under 13 years old:

GET /people?age=lt.13

Adding multiple parameters conjoins the conditions:

GET /people?age=gte.18&student=is.true

These operators are available:

abbreviation	meaning

    eq	equals
    gte	greater than or equal
    gt	greater than
    lte	less than or equal
    lt	less than
    neq	not equal
    like	LIKE operator (use * in place of %)
    ilike	ILIKE operator (use * in place of %)
    in	one of a list of values e.g. ?a=in.1,2,3
    notin	not one of a list of values e.g. ?a=notin.1,2,3
    is	checking for exact equality (null,true,false)
    isnot	checking for exact inequality (null,true,false)
    @@	full-text search using to_tsquery
    @>	contains e.g. ?tags=@>.{example, new}
    <@	contained in e.g. values=<@{1,2,3}
    not	negates another operator, see below
    To negate any operator, prefix it with not like ?a=not.eq.2.



##Postgres conf files


## collapsible markdown?

<details><summary>CLICK ME</summary>
<p>

```python
# PostgreSQL Client Authentication Configuration File
# ===================================================
#
# Refer to the "Client Authentication" section in the PostgreSQL
# documentation for a complete description of this file.  A short
# synopsis follows.
#
# This file controls: which hosts are allowed to connect, how clients
# are authenticated, which PostgreSQL user names they can use, which
# databases they can access.  Records take one of these forms:
#
# local      DATABASE  USER  METHOD  [OPTIONS]
# host       DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostssl    DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostnossl  DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
#
# (The uppercase items must be replaced by actual values.)
#
# The first field is the connection type: "local" is a Unix-domain
# socket, "host" is either a plain or SSL-encrypted TCP/IP socket,
# "hostssl" is an SSL-encrypted TCP/IP socket, and "hostnossl" is a
# plain TCP/IP socket.
#
# DATABASE can be "all", "sameuser", "samerole", "replication", a
# database name, or a comma-separated list thereof. The "all"
# keyword does not match "replication". Access to replication
# must be enabled in a separate record (see example below).
#
# USER can be "all", a user name, a group name prefixed with "+", or a
# comma-separated list thereof.  In both the DATABASE and USER fields
# you can also write a file name prefixed with "@" to include names
# from a separate file.
#
# ADDRESS specifies the set of hosts the record matches.  It can be a
# host name, or it is made up of an IP address and a CIDR mask that is
# an integer (between 0 and 32 (IPv4) or 128 (IPv6) inclusive) that
# specifies the number of significant bits in the mask.  A host name
# that starts with a dot (.) matches a suffix of the actual host name.
# Alternatively, you can write an IP address and netmask in separate
# columns to specify the set of hosts.  Instead of a CIDR-address, you
# can write "samehost" to match any of the server's own IP addresses,
# or "samenet" to match any address in any subnet that the server is
# directly connected to.
#
# METHOD can be "trust", "reject", "md5", "password", "scram-sha-256",
# "gss", "sspi", "ident", "peer", "pam", "ldap", "radius" or "cert".
# Note that "password" sends passwords in clear text; "md5" or
# "scram-sha-256" are preferred since they send encrypted passwords.
#
# OPTIONS are a set of options for the authentication in the format
# NAME=VALUE.  The available options depend on the different
# authentication methods -- refer to the "Client Authentication"
# section in the documentation for a list of which options are
# available for which authentication methods.
#
# Database and user names containing spaces, commas, quotes and other
# special characters must be quoted.  Quoting one of the keywords
# "all", "sameuser", "samerole" or "replication" makes the name lose
# its special character, and just match a database or username with
# that name.
#
# This file is read on server startup and when the server receives a
# SIGHUP signal.  If you edit the file on a running system, you have to
# SIGHUP the server for the changes to take effect, run "pg_ctl reload",
# or execute "SELECT pg_reload_conf()".
#
# Put your actual configuration here
# ----------------------------------
#
# If you want to allow non-local connections, you need to add more
# "host" records.  In that case you will also need to make PostgreSQL
# listen on a non-local interface via the listen_addresses
# configuration parameter, or via the -i or -h command line switches.

# DO NOT DISABLE!
# If you change this first entry you will need to make sure that the
# database superuser can access the database using some other method.
# Noninteractive access to all databases is required during automatic
# maintenance (custom daily cronjobs, replication, and similar tasks).
#
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             0.0.0.0/0               md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```

</p>
</details>


<details><summary>Click to expand cat /etc/postgresql/11/main/pg_hba.conf</summary>
<p>

```js
# PostgreSQL Client Authentication Configuration File
# ===================================================
#
# Refer to the "Client Authentication" section in the PostgreSQL
# documentation for a complete description of this file.  A short
# synopsis follows.
#
# This file controls: which hosts are allowed to connect, how clients
# are authenticated, which PostgreSQL user names they can use, which
# databases they can access.  Records take one of these forms:
#
# local      DATABASE  USER  METHOD  [OPTIONS]
# host       DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostssl    DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostnossl  DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
#
# (The uppercase items must be replaced by actual values.)
#
# The first field is the connection type: "local" is a Unix-domain
# socket, "host" is either a plain or SSL-encrypted TCP/IP socket,
# "hostssl" is an SSL-encrypted TCP/IP socket, and "hostnossl" is a
# plain TCP/IP socket.
#
# DATABASE can be "all", "sameuser", "samerole", "replication", a
# database name, or a comma-separated list thereof. The "all"
# keyword does not match "replication". Access to replication
# must be enabled in a separate record (see example below).
#
# USER can be "all", a user name, a group name prefixed with "+", or a
# comma-separated list thereof.  In both the DATABASE and USER fields
# you can also write a file name prefixed with "@" to include names
# from a separate file.
#
# ADDRESS specifies the set of hosts the record matches.  It can be a
# host name, or it is made up of an IP address and a CIDR mask that is
# an integer (between 0 and 32 (IPv4) or 128 (IPv6) inclusive) that
# specifies the number of significant bits in the mask.  A host name
# that starts with a dot (.) matches a suffix of the actual host name.
# Alternatively, you can write an IP address and netmask in separate
# columns to specify the set of hosts.  Instead of a CIDR-address, you
# can write "samehost" to match any of the server's own IP addresses,
# or "samenet" to match any address in any subnet that the server is
# directly connected to.
#
# METHOD can be "trust", "reject", "md5", "password", "scram-sha-256",
# "gss", "sspi", "ident", "peer", "pam", "ldap", "radius" or "cert".
# Note that "password" sends passwords in clear text; "md5" or
# "scram-sha-256" are preferred since they send encrypted passwords.
#
# OPTIONS are a set of options for the authentication in the format
# NAME=VALUE.  The available options depend on the different
# authentication methods -- refer to the "Client Authentication"
# section in the documentation for a list of which options are
# available for which authentication methods.
#
# Database and user names containing spaces, commas, quotes and other
# special characters must be quoted.  Quoting one of the keywords
# "all", "sameuser", "samerole" or "replication" makes the name lose
# its special character, and just match a database or username with
# that name.
#
# This file is read on server startup and when the server receives a
# SIGHUP signal.  If you edit the file on a running system, you have to
# SIGHUP the server for the changes to take effect, run "pg_ctl reload",
# or execute "SELECT pg_reload_conf()".
#
# Put your actual configuration here
# ----------------------------------
#
# If you want to allow non-local connections, you need to add more
# "host" records.  In that case you will also need to make PostgreSQL
# listen on a non-local interface via the listen_addresses
# configuration parameter, or via the -i or -h command line switches.

# DO NOT DISABLE!
# If you change this first entry you will need to make sure that the
# database superuser can access the database using some other method.
# Noninteractive access to all databases is required during automatic
# maintenance (custom daily cronjobs, replication, and similar tasks).
#
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             0.0.0.0/0               md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```        
</p>
</details>

<details><summary>Click to expand cat /etc/postgresql/11/main/pg_ctl.conf</summary>
<p>

```js
    # Automatic pg_ctl configuration
    # This configuration file contains cluster specific options to be passed to
    # pg_ctl(1).
    
    pg_ctl_options = ''
    
    root@ip-172-31-47-137:/home/ubuntu# cat /etc/postgresql/11/main/pg_ident.conf
    
    # PostgreSQL User Name Maps
    # =========================
    #
    # Refer to the PostgreSQL documentation, chapter "Client
    # Authentication" for a complete description.  A short synopsis
    # follows.
    #
    # This file controls PostgreSQL user name mapping.  It maps external
    # user names to their corresponding PostgreSQL user names.  Records
    # are of the form:
    #
    # MAPNAME  SYSTEM-USERNAME  PG-USERNAME
    #
    # (The uppercase quantities must be replaced by actual values.)
    #
    # MAPNAME is the (otherwise freely chosen) map name that was used in
    # pg_hba.conf.  SYSTEM-USERNAME is the detected user name of the
    # client.  PG-USERNAME is the requested PostgreSQL user name.  The
    # existence of a record specifies that SYSTEM-USERNAME may connect as
    # PG-USERNAME.
    #
    # If SYSTEM-USERNAME starts with a slash (/), it will be treated as a
    # regular expression.  Optionally this can contain a capture (a
    # parenthesized subexpression).  The substring matching the capture
    # will be substituted for \1 (backslash-one) if present in
    # PG-USERNAME.
    #
    # Multiple maps may be specified in this file and used by pg_hba.conf.
    #
    # No map names are defined in the default configuration.  If all
    # system user names and PostgreSQL user names are the same, you don't
    # need anything in this file.
    #
    # This file is read on server startup and when the postmaster receives
    # a SIGHUP signal.  If you edit the file on a running system, you have
    # to SIGHUP the postmaster for the changes to take effect.  You can
    # use "pg_ctl reload" to do that.
    
    # Put your actual configuration here
    # ----------------------------------
    
    # MAPNAME       SYSTEM-USERNAME         PG-USERNAME
```
    
</p>
</details>    


<details><summary>Click to expand cat /etc/postgresql/11/main/environment</summary>
<p>

    # environment variables for postgres processes
    # This file has the same syntax as postgresql.conf:
    #  VARIABLE = simple_value
    #  VARIABLE2 = 'any value!'
    # I. e. you need to enclose any value which does not only consist of letters,
    # numbers, and '-', '_', '.' in single quotes. Shell commands are not
    # evaluated.
    
</p>
</details>    

<details><summary>Click to expand cat /etc/postgresql/11/main/postgresql.conf</summary>
<p>

    # -----------------------------
    # PostgreSQL configuration file
    # -----------------------------
    #
    # This file consists of lines of the form:
    #
    #   name = value
    #
    # (The "=" is optional.)  Whitespace may be used.  Comments are introduced with
    # "#" anywhere on a line.  The complete list of parameter names and allowed
    # values can be found in the PostgreSQL documentation.
    #
    # The commented-out settings shown in this file represent the default values.
    # Re-commenting a setting is NOT sufficient to revert it to the default value;
    # you need to reload the server.
    #
    # This file is read on server startup and when the server receives a SIGHUP
    # signal.  If you edit the file on a running system, you have to SIGHUP the
    # server for the changes to take effect, run "pg_ctl reload", or execute
    # "SELECT pg_reload_conf()".  Some parameters, which are marked below,
    # require a server shutdown and restart to take effect.
    #
    # Any parameter can also be given as a command-line option to the server, e.g.,
    # "postgres -c log_connections=on".  Some parameters can be changed at run time
    # with the "SET" SQL command.
    #
    # Memory units:  kB = kilobytes        Time units:  ms  = milliseconds
    #                MB = megabytes                     s   = seconds
    #                GB = gigabytes                     min = minutes
    #                TB = terabytes                     h   = hours
    #                                                   d   = days
    
    
    #------------------------------------------------------------------------------
    # FILE LOCATIONS
    #------------------------------------------------------------------------------
    
    # The default values of these variables are driven from the -D command-line
    # option or PGDATA environment variable, represented here as ConfigDir.
    
    data_directory = '/var/lib/postgresql/11/main'          # use data in another directory
                                            # (change requires restart)
    hba_file = '/etc/postgresql/11/main/pg_hba.conf'        # host-based authentication file
                                            # (change requires restart)
    ident_file = '/etc/postgresql/11/main/pg_ident.conf'    # ident configuration file
                                            # (change requires restart)
    
    # If external_pid_file is not explicitly set, no extra PID file is written.
    external_pid_file = '/var/run/postgresql/11-main.pid'                   # write an extra PID file
                                            # (change requires restart)
    
    
    #------------------------------------------------------------------------------
    # CONNECTIONS AND AUTHENTICATION
    #------------------------------------------------------------------------------
    
    # - Connection Settings -
    
    listen_addresses = '*'                  # what IP address(es) to listen on;
                                            # comma-separated list of addresses;
                                            # defaults to 'localhost'; use '*' for all
                                            # (change requires restart)
    port = 5444                             # (change requires restart) SMH
    max_connections = 100                   # (change requires restart)
    #superuser_reserved_connections = 3     # (change requires restart)
    unix_socket_directories = '/var/run/postgresql' # comma-separated list of directories
                                            # (change requires restart)
    #unix_socket_group = ''                 # (change requires restart)
    #unix_socket_permissions = 0777         # begin with 0 to use octal notation
                                            # (change requires restart)
    #bonjour = off                          # advertise server via Bonjour
                                            # (change requires restart)
    #bonjour_name = ''                      # defaults to the computer name
                                            # (change requires restart)
    
    # - TCP Keepalives -
    # see "man 7 tcp" for details
    
    #tcp_keepalives_idle = 0                # TCP_KEEPIDLE, in seconds;
                                            # 0 selects the system default
    #tcp_keepalives_interval = 0            # TCP_KEEPINTVL, in seconds;
                                            # 0 selects the system default
    #tcp_keepalives_count = 0               # TCP_KEEPCNT;
                                            # 0 selects the system default
    
    # - Authentication -
    
    #authentication_timeout = 1min          # 1s-600s
    #password_encryption = md5              # md5 or scram-sha-256
    #db_user_namespace = off
    
    # GSSAPI using Kerberos
    #krb_server_keyfile = ''
    #krb_caseins_users = off
    
    # - SSL -
    
    ssl = on
    #ssl_ca_file = ''
    ssl_cert_file = '/etc/ssl/certs/ssl-cert-snakeoil.pem'
    #ssl_crl_file = ''
    ssl_key_file = '/etc/ssl/private/ssl-cert-snakeoil.key'
    #ssl_ciphers = 'HIGH:MEDIUM:+3DES:!aNULL' # allowed SSL ciphers
    #ssl_prefer_server_ciphers = on
    #ssl_ecdh_curve = 'prime256v1'
    #ssl_dh_params_file = ''
    #ssl_passphrase_command = ''
    #ssl_passphrase_command_supports_reload = off
    
    
    #------------------------------------------------------------------------------
    # RESOURCE USAGE (except WAL)
    #------------------------------------------------------------------------------
    
    # - Memory -
    
    shared_buffers = 2GB                    # min 128kB
                                            # (change requires restart)
    #huge_pages = try                       # on, off, or try
                                            # (change requires restart)
    #temp_buffers = 8MB                     # min 800kB
    #max_prepared_transactions = 0          # zero disables the feature
                                            # (change requires restart)
    # Caution: it is not advisable to set max_prepared_transactions nonzero unless
    # you actively intend to use prepared transactions.
    work_mem = 10208kB                      # min 64kB SMH
    maintenance_work_mem = 1GB              # min 1MB SMH
    #autovacuum_work_mem = -1               # min 1MB, or -1 to use maintenance_work_mem
    #max_stack_depth = 2MB                  # min 100kB
    dynamic_shared_memory_type = posix      # the default is the first option
                                            # supported by the operating system:
                                            #   posix
                                            #   sysv
                                            #   windows
                                            #   mmap
                                            # use none to disable dynamic shared memory
                                            # (change requires restart)
    
    # - Disk -
    
    #temp_file_limit = -1                   # limits per-process temp file space
                                            # in kB, or -1 for no limit
    
    # - Kernel Resources -
    
    #max_files_per_process = 1000           # min 25
                                            # (change requires restart)
    
    # - Cost-Based Vacuum Delay -
    
    #vacuum_cost_delay = 0                  # 0-100 milliseconds
    #vacuum_cost_page_hit = 1               # 0-10000 credits
    #vacuum_cost_page_miss = 10             # 0-10000 credits
    #vacuum_cost_page_dirty = 20            # 0-10000 credits
    #vacuum_cost_limit = 200                # 1-10000 credits
    
    # - Background Writer -
    
    #bgwriter_delay = 200ms                 # 10-10000ms between rounds
    #bgwriter_lru_maxpages = 100            # max buffers written/round, 0 disables
    #bgwriter_lru_multiplier = 2.0          # 0-10.0 multiplier on buffers scanned/round
    #bgwriter_flush_after = 512kB           # measured in pages, 0 disables
    
    # - Asynchronous Behavior -
    
    effective_io_concurrency = 200          # 1-1000; 0 disables prefetching SMH
    max_worker_processes = 13               # SMH (change requires restart)
    #max_parallel_maintenance_workers = 2   # taken from max_parallel_workers
    max_parallel_workers_per_gather = 1     # taken from max_parallel_workers
    #parallel_leader_participation = on
    max_parallel_workers = 2                # SMH maximum number of max_worker_processes that
                                            # can be used in parallel operations
    #old_snapshot_threshold = -1            # 1min-60d; -1 disables; 0 is immediate
                                            # (change requires restart)
    #backend_flush_after = 0                # measured in pages, 0 disables
    
    
    #------------------------------------------------------------------------------
    # WRITE-AHEAD LOG
    #------------------------------------------------------------------------------
    
    # - Settings -
    
    #wal_level = replica                    # minimal, replica, or logical
                                            # (change requires restart)
    #fsync = on                             # flush data to disk for crash safety
                                            # (turning this off can cause
                                            # unrecoverable data corruption)
    #synchronous_commit = on                # synchronization level;
                                            # off, local, remote_write, remote_apply, or on
    #wal_sync_method = fsync                # the default is the first option
                                            # supported by the operating system:
                                            #   open_datasync
                                            #   fdatasync (default on Linux)
                                            #   fsync
                                            #   fsync_writethrough
                                            #   open_sync
    #full_page_writes = on                  # recover from partial page writes
    #wal_compression = off                  # enable compression of full-page writes
    #wal_log_hints = off                    # also do full page writes of non-critical updates
                                            # (change requires restart)
    wal_buffers = 16MB                      # min 32kB, -1 sets based on shared_buffers SMH
                                            # (change requires restart)
    #wal_writer_delay = 200ms               # 1-10000 milliseconds
    #wal_writer_flush_after = 1MB           # measured in pages, 0 disables
    
    #commit_delay = 0                       # range 0-100000, in microseconds
    #commit_siblings = 5                    # range 1-1000
    
    # - Checkpoints -
    
    #checkpoint_timeout = 5min              # range 30s-1d
    max_wal_size = 8GB                      # SMH
    min_wal_size = 4GB                      # SMH
    checkpoint_completion_target = 0.9      # checkpoint target duration, 0.0 - 1.0 SMH
    #checkpoint_flush_after = 256kB         # measured in pages, 0 disables
    #checkpoint_warning = 30s               # 0 disables
    
    # - Archiving -
    
    #archive_mode = off             # enables archiving; off, on, or always
                                    # (change requires restart)
    #archive_command = ''           # command to use to archive a logfile segment
                                    # placeholders: %p = path of file to archive
                                    #               %f = file name only
                                    # e.g. 'test ! -f /mnt/server/archivedir/%f && cp %p /mnt/server/archivedir/%f'
    #archive_timeout = 0            # force a logfile segment switch after this
                                    # number of seconds; 0 disables
    
    
    #------------------------------------------------------------------------------
    # REPLICATION
    #------------------------------------------------------------------------------
    
    # - Sending Servers -
    
    # Set these on the master and on any standby that will send replication data.
    
    #max_wal_senders = 10           # max number of walsender processes
                                    # (change requires restart)
    #wal_keep_segments = 0          # in logfile segments; 0 disables
    #wal_sender_timeout = 60s       # in milliseconds; 0 disables
    
    #max_replication_slots = 10     # max number of replication slots
                                    # (change requires restart)
    #track_commit_timestamp = off   # collect timestamp of transaction commit
                                    # (change requires restart)
    
    # - Master Server -
    
    # These settings are ignored on a standby server.
    
    #synchronous_standby_names = '' # standby servers that provide sync rep
                                    # method to choose sync standbys, number of sync standbys,
                                    # and comma-separated list of application_name
                                    # from standby(s); '*' = all
    #vacuum_defer_cleanup_age = 0   # number of xacts by which cleanup is delayed
    
    # - Standby Servers -
    
    # These settings are ignored on a master server.
    
    #hot_standby = on                       # "off" disallows queries during recovery
                                            # (change requires restart)
    #max_standby_archive_delay = 30s        # max delay before canceling queries
                                            # when reading WAL from archive;
                                            # -1 allows indefinite delay
    #max_standby_streaming_delay = 30s      # max delay before canceling queries
                                            # when reading streaming WAL;
                                            # -1 allows indefinite delay
    #wal_receiver_status_interval = 10s     # send replies at least this often
                                            # 0 disables
    #hot_standby_feedback = off             # send info from standby to prevent
                                            # query conflicts
    #wal_receiver_timeout = 60s             # time that receiver waits for
                                            # communication from master
                                            # in milliseconds; 0 disables
    #wal_retrieve_retry_interval = 5s       # time to wait before retrying to
                                            # retrieve WAL after a failed attempt
    
    # - Subscribers -
    
    # These settings are ignored on a publisher.
    
    #max_logical_replication_workers = 4    # taken from max_worker_processes
                                            # (change requires restart)
    #max_sync_workers_per_subscription = 2  # taken from max_logical_replication_workers
    
    
    #------------------------------------------------------------------------------
    # QUERY TUNING
    #------------------------------------------------------------------------------
    
    # - Planner Method Configuration -
    
    #enable_bitmapscan = on
    #enable_hashagg = on
    #enable_hashjoin = on
    #enable_indexscan = on
    #enable_indexonlyscan = on
    #enable_material = on
    #enable_mergejoin = on
    #enable_nestloop = on
    #enable_parallel_append = on
    #enable_seqscan = on
    #enable_sort = on
    #enable_tidscan = on
    #enable_partitionwise_join = off
    #enable_partitionwise_aggregate = off
    #enable_parallel_hash = on
    #enable_partition_pruning = on
    
    # - Planner Cost Constants -
    
    #seq_page_cost = 1.0                    # measured on an arbitrary scale
    random_page_cost = 1.1                  # same scale as above SMH
    #cpu_tuple_cost = 0.01                  # same scale as above
    #cpu_index_tuple_cost = 0.005           # same scale as above
    #cpu_operator_cost = 0.0025             # same scale as above
    #parallel_tuple_cost = 0.1              # same scale as above
    #parallel_setup_cost = 1000.0   # same scale as above
    
    #jit_above_cost = 100000                # perform JIT compilation if available
                                            # and query more expensive than this;
                                            # -1 disables
    #jit_inline_above_cost = 500000         # inline small functions if query is
                                            # more expensive than this; -1 disables
    #jit_optimize_above_cost = 500000       # use expensive JIT optimizations if
                                            # query is more expensive than this;
                                            # -1 disables
    
    #min_parallel_table_scan_size = 8MB
    #min_parallel_index_scan_size = 512kB
    effective_cache_size = 6GB              # SMH
    
    # - Genetic Query Optimizer -
    
    #geqo = on
    #geqo_threshold = 12
    #geqo_effort = 5                        # range 1-10
    #geqo_pool_size = 0                     # selects default based on effort
    #geqo_generations = 0                   # selects default based on effort
    #geqo_selection_bias = 2.0              # range 1.5-2.0
    #geqo_seed = 0.0                        # range 0.0-1.0
    
    # - Other Planner Options -
    
    default_statistics_target = 500         # range 1-10000 SMH
    #constraint_exclusion = partition       # on, off, or partition
    #cursor_tuple_fraction = 0.1            # range 0.0-1.0
    #from_collapse_limit = 8
    #join_collapse_limit = 8                # 1 disables collapsing of explicit
                                            # JOIN clauses
    #force_parallel_mode = off
    #jit = off                              # allow JIT compilation
    
    
    #------------------------------------------------------------------------------
    # REPORTING AND LOGGING
    #------------------------------------------------------------------------------
    
    # - Where to Log -
    
    #log_destination = 'stderr'             # Valid values are combinations of
                                            # stderr, csvlog, syslog, and eventlog,
                                            # depending on platform.  csvlog
                                            # requires logging_collector to be on.
    
    # This is used when logging to stderr:
    #logging_collector = off                # Enable capturing of stderr and csvlog
                                            # into log files. Required to be on for
                                            # csvlogs.
                                            # (change requires restart)
    
    # These are only used if logging_collector is on:
    #log_directory = 'log'                  # directory where log files are written,
                                            # can be absolute or relative to PGDATA
    #log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'        # log file name pattern,
                                            # can include strftime() escapes
    #log_file_mode = 0600                   # creation mode for log files,
                                            # begin with 0 to use octal notation
    #log_truncate_on_rotation = off         # If on, an existing log file with the
                                            # same name as the new log file will be
                                            # truncated rather than appended to.
                                            # But such truncation only occurs on
                                            # time-driven rotation, not on restarts
                                            # or size-driven rotation.  Default is
                                            # off, meaning append to existing files
                                            # in all cases.
    #log_rotation_age = 1d                  # Automatic rotation of logfiles will
                                            # happen after that time.  0 disables.
    #log_rotation_size = 10MB               # Automatic rotation of logfiles will
                                            # happen after that much log output.
                                            # 0 disables.
    
    # These are relevant when logging to syslog:
    #syslog_facility = 'LOCAL0'
    #syslog_ident = 'postgres'
    #syslog_sequence_numbers = on
    #syslog_split_messages = on
    
    # This is only relevant when logging to eventlog (win32):
    # (change requires restart)
    #event_source = 'PostgreSQL'
    
    # - When to Log -
    
    #log_min_messages = warning             # values in order of decreasing detail:
                                            #   debug5
                                            #   debug4
                                            #   debug3
                                            #   debug2
                                            #   debug1
                                            #   info
                                            #   notice
                                            #   warning
                                            #   error
                                            #   log
                                            #   fatal
                                            #   panic
    
    #log_min_error_statement = error        # values in order of decreasing detail:
                                            #   debug5
                                            #   debug4
                                            #   debug3
                                            #   debug2
                                            #   debug1
                                            #   info
                                            #   notice
                                            #   warning
                                            #   error
                                            #   log
                                            #   fatal
                                            #   panic (effectively off)
    
    #log_min_duration_statement = -1        # -1 is disabled, 0 logs all statements
                                            # and their durations, > 0 logs only
                                            # statements running at least this number
                                            # of milliseconds
    
    
    # - What to Log -
    
    #debug_print_parse = off
    #debug_print_rewritten = off
    #debug_print_plan = off
    #debug_pretty_print = on
    #log_checkpoints = off
    #log_connections = off
    #log_disconnections = off
    #log_duration = off
    #log_error_verbosity = default          # terse, default, or verbose messages
    #log_hostname = off
    log_line_prefix = '%m [%p] %q%u@%d '            # special values:
                                            #   %a = application name
                                            #   %u = user name
                                            #   %d = database name
                                            #   %r = remote host and port
                                            #   %h = remote host
                                            #   %p = process ID
                                            #   %t = timestamp without milliseconds
                                            #   %m = timestamp with milliseconds
                                            #   %n = timestamp with milliseconds (as a Unix epoch)
                                            #   %i = command tag
                                            #   %e = SQL state
                                            #   %c = session ID
                                            #   %l = session line number
                                            #   %s = session start timestamp
                                            #   %v = virtual transaction ID
                                            #   %x = transaction ID (0 if none)
                                            #   %q = stop here in non-session
                                            #        processes
                                            #   %% = '%'
                                            # e.g. '<%u%%%d> '
    #log_lock_waits = off                   # log lock waits >= deadlock_timeout
    #log_statement = 'none'                 # none, ddl, mod, all
    #log_replication_commands = off
    #log_temp_files = -1                    # log temporary files equal or larger
                                            # than the specified size in kilobytes;
                                            # -1 disables, 0 logs all temp files
    log_timezone = 'Etc/UTC'
    
    #------------------------------------------------------------------------------
    # PROCESS TITLE
    #------------------------------------------------------------------------------
    
    cluster_name = '11/main'                        # added to process titles if nonempty
                                            # (change requires restart)
    #update_process_title = on
    
    
    #------------------------------------------------------------------------------
    # STATISTICS
    #------------------------------------------------------------------------------
    
    # - Query and Index Statistics Collector -
    
    #track_activities = on
    #track_counts = on
    #track_io_timing = off
    #track_functions = none                 # none, pl, all
    #track_activity_query_size = 1024       # (change requires restart)
    stats_temp_directory = '/var/run/postgresql/11-main.pg_stat_tmp'
    
    
    # - Monitoring -
    
    #log_parser_stats = off
    #log_planner_stats = off
    #log_executor_stats = off
    #log_statement_stats = off
    
    
    #------------------------------------------------------------------------------
    # AUTOVACUUM
    #------------------------------------------------------------------------------
    
    #autovacuum = on                        # Enable autovacuum subprocess?  'on'
                                            # requires track_counts to also be on.
    #log_autovacuum_min_duration = -1       # -1 disables, 0 logs all actions and
                                            # their durations, > 0 logs only
                                            # actions running at least this number
                                            # of milliseconds.
    autovacuum_max_workers = 10             # max number of autovacuum subprocesses
                                            # (change requires restart)
    autovacuum_naptime = 10         # time between autovacuum runs
    #autovacuum_vacuum_threshold = 50       # min number of row updates before
                                            # vacuum
    #autovacuum_analyze_threshold = 50      # min number of row updates before
                                            # analyze
    #autovacuum_vacuum_scale_factor = 0.2   # fraction of table size before vacuum
    #autovacuum_analyze_scale_factor = 0.1  # fraction of table size before analyze
    #autovacuum_freeze_max_age = 200000000  # maximum XID age before forced vacuum
                                            # (change requires restart)
    #autovacuum_multixact_freeze_max_age = 400000000        # maximum multixact age
                                            # before forced vacuum
                                            # (change requires restart)
    #autovacuum_vacuum_cost_delay = 20ms    # default vacuum cost delay for
                                            # autovacuum, in milliseconds;
                                            # -1 means use vacuum_cost_delay
    #autovacuum_vacuum_cost_limit = -1      # default vacuum cost limit for
                                            # autovacuum, -1 means use
                                            # vacuum_cost_limit
    
    
    #------------------------------------------------------------------------------
    # CLIENT CONNECTION DEFAULTS
    #------------------------------------------------------------------------------
    
    # - Statement Behavior -
    
    #client_min_messages = notice           # values in order of decreasing detail:
                                            #   debug5
                                            #   debug4
                                            #   debug3
                                            #   debug2
                                            #   debug1
                                            #   log
                                            #   notice
                                            #   warning
                                            #   error
    #search_path = '"$user", public'        # schema names
    #row_security = on
    #default_tablespace = ''                # a tablespace name, '' uses the default
    #temp_tablespaces = ''                  # a list of tablespace names, '' uses
                                            # only default tablespace
    #check_function_bodies = on
    #default_transaction_isolation = 'read committed'
    #default_transaction_read_only = off
    #default_transaction_deferrable = off
    #session_replication_role = 'origin'
    #statement_timeout = 0                  # in milliseconds, 0 is disabled
    #lock_timeout = 0                       # in milliseconds, 0 is disabled
    #idle_in_transaction_session_timeout = 0        # in milliseconds, 0 is disabled
    #vacuum_freeze_min_age = 50000000
    #vacuum_freeze_table_age = 150000000
    #vacuum_multixact_freeze_min_age = 5000000
    #vacuum_multixact_freeze_table_age = 150000000
    #vacuum_cleanup_index_scale_factor = 0.1        # fraction of total number of tuples
                                                    # before index cleanup, 0 always performs
                                                    # index cleanup
    #bytea_output = 'hex'                   # hex, escape
    #xmlbinary = 'base64'
    #xmloption = 'content'
    #gin_fuzzy_search_limit = 0
    #gin_pending_list_limit = 4MB
    
    # - Locale and Formatting -
    
    datestyle = 'iso, mdy'
    #intervalstyle = 'postgres'
    timezone = 'Etc/UTC'
    #timezone_abbreviations = 'Default'     # Select the set of available time zone
                                            # abbreviations.  Currently, there are
                                            #   Default
                                            #   Australia (historical usage)
                                            #   India
                                            # You can create your own file in
                                            # share/timezonesets/.
    #extra_float_digits = 0                 # min -15, max 3
    #client_encoding = sql_ascii            # actually, defaults to database
                                            # encoding
    
    # These settings are initialized by initdb, but they can be changed.
    lc_messages = 'C.UTF-8'                 # locale for system error message
                                            # strings
    lc_monetary = 'C.UTF-8'                 # locale for monetary formatting
    lc_numeric = 'C.UTF-8'                  # locale for number formatting
    lc_time = 'C.UTF-8'                             # locale for time formatting
    
    # default configuration for text search
    default_text_search_config = 'pg_catalog.english'
    
    # - Shared Library Preloading -
    
    #shared_preload_libraries = ''  # (change requires restart)
    #local_preload_libraries = ''
    #session_preload_libraries = ''
    #jit_provider = 'llvmjit'               # JIT library to use
    
    # - Other Defaults -
    
    #dynamic_library_path = '$libdir'
    
    
    #------------------------------------------------------------------------------
    # LOCK MANAGEMENT
    #------------------------------------------------------------------------------
    
    #deadlock_timeout = 1s
    max_locks_per_transaction = 64          # min 10
                                            # (change requires restart)
    #max_pred_locks_per_transaction = 64    # min 10
                                            # (change requires restart)
    #max_pred_locks_per_relation = -2       # negative values mean
                                            # (max_pred_locks_per_transaction
                                            #  / -max_pred_locks_per_relation) - 1
    #max_pred_locks_per_page = 2            # min 0
    
    
    #------------------------------------------------------------------------------
    # VERSION AND PLATFORM COMPATIBILITY
    #------------------------------------------------------------------------------
    
    # - Previous PostgreSQL Versions -
    
    #array_nulls = on
    #backslash_quote = safe_encoding        # on, off, or safe_encoding
    #default_with_oids = off
    #escape_string_warning = on
    #lo_compat_privileges = off
    #operator_precedence_warning = off
    #quote_all_identifiers = off
    #standard_conforming_strings = on
    #synchronize_seqscans = on
    
    # - Other Platforms and Clients -
    
    #transform_null_equals = off
    
    
    #------------------------------------------------------------------------------
    # ERROR HANDLING
    #------------------------------------------------------------------------------
    
    #exit_on_error = off                    # terminate session on any error?
    #restart_after_crash = on               # reinitialize after backend crash?
    #data_sync_retry = off                  # retry or panic on failure to fsync
                                            # data?
                                            # (change requires restart)
    
    
    #------------------------------------------------------------------------------
    # CONFIG FILE INCLUDES
    #------------------------------------------------------------------------------
    
    # These options allow settings to be loaded from files other than the
    # default postgresql.conf.
    
    include_dir = 'conf.d'                  # include files ending in '.conf' from
                                            # a directory, e.g., 'conf.d'
    #include_if_exists = ''                 # include file only if it exists
    #include = ''                           # include file
    
    
    #------------------------------------------------------------------------------
    # CUSTOMIZED OPTIONS
    #------------------------------------------------------------------------------
    
    # Add settings for extensions here
    shared_preload_libraries = 'timescaledb'
    timescaledb.max_background_workers = 8
    timescaledb.last_tuned = '2019-07-29T19:10:58Z'
    timescaledb.last_tuned_version = '0.7.0'

</p>
</details>
