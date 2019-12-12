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

The project is an `activator` webframework written in Scala and Knockout JS. It relies on an OR mapper automatically creating database tables from Scala classes. 

The project can locally be tested and built using the `activator` framework.

## Running local tests

For local tests the project currently creates an h2 database from scratch and runs all tests against this database.

For a deployed version the project creates a postgres database. Note that the databases behave slightly differently. Succeeding tests with the h2 database might not mean, that the tested point will behave identically with the postgres database.

# Docker build and deployment notes