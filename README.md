# Getting Started
Welcome to the log reader! This little app will return files in reversed order

## Running the service
Once you've cloned the repo, you can run it right away with default settings:

```
./gradlew clean build bootRun
```

You may have to first set the gradle wrapper to executable:

```
chmod +x gradlew
```

You can then pop over to http://localhost:8080/logs to take a look at the logs in the directory!

You can also take a look at http://localhost:8080/aggregate, but you'll need some config and other 
servers to make that useful

Take a look at the query options to see what else you can do 

If you'd like to use something other than the default settings, take a look at the 
configuration options to see what environment variables you can set to modify its behavior.

## Endpoints

Both endpoints below can take any query parameters from the query parameters section of this readme.

### /logs

This endpoint can be used to fetch the logs from the configured directory and return them as JSON with
the lines from the logs in reverse order

### /aggregate

This endpoint will hit the `/logs` endpoints of every server configured via `LOG_SERVERS`.

This will pass the provided query parameters (if any) down to called servers.

A note on testing: This does not prevent you from adding http://localhost:8080 alongside self in config,
while this does result in duplicate data, you can see it working as intended through that. You can also add
random URLs, but unless they serve a `/log` endpoint in the same format it will just result in an error message
for that URL in the returned object..

## Query Parameters
A couple of query parameters are available on the logs endpoint, see below for brief descriptions 
of them.

All query parameters are optional.

### fileName
You may specify a directory or file within the configured directory to return

**Example:** http://localhost:8080/logs?fileName=numberFile.txt

### logLines
You may specify a maximum number of lines to return for any given logfile. This will be applied to
all files returned

**Example:** http://localhost:8080/logs?logLines=10

### searchTerm
You may specify a search term which will act as a case-sensitive filter for log lines - only lines
containing that search term will be returned

**Example:** http://localhost:8080/logs?searchTerm=test

## Configuration Options
There's a few config options available to you when starting the application and can all 
be set as environment variables.

### LOG_DIRECTORY
This specifies the direction to scrape for logs. Users will be disallowed from requesting 
any files outside this directory.

This directory must be an absolute path.

**Default:** /var/log

### DEFAULT_LOG_LINE_LIMIT
This specifies the maximum number of lines to return if none is provided in the API.

This can be set to a negative number to disable this check and allow the API to request all lines from a log.

**Default:** Unlimited

### LOG_SERVER_NAME
A user-friendly name for the server if desired. If using aggregation, it's highly advised to set this.

**Default:** MyServer

### LOG_SERVERS
A comma seperated list of servers to hit when calling the aggregation endpoint.

Note that the special value `self` can be used to also fetch logs from the current server.

By default only `self` is set, making this endpoint essentially the same as the `/logs` endpoint.

The endpoint should not be specified - the app will automatically append `/logs` to the provided URLs.

**Default:** self

## API Fields

Below are a brief description of API fields

### errors
Where present, this is a list of errors encountered processing the request at a given level

### error
This only exists at the file level, and will contain a message if a file could not be processed

### fileName
The name of the file the log lines came from

### filePath
The relative path of the file from `LOG_DIRECTORY`

### serverName
The friendly name of the server, set by `LOG_SERVER_NAME`

### logLines
A list of lines from the given logfile, in reverse order (Bottom of the file will be the first line in the list)

## Limitations
The parser used for files currently only supports UTF-8 and single character encodings, and in fact
the app is setup to assume UTF-8 encoding for all files it encounters.

The app will ignore any files that it cannot determine are of type `text/plain` or with a `.log` or `.txt` 
file extension, since it likely can't parse anything useful from them.

Likewise this application is limited to the access level it's run at - some directories or log files may not
be accessible to it. While it will fail gracefully, these files will show up in the API with an error message.

## Performance
It takes approximately 84ms for the ReverseFileReader implementation to read all 196037 lines of 
Shakespeare's complete works into memory on the test machine.

## Planned Improvements
Currently on the roadmap we have a few items:

- Add a simple webpage to fetch and display results

## Wishlist
This is a time limited challenge and circumstances have limited it a bit further. Because of this,
a few things are on a wishlist I'd want to definitely get to if I had the time:

- Figure out local issues with docker and send out the app with a container
- Docker-compose especially for local testing of the aggregation functionality
- Try my hand at writing some log parsing as well