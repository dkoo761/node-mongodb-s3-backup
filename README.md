# Node MongoDB / S3 Backup

### This is a fork to fix a breaking bug in the original repo

The original repo (https://github.com/theycallmeswift/node-mongodb-s3-backup) is no longer working due to a breaking 
change in the knox S3 library.

#### CHANGELOG

1. Replaced `knox` with `knox-s3` which is a fork of `knox` that fixes the issue 
since `knox` itself is no longer maintained.

2. Added some new options to the `mongodb` portion of the JSON config file (see details below).

3. Replaced `tmpDir()` with `tmpdir()` to get rid of a deprecation warning.

4. Changed `stderr` logging to use `[info]` instead of `[error]` since it was marking normal results as errors.

### Original Intro...

This is a package that makes backing up your mongo databases to S3 simple.
The binary file is a node cronjob that runs at midnight every day and backs up
the database specified in the config file.

## Installation

Since this repo is NOT PUBLISHED IN NPM as it is a fork of the original, you can install it by:

1) Downloading this repo

2) `cd /path/to/downloaded/repo`

3) `npm install -g`

## Configuration

To configure the backup, you need to pass the binary a JSON configuration file.

Note the following new options available under the `mongodb` portion of the JSON config file:

`type` - MUST be either `standalone` or `replicaSet`

`host` - if using type `replicaSet`, include the usual comma-separated list of replica set members with both host and port numbers for each member

`port` - optional. If using type `replicaSet`, you must OMIT this field completely!

`isSSL` - optional. Set to `true` if connecting via SSL (this simply sets the `--ssl` option)

`authenticationDatabase` - optional. Set to the name of your authenticationDatabase if using one.

There is a sample configuration file supplied in the package (`config.sample.json`).

The file should have the following format:

    {
      "mongodb": {
        "type": "standalone",
        "host": "localhost",
        "port": 27017,
        "username": false,
        "password": false,
        "db": "database_to_backup"
      },
      "s3": {
        "key": "your_s3_key",
        "secret": "your_s3_secret",
        "bucket": "s3_bucket_to_upload_to",
        "destination": "/",
        "encrypt": true,
        "region": "s3_region_to_use"
      },
      "cron": {
        "time": "11:59",
      }
    }

All options in the "s3" object, except for desination, will be directly passed to knox, therefore, you can include any of the options listed [in the knox documentation](https://github.com/LearnBoost/knox#client-creation-options "Knox README").

### Crontabs

You may optionally substitute the cron "time" field with an explicit "crontab"
of the standard format `0 0 * * *`.

      "cron": {
        "crontab": "0 0 * * *"
      }

*Note*: The version of cron that we run supports a sixth digit (which is in seconds) if
you need it.

### Timezones

The optional "timezone" allows you to specify timezone-relative time regardless
of local timezone on the host machine.

      "cron": {
        "time": "00:00",
        "timezone": "America/New_York"
      }

You must first `npm install time` to use "timezone" specification.

## Running

To start a long-running process with scheduled cron job:

    mongodb_s3_backup <path to config file>

To execute a backup immediately and exit:

    mongodb_s3_backup -n <path to config file>
