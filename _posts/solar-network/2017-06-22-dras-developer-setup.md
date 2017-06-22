---
layout: post
category : solarnetwork
tagline: "SolarDRAS Development Setup on OSX"
tags : [solarnetwork, dras]
---
{% include JB/setup %}

# SolarDRAS Development Setup on OSX
I recently did a small piece of work on [SolarNetwork](https://github.com/SolarNetwork/solarnetwork/wiki) and had to set up my environment again. This is very well documented on the wiki: [Developer Guide](https://github.com/SolarNetwork/solarnetwork/wiki/Developer-Guide) but I thought I'd put together a quick guide on how to get up and running quickly on Mac OSX.

Prerequisites:
* Install [Eclipse](http://www.eclipse.org/downloads/), Neon 3 is the supported version at the time of writing.
* Install [PostgreSQL](https://www.postgresql.org/download/macosx/), 9.6 at the time of writing.
I chose the [Postgres.app](http://postgresapp.com/) as it's nice and self contained (make sure you follow the installation steps so that the command line commands are available).

### Git

Check out the following git repositories into your workspace:
* git@github.com:SolarNetwork/solarnetwork-build.git
* git@github.com:SolarNetwork/solarnetwork-common.git
* git@github.com:SolarNetwork/solarnetwork-external.git
* git@github.com:SolarNetwork/solarnetwork-central.git
* git@github.com:SolarNetwork/solarnetwork-dras.git

Depending on you're development requirements you could also check out node:
* git@github.com:SolarNetwork/solarnetwork-node.git

### Eclipse

You'll need to follow the instructions at: [Eclipse Setup Guide](https://github.com/SolarNetwork/solarnetwork/wiki/Eclipse-Setup-Guide) for setting up eclipse.

Personally I skipped the git configuration here as I was more comfortable doing this outside Eclipse.

I found at this point that I had to disable a number of OSGI bundles in eclipse:
* archiva-obr-plugin
* net.solarnetwork.central.common.mail.javamail
* net.solarnetwork.central.dras.biz.dao.ibatis.test
* net.solarnetwork.central.dras.dao.ibatis
* net.solarnetwork.central.dras.dao.ibatis.test
* net.solarnetwork.central.user.pki.dogtag
* net.solarnetwork.central.user.pki.dogtag.test
* net.solarnetwork.common.pidfile

I also had to make sure that the `net.solarnetwork.external.org.apache.log4j.config` bundle was the one provided by `solarnetwork-central`.

### Database

You can either follow the wiki instructions on the database setup or run the script below if you've checked the git repositories out into your workspace (you may need to alter ~/workspace to reflect the location that you've checked out the code).

```
#!/bin/bash
dropdb solarnetwork
dropuser solarnet

dropdb solarnet_unittest
dropuser solarnet_test

createuser -AD solarnet
psql -U postgres -d postgres -c "alter user solarnet with password 'solarnet';"
createdb -E UNICODE -l C -T template0 -O solarnet solarnetwork
createlang plv8 solarnetwork
psql -U postgres -d solarnetwork -c "CREATE EXTENSION IF NOT EXISTS citext WITH SCHEMA public;"

createuser -AD solarnet_test
psql -U postgres -d postgres -c "alter user solarnet_test with password 'solarnet_test';"
createdb -E UNICODE -l C -T template0 -O solarnet_test solarnet_unittest
createlang plv8 solarnet_unittest
psql -U postgres -d solarnet_unittest -c "CREATE EXTENSION IF NOT EXISTS citext WITH SCHEMA public;"

# Setup base database
cd ~/workspace/solarnetwork-central/net.solarnetwork.central.datum/defs/sql/postgres

# for some reason, plv8 often chokes on the inline comments, so strip them out
sed -e '/^\/\*/d' -e '/^ \*/d' postgres-init-plv8.sql | psql -d solarnetwork -U postgres
psql -d solarnetwork -U solarnet -f postgres-init.sql
psql -d solarnetwork -U solarnet -f postgres-init-data.sql

# for some reason, plv8 often chokes on the inline comments, so strip them out
sed -e '/^\/\*/d' -e '/^ \*/d' postgres-init-plv8.sql | psql -d solarnet_unittest -U postgres
psql -d solarnet_unittest -U solarnet_test -f postgres-init.sql

# DRAS extensions
cd ~/workspace/solarnetwork-dras/net.solarnetwork.central.dras/defs/sql/postgres

psql -d solarnetwork -U solarnet -f dras-reboot.sql
psql -U psql -d solarnetwork -c "ALTER ROLE solarnet SET intervalstyle = 'iso_8601'"

```

### DRAS Configuration

In order for the Google maps integration to work make sure to configure the Google Maps API key in the `net.solarnetwork.central.dras.web` bundle by copying and configuring the example configuration to `solarnetwork-osgi-target`. This will require copying the `net.solarnetwork.central.dras.web.properties` file into `configuration/services` and changing the suffix from `properties` to `cfg`.

You'll need to insert a user directly into the database to be able to access the site as no example data is currently provided. Details on the password hashing, as well as other required DRAS configuration can be found at [SolarDRAS Development Guide](https://github.com/SolarNetwork/solarnetwork/wiki/SolarDRAS-Development-Guide).

With all this done, you should be able to start the target in eclipse and access the website.
