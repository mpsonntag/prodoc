# - GCA-Web user documentation

## -- Normal user documentation

### Registration workflow

### Abstract submission


## -- Conference admin documentation

### Create conference

### Conference workflow

### Abstract administration workflow

  val ownerStates = Map("isOpen"  -> Map(InPreparation -> (Submitted :: Nil),
                                         Submitted     -> (InPreparation :: Withdrawn :: Nil),
                                         InRevision    -> (Submitted :: Nil)),
                       "isClosed" -> Map(InRevision    -> (Submitted :: Nil)))

  val adminStates = Map(InPreparation -> (Submitted :: Nil),
                        Submitted  -> (InReview :: Nil),
                        InReview   -> (Accepted :: Rejected :: InRevision :: Withdrawn :: Nil),
                        Accepted   -> (InRevision :: Withdrawn :: Nil),
                        Rejected   -> (InRevision :: Withdrawn :: Nil),
                        InRevision -> (InReview :: Nil))

### Conference schedule JSON description

Three main nested objects are supported: `session`, `tracks`, `events`

`events` can be nested within `tracks`
`tracks` can be nested within `sessions`

- `session` object has supported keys:
  `title`, `subtitle`, `tracks`
  - `title` and `subtitle` values are text
  - `tracks` value is an object array containing `track` objects

- `track` object has supported keys:
  `title`, `subtitle`, `chair`, `events`
  - `title` and `subtitle` values are text
  - `chair` value is an array of text
  - `events` value is an object array containing `event` objects

- `event` object has supported keys:
  `title`, `subtitle`, `start`, `end`, `date`, `location`, `authors`, `type`, `abstract`
  - all values are text
  - `start` and `end` time values have to be in format `HH:mm`.
  - `date` value has to be in format `YYYY-MM-DD`

## Minimal schedule json example

```
[
  {
    "title": "",
    "subtitle": "",
    "tracks": [
      {
        "title": "Session title",
        "subtitle": null,
        "chair": [""],
        "events": [
          {
            "title": "",
            "subtitle": null,
            "start": "",
            "end": "",
            "date": "",
            "location": "",
            "type": "",
            "authors": [""]
          }
        ]
      }
    }
  }
]
```

# GCA-Web dev documentation

## Project structure

- `/app` contains backend and frontend code.
- `/conf` contains the API routes and service configuration files
- `/doc` contains user and developer documentations
- `/patch` contains database patch files for version transitions
- `/project` sbt dependencies
- `/public` fonts, logos and javascript libraries
- `/test` folder containing all tests

## Abstract submission workflow

Check `/app/models/AbstractState` for details.

## Class description

# Build and debug notes

The project is a [play v2.3](https://www.playframework.com/documentation/2.3.x/) webframework written in Scala and Knockout JS. It relies on an OR mapper automatically creating database tables from Scala classes. 

The project can locally be tested and built using the play framework. It requires Java 8 (OpenJDK 1.8) and the sbt build tool `activator` v1.3.7. Which can be downloaded [here](https://downloads.typesafe.com/typesafe-activator/1.3.7/typesafe-activator-1.3.7-minimal.zip).

## Running local tests

For local tests the project currently creates an h2 database from scratch and runs all tests against this database.

For a deployed version the project creates a postgres database. Note that the databases behave slightly differently. Succeeding tests with the h2 database might not mean, that the tested point will behave identically with the postgres database.

# Docker build and deployment notes