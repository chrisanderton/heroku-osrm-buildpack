#OSRM Buildpack for Heroku

Experimental buildpack for running [OSRM](http://project-osrm.org) on Heroku - this has not been tested in a production environment (or with any kind of usage beyond simple API queries!)

The Buildpack will download and compile OSRM and depending on the data files provided will extract and prepare the data if necessary and run `osrm-routed`.

##Example

```
git init
heroku apps:create
heroku buildpacks:set https://github.com/chrisanderton/heroku-osrm-buildpack
curl -L https://raw.githubusercontent.com/Project-OSRM/osrm-backend/develop/profiles/car.lua -o profile.lua
mkdir lib
curl -L https://raw.githubusercontent.com/Project-OSRM/osrm-backend/develop/profiles/lib/access.lua -o lib/access.lua
git add .
git commit -a -m "add profile"
heroku config:set OSRM_DATA_PBF_FILE_URL=http://download.geofabrik.de/europe/great-britain/england/greater-london-latest.osm.pbf
git push heroku master
```

##Usage

###OSRM Release Version

Uses OSRM release 4.8.1 by default - this can be overridden by setting the `OSRM_VERSION` config variable.

###Procfile

The buildpack will create a default Procfile if none is present.

###Data Files

There are a number of options for specifying the data files that will be used by `osrm-routed`.

####Data Files in the App's Git Repository

The root directory of the app's Git repository must match one of the following criteria for the detect phase to succeed:

- a PBF file called `data.pbf` AND a profile called `profile.lua`

*or*

- include extracted files `data.osrm`, `data.osrm.nodes`, `data.osrm.names`, etc AND a profile called `profile.lua`

*or* 

- include prepared files - `data.osrm.core`, `data.osrm.level`, etc AND a profile called `profile.lua`

This approach is only suitable for files up to a certain size - there is a limit on the overall 'slug' size supported by Heroku as well as timeouts applied to how long an application can take to build and start.

####Specify Config Variables for Data Files

To pass the detect phase the following file is required - note that if the file is present, deployment will proceed, but if the config variables are missing or incorrect deployment will fail:

- The root directory must contain a profile called `profile.lua`

To initiate a download of data files during the compile phase:

- A config variable must be set called `OSRM_DATA_PBF_FILE_URL` that contains a publically accessible URL to a PBF file

*or*

- A config variable must be set called `OSRM_DATA_BASE_URL` that contains a publically accessible _base URL_ to a set the files needed in the OSRM hierachy - e.g. the _base URL_ will be downloaded and concatenated with the necessary file extenstions (.core, .edges, etc) to download all files in the hierachy. You can provide files from either the _extract_ or _prepare_ phase of the OSRM pipeline.

This approach is only suitable for files up to a certain size - there is a limit on the overall 'slug' size supported by Heroku as well as timeouts applied to how long an application can take to build and start.

####Custom Startup Script

You could try creating a custom startup script that downloads the files before starting `osrm-routed`.

The main limitation here is the default 60 startup time imposed on dynos - during this time the dyno will need to download the files, extract/process if required and start osrm-routed.