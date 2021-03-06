# Jira worklog

[![Build Status](https://img.shields.io/travis/alexharv074/jira_worklog.svg)](https://travis-ci.org/alexharv074/jira_worklog)

This utility is intended to assist time-tracking given a requirement to log all work hours to Jira tickets.

Using Jira's point-and-click web UI can be quite slow when working simultaneously on many Jira tickets, and this utility helps in minimising the time spent on such time tracking and data entry.

We use the [Jira REST API version 2](https://docs.atlassian.com/jira/REST/latest/#api/2/) called via [Unirest](http://unirest.io/ruby.html).  [Webmocks](https://github.com/bblimke/webmock) is used in the tests.  Ruby version 2.0 or higher is required.

### Usage

To install:

```
$ git clone https://github.com/alexharv074/jira_worklog.git
$ cd jira_worklog.git
$ ./setup.sh
Password: <enter root password>
```

(The root password is required to copy the executable to `/usr/local/bin`.)

To upgrade:

```
$ ./setup.sh -upgrade
Password: <enter root password>
```

To run:

```
$ jira_worklog.rb --help
Usage: jira_worklog.rb [options]

Options:
    -f, --datafile DATAFILE          data file with worklog data
    -c, --configfile CONFIGFILE      file containing server, user name and infill
```

Specifying a data file:

```
$ jira_worklog.rb -f data.yml
```

### Config file

A config file is expected to be found in `~/.jira_worklog/config.yml` or specified on the command line with `-c`.

```yaml
---
server: 'jira.example.com'
username: 'fred'
infill: '8h'
```

The parameters are as follows:

#### `infill`

The `infill` option specifies the number of hours worked per day, and is used to calculate the number of hours to infill to the `default` ticket.  The default is `8h`.  To disable infilling, do not specify `default` in the data file (see below).

If you need to override infilling to the `default` ticket for one day only, this can be specified as:

```yaml
'2016-04-19':
- DEV-123:infill
```

If you wish to disable infilling for one day only, add a line `noinfill` as follows:

```yaml
'2016-04-19':
- DEV-123:4h
- noinfill
```

Note that infilling will never occur on Saturdays and Sundays, unless explicitly told to do so, although at this stage it will on public holidays.  Infilling will also not occur if the number of hours worked equals or exceeds `infill`.

In the event that a date is already in the state file and a new time entry is found in data (which would only happen if a previous date is modified in the data file, after it has already been loaded), infilling is assumed to have already taken place during the load, and is disabled for this day automatically.

#### `server`

The Jira server host name.

#### `username`

The username for logging into the Jira server.

#### `time_string`

This optional parameter can be used to change the default time of day or timezone (note that this time format is required by the Jira REST API).  It defaults to `T09:00:00.000+1000`, i.e. 9am in AEST timezone.  See [here](https://answers.atlassian.com/questions/241271/api-call-to-issuekeyworklog-always-fails-with-400-or-500-never-works) for more info about this.

### Data file

A data file is expected to be specified via `-f` and defaults otherwise to `~/.jira_worklog/data.yml`.

For example:

```yaml
---
default: 'PROJ-4123' # default ticket for infill.
worklog:
  '2016-04-15':
  - PROJ-4123:3h 30m
  - DEV-6233:3h 30m
  - DEV-6300:30m
  '2016-04-16': []   # infill all to default.
  '2016-04-17':
  - PROJ-4123:1h:Woken by pager  # add an optional comment.
  '2016-04-18':
  - PROJ-4123:1h
  - noinfill         # disable infilling for this day.
  '2016-04-19':
  - PROJ-4123:1h
  - DEV-6233:infill  # specify infill explicitly.
```

#### `default`

This is a default Jira ticket that can be used to log all remaining time against.  This is used in conjunction with the `infill` configuration option.  If default is not specified, infilling will not occur.

#### `worklog`

The worklog is a Hash of Arrays of `date:time` pairs, or optionally `date:time:comment` triplets.  The date must be in ISO 8601 date format.  We use a colon-separated string to minimise key strokes spent on data entry.

#### `noinfill`

If a worklog contains a row `noinfill`, infilling is disabled for that day only.  If you need to disable infilling everywhere, simply do not specify a default.

### State file

In order to guarantee idempotence (that is, running the script twice using the same input data should not result in the worklogs being added to Jira twice), a state file is created in `~/.jira_worklog/state.yml`.  Any attempt to add entries from `data.yml` already appearing in the state file will be silently ignored.

### Known issues

* No management of the state file.  You can safely delete it if it gets too big so long as you also rotate the data file at the same time.

### Contributing

PRs welcome!

To run the tests:

```
$ gem install bundler
$ bundle install
$ bundle exec rake spec
```

### Thanks!
