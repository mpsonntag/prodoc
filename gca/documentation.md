# - GCA-Web user documentation

## -- Normal user documentation

### Registration workflow

### Abstract submission


## -- Conference admin documentation

### Create conference

Admin panel - main tab field description

- Submission ... abstracts can be submitted; sole option to activate and inactivate abstract submission.
- Published ... the conference will appear in the lower part of the conference list on the main page.
- Active ... the conference will appear in the top part of the conference list of the main page.
NOTE: if neither Published or Active are selected, there will be no link to access this conference. Use abstracts.g-node.org/conference/[conference short] to access an unpublished, inactive conference.
- The banner (via logo link or logo file) on the main conference page will only be displayed, if the 'Link' to an external page has been provided, otherwise the banner will not be shown.
- Name ... long name of the conference
- Short ... short name of the conference e.g. BC14 -> is required and must be unique
- Group ... ???
- Cite ... text that will be displayed in the "my abstracts" and "my favorites" abstract lists [xxx]
- Start ... start date of the conference; displayed on the main abstracts page and the main conference page
- end ... end date of the conference; displayed on the main abstracts page and the main conference page
- deadline ... end date of the abstracts submission; it does not end abstract submission automatically. to end abstract submission, untick the Submission checkbox.
- logo file ... upload an image that will be displayed on the main page if the conference is active and on the main conference page if a conference link has been provided
- logo link ... link to a hosted image; the file must be hosted within the same domain (e.g. g-node). if a file has been uploaded via the logo file, logo link will be ignored. will be removed in later versions.
- thumbnail file ... upload an image that will be displayed on the main page if the conference is published but not active.
- thumbnail link ... link to a hosted image; the file has to be hosted within the same domain (e.g. g-node). if a file has been uploaded via the thumbnail file, thumbnail link will be ignored. will be removed in later version.
- iOSApp ... id of the ios gca-web app for this conference. will be discontinued.
- Link ... link to the external conference web page. if this link is provided, the logo image will be displayed on the main conference page and top navigation will feature a "Conference home" link. It will also show up in the abstract submission form to remind people after an abstract submission to also sign up for the conference itself.
- Description ... markdown capable text that is displayed on the main page if the conference is active and on the main conference page.
- Notice ... markdown capable text that is always displayed on the top of every page within the scope of the conference for last minute information for the conference attendees.
- Presentation preferences ...
  - this will enable presentation preferences in the abstract submission.
  - NOTE the specifics have to be entered in the "Groups" tab of the Admin panel; otherwise there will be an empty popup in the abstract submission form.
  - Only one presentation preference can be selected when submitting an abstract.
  - The corresponding field is called "Presentation type" in the abstract submission form.
  - presentation preferences will be the categories that are displayed on the main abstract list page.
  - NOTE abstracts have to be manually downloaded, ordered and uploaded to these lists via the GCA-Python command line tool.
- Topics ...
  - topics that can be used to specify keywords for poster abstracts.
  - NOTE! even if topics are added, they have to be saved via the "save" button at the bottom of the page!
  - Only one topic can be selected per abstract.
- Max. Abs. Len ... Maximum abstract length; this is a required field and cannot be zero. It defines the allowed character length of an abstract including whitespaces.
- Max. Figures ... Maximum number of abstracts that can be added to an abstract.

Groups description:
- these are the display categeories (presentation preferences) of published abstracts.
- prefix is a number that defines the display order
- short is a string that is displayed in the abstract list with every abstract to note which category an abstract belongs to. it is also displayed on the abstract display page itself to again note the presentation pereference.
- long is is the text that is displayed as the tab text on the abstract list page


### Conference workflow

### Abstract administration workflow
- As long as your abstract is `InPreparation`, you can make any changes you like. You will have to save the abstract manually once, after that any change you make will be automatically saved.
- When you submit your abstract, the state of your abstract will change to `Submitted`. This means that you cannot make changes to your abstract, signalling to the reviewers, that there will be no more changes from your end and they can begin the review process.
- If you still need to add changes to your abstract, you can set the state of your abstract back to `InPreparation` again, if it is in state `Submitted`, signalling to the reviewers that changes to the abstract will still happen.
- Reviewers will set the state of an abstract from `Submitted` to `InRevision`. Submitters cannot make any changes to their abstract in this state.
- Reviewers can now set the state of the abstract to any of the following states: `Accepted`, `Rejected`, `InRevision`. Both states `Accepted` and `Rejected` 

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