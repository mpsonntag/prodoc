# GCA-Web user documentation

## Normal user documentation

### Abstract submission

## Conference admin documentation

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

## Abstract submission workflow

Check `/app/models/AbstractState` for details.

## Class description


# Docker build and deployment notes