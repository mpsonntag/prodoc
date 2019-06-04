
NWB:N and NIX have similar goals but apply fundamentally different
approaches. The NIX concept is much more general than the NWB
approach. NWB:N standardizes the items and the related information
that are stored by providing individual structures for each item while
NIX standardizes the way how scientific data and metadata objects are
stored. That is, NWB focuses on specifying individual data items and
where they should be stored while NIX focuses on providing a
framework/data model to represent scientific data and
metadata objects and their relations. NIX has the benefit that scientists
can store a multitude of data types with only a small number of
entities.

In NIX both the data and the metadata parts support standardization
. NIX libraries can be included in data recording tools as well as data analysis scripts for automated processing. On top of the "low-level" APIs so far there are two
"high-level" APIs for persisting and reading NEO structures and
EEG data. Any tool, that works for example on NEO objects can automatically handle NIX persisted NEO objects. It is, and we tested this during a recent NWB
Hackathon, possible to represent NWB entities in NIX. The other way
around can not be guaranteed in case the respective definitions have
not been made on the NWB side, because NIX can store a larger multitude of data types than NWB.

The statement by the NWB:N developers suggest that flexibility is a
disadvantage. We would agree if the flexibility were
arbitrary. However, NIX specifies a well-defined data model so that
stored data are both understandable for humans as well as machine
actionable. Core properties of data and metadata such as data types,
dimensions, labels and units are mandatory and their presence
can be enforced and validated, such that an automated tool can
always access and process the data appropriately.
We argue that the flexibility of the NIX approach is ultimately needed and
is thus a special design feature rather than a disadvantage. For example, using NIX as storage backend for data acquisition requires this flexibility. The
recording software relacs (www.relacs.net) is highly configurable and
user extensible, it dynamically adapts e.g.  stimulus parameters to
the current recording. It thus has to be possible to store parameters
that are specific to a given experiment and that might even change
from trial to trial. User customization goes down to completely new
protocols that may provide and depend on parameters that could not be
foreseen during the development of the tool, or the storage backend,
itself.  Likewise, every experiment is different and scientists use
not only simple and stereotyped methods, but often create multimodal
data and metadata with complex relationships in a single experiment.
With NIX the scientist has a manageable set of standardized,
machine-actionable objects to describe the data, metadata, and their
relations according to the structure of the experiment.  The larger flexibility of NIX is definitely seen as a benefit by
users (see for example the letter of support for NIX provided by
Andrew Davison).

Both data and metadata parts can be validated and thus
interoperability can be ensured. In contrast to what is suggested by
the statement of the NWB developers, NWB is not superior to NIX with
respect to automated processing.  The mandatory information in NWB
does not exceed the information that NIX also requires (basically the
data definitions). Many other information in NWB:N can be given, or
can be left away. The example datasets
(https://neurodatawithoutborders.github.io/exampledata) often give
"UNKNOWN" or "PLACEHOLDER" for metadata information. If this is the
case, any tool relying on such information will fail despite strict
structural definitions. Further, to our knowledge, many parameters are
stored as text. Interpreting them correctly requires parsing of the
string and searching e.g. for units (Hz, kHz? seconds, or
milliseconds?). Things like units are explicit in NIX
and are handled appropriately by the library. Moreover, in NIX each metadata entity can be linked to an ontology
if available, and each data entity has a type that likewise can be
linked to a definition to provide semantic meaning.

The comments of the NWB developers suggest a notion on standardization
and automation that is valid, but not the only one. With the NIX data
model it is perfectly possible to perform automated data processing,
including automated selection of the parts of the data relevant for a
given analysis, such as periods of stimulation in recorded responses,
regions of interest in images, etc. At the same time it provides the
necessary flexibility to account for current and future variety of
experimental paradigms and thus dataset structures.

We think that the approaches to standardization that NWB and NIX
are taking, while being diametrically opposite, are both valid
and valuable to the community.
