regarding links: hardlinks are used, since we know how many hardlinks there are, pointing towards an 
object, but not how many softlinks point there... the object then could not be gc'd when the "last"
softlink is removed...

mx      Entity.h (IEntity) |EntityHDF5.h| {EntityHDF5.c}
        {entity_id}                     // H5G.a=H5Group.attribute
        {updated_at}                    // H5G.a
        {created_at}                    // H5G.a

        |(+)|   id()                    ->s
        |(+)|   updatedAt()             ->t=time
        |(+)|   createdAt()             ->t
        |(+)|   setUpdatedAt()          <-v=void
        |(+)|   forceUpdatedAt()        <-v
        |(+)|   setCreatedAt()          <-v
        |(+)|   forceCreatedAt(t)       <-v
        |(+)|   isValidEntity()         ->b=bool          {group().referenceCount()>0}
        +||     group()                 ->H5Group
        ~||     file()                  ->ptr


mx       NamedEntity.h (INamedEntity) |NamedEntityHDF5.h| {NamedEntityHDF5.c}
        {name}                          // H5G.a
        {type}                          // H5G.a
        {definition}                    // H5G.a

        |(+)|   type(s)                 <-v
        |(+)|   type()                  ->s
        |(+)|   name()                  ->s
        |(+)|   definition(s)           <-v
        |(+)|   definition()            ->s
        |(+)|   definition(n)           <-v // n=none, deleter
        |(+)|   compare(NamedEntity)    ->d=int


mx       EntityWithMetaData.h (IEntityWithMetaData) |EntityWithMetaDataHDF5.h| {EntityWithMetaDataHDF5.c}
        {metadata}                      // H5Group

        |(+)|   metadata()              ->Section           {group().openGroup("metadata")}
          +     metadata(Section)       <-v                 backend.metadata(Section.id())
        |(+)|   metadata(id)            <-v // id=s         {group().createLink(file().findSection(id).group(), "metadata")}    // H5Lcreate_hard, can only ever be one per Entity
        |(+)|   metadata(n)             <-v                 {group().removeGroup("metadata")}                                   // H5Gunlink


mx       EntityWithSources.h (IEntityWithSources) |EntityWithSourcesHDF5.h| {EntityWithSourcesHDF5.c}
        -||     sources                 // H5Group optGroup

        |(+)|   sourceCount()           ->d                 // d=any int
        |(+)|   hasSource(n_id)         ->b                 {sources.hasGroup(n_id)}
          +     hasSource(Source)       ->b                 backend.hasSource(Source.id())
        |(+)|   getSource(n_id)         ->Source            {sources.openGroup(block().findSources(n_id).id())}
        |(+)|   getSource(idx)          ->Source            //idx=d
          +     sources(filter)         -><Source>
        |(+)|   sources(<Source>)       <-v                 {for each Source call addSource(Source.id())}
        |(+)|   addSource(n_id)         <-v                 {sources.createLink(block().findSources(n_id).group(), n_id)}       // H5Lcreate_hard
          +     addSource(Source)       <-v                 backend.addSource(Source.id())
        |(+)|   removeSource(n_id)      <-b                 {sources.removeGroup(n_id)}                                         // H5Gunlink
          +     removeSource(Source)    <-b                 backend.removeSource(Source.id())
        +||     block()                 ->ptr



// How does H5Group.openGroup actually create a new group with name and id? uses H5Pcreate & H5Gcreate2
// with the "name" it gets, but how is this also linked with an additionally created UUID?
// Why is this creation process different than createLink from EntityXXX (H5Lcreate_hard)

mx       File.h (IFile) |FileHDF5| {FileHDF5.c}
        {format}                            // H5G.a
        {version}                           // H5G.a
        {updated_at}                        // H5G.a
        {created_at}                        // H5G.a

        -||     root, metadata, data        // H5Groups "root" directory, containing one directory "data", one "metadata"

x         +     open(name, mode, backend)   -> //name=s mode=FileMode, backend=...,HDF5
        |(+)|   flush()                     ->b // persist changes

x       |(+)|   blockCount()                ->d
x       |(+)|   hasBlock(n_id)              ->b // n_id=s       {getBlock(n_id)}
-         +     hasBlock(Block)             ->b                 backend.hasBlock(Block.id())
x       |(+)|   getBlock(n_id)              ->Block             {data.findGroupByNameOrAttribute("entity_id", n_id)}
x       |(+)|   getBlock(idx)               ->Block
x       |(+)|   createBlock(name, type)     <-Block             {data.openGroup(name)}                          // H5Pcreate & H5Gcreate2, created with name, not id
x       |(+)|   deleteBlock(n_id)           <-b                 {data.removeAllLinks(getBlock(n_id).name())}    // H5Ldelete
-         +     deleteBlock(Block)          <-b                 backend.deleteBlock(Block.id())
x         +     blocks(filter)              -><Block>
x         +     blocks()                    -><Block>

x       |(+)|   hasSection(n_id)            ->b                 {getSection(n_id)}
-         +     hasSection(Section)         ->b                 backend.hasSection(Section.id())
x       |(+)|   getSection(n_id)            ->Section           {metadata.findGroupByNameOrAttribute("entity_id", n_id)}
x       |(+)|   getSection(idx)             ->Section
x       |(+)|   sectionCount()              ->d
x         +     sections(filter)            -><Section>
x         +     sections()                  -><Section>
x         +     findSections(filter, depth) -><Section>         // depth=int
x         +     findSections(depth)         -><Section>
x       |(+)|   createSection(name, type)   <-Section           {metadata.openGroup(name)}                      // H5Pcreate & H5Gcreate2, created with name, not id
x       |(+)|   deleteSection(n_id)         <-b                 {metadata.removeAllLinks(section.name())}       // H5Ldelete, also deletes all child sections by child.id()
-         +     deleteSection(Section)      <-b                 backend.deleteSection(Section.id())

x       |(+)|   version()                   -><d>
x       |(+)|   format()                    ->s
x       |(+)|   location()                  ->s                 // URI
x       |(+)|   createdAt()                 ->t
x       |(+)|   updatedAt()                 ->t
        |(+)|   setUpdatedAt()              <-v
        |(+)|   forceUpdatedAt()            <-v
        |(+)|   setCreatedAt()              <-v
        |(+)|   forceCreatedAt(t)           <-v
        |(+)|   close()                     <-v
x       |(+)|   isOpen()                    ->b
x       |(+)|   fileMode()                  ->FileMode
x         +     validate()                  ->Result            // Result contains warning and error messages
        -||     fileExists()                ->b
        -||     openRoot()                  <-v
        -||     checkHeader(FileMode)       ->b
        -||     createHeader()              <-v


mx       Block.h (IBlock) |BlockHDF5.h| {BlockHDF5.c}
        ?{}     data_arrays, tags, multi_tags, sources, groups      // H5Groups, optional

x       |(+)|   hasSource(n_id)             ->b
-         +     hasSource(Source)           ->b                     backend.getSource(Source.name())
x       |(+)|   getSource(n_id)             ->Source                {sources.findGroupByNameOrAttribute("entity_id", n_id)}
x       |(+)|   getSource(idx)              ->Source
x       |(+)|   sourceCount()               ->d
x         +     sources(filter)             -><Source>
x         +     findSources(filter, depth)  -><Source>
x       |(+)|   createSource(name, type)    <-Source                {sources.openGroup(name)}               // H5Pcreate & H5Gcreate2, created with name, not id
x       |(+)|   deleteSource(n_id)          <-b                     {sources.removeAllLinks(block().getSource(n_id).name())}            // H5Ldelete, children removed by child.id()
-         +     deleteSource(Source)        <-b                     backend.deleteSource(Source.name())

x       |(+)|   hasDataArray(n_id)          ->b
-         +     hasDataArray(DataArray)     ->b                     backend.getDataArray(DataArray.name())
x       |(+)|   getDataArray(n_id)          ->DataArray             {data_arrays.findGroupByNameOrAttribute("entity_id", n_id)}
x       |(+)|   getDataArray(idx)           ->DataArray
x         +     dataArrays(filter)          -><DataArray>
x       |(+)|   dataArrayCount()            ->d
x       |(+)|   createDataArray(name, type, DataType, size)     <-DataArray     {data_arrays.openGroup(name) + newDA.group().createData("data")}  // H5Pcreate & H5Gcreate2 + H5Pcreate & H5Dcreate
-         +     createDataArray(name, type, <data>, DataType)   <-DataArray
x       |(+)|   deleteDataArray(n_id)       <-b                     {data_arrays.removeAllLinks(block().getDataArray(n_id).name())} // H5Ldelete
-         +     deleteDataArray(DataArray)  <-b                     backend.deleteDataArray(DataArray.name())

x       |(+)|   hasTag(n_id)                ->b
-         +     hasTag(Tag)                 ->b                     backend.getTag(Tag.name())
x       |(+)|   getTag(n_id)                ->Tag                   {tags.findGroupByNameOrAttribute("entity_id", n_id)}
x       |(+)|   getTag(idx)                 ->Tag
x         +     tags(filter)                -><Tag>
x       |(+)|   tagCount()                  ->d
x       |(+)|   createTag(name, type, position) <-Tag // position=<double>  {tags.openGroup(name)}          // H5Pcreate & H5Gcreate2, created with name, not id
x       |(+)|   deleteTag(n_id)             <-b                     {tags.removeAllLinks(block().getTag(n_id).name())}             // H5Ldelete
-         +     deleteTag(Tag)              <-b                     backend.deleteTag(Tag.name())

x       |(+)|   hasMultiTag(n_id)           ->b
-         +     hasMultiTag(Tag)            ->b                     backend.getMultiTag(MultiTag.name())
x       |(+)|   getMultiTag(n_id)           ->MultiTag              {multi_tags.findGroupByNameOrAttribute("entity_id", n_id)}
x       |(+)|   getMultiTag(idx)            ->MultiTag
x         +     multiTags(filter)           -><MultiTag>
x       |(+)|   multiTagCount()             ->d
x       |(+)|   createMultiTag(name, type, positions)   <-MultiTag // positions=DataArray   {multi_tags.openGroup(name)}            // H5Pcreate & H5Gcreate2, created with name, not id
x       |(+)|   deleteMultiTag(n_id)        <-b                     {multi_tags.removeAllLinks(block().getMultiTag(n_id).name())}   // H5Ldelete
-         +     deleteMultiTag(MultiTag)    <-b                     backend.deleteMultiTag(MultiTag.name())

x       |(+)|   hasGroup(n_id)              ->b
-         +     hasGroup(Group)             ->b                     backend.getGroup(Group.name())
x       |(+)|   getGroup(n_id)              ->Group                 {groups.findGroupByNameOrAttribute("entity_id", n_id)}
x       |(+)|   getGroup(idx)               ->Group
x         +     groups(filter)              -><Group>
x       |(+)|   groupCount()                ->d
x       |(+)|   createGroup(name, type)     <-Group                 {groups.openGroup(name)}                // H5Pcreate & H5Gcreate2, created with name, not id
x       |(+)|   deleteGroup(n_id)           <-b                     {groups.removeAllLinks(block().getGroup(n_id).name())}           // H5Ldelete
-         +     deleteGroup(Group)          <-b                     backend.deleteGroup(Group.name())
        +||     block()                     ->ptr

        +Entity +NamedEntity +EntityWithMetaData
x (desc)|(+)|   id()                    ->s
x (desc)|(+)|   name()                  ->s
x (desc)|(+)|   type()                  ->s
x       |(+)|   type(s)                 <-v
x       |(+)|   definition(s)           <-v
x (desc)|(+)|   definition()            ->s
x       |(+)|   definition(n)           <-v
x       |(+)|   compare(NamedEntity)    ->d

x       |(+)|   metadata()              ->Section
-         +     metadata(Section)       <-v
x       |(+)|   metadata(id)            <-v
x       |(+)|   metadata(n)             <-v


mx       Group.h (IGroup) |GroupHDF5.h| {GroupHDF5.c}
        -||     data_array, tag, multi_tag  // H5Group

x       |(+)|   hasDataArray(n_id)          ->b                 {data_array.hasGroup(block().getDataArray(n_id).id())}
-         +     hasDataArray(DataArray)     ->b                 backend.hasDataArray(DataArray.id())
x       |(+)|   dataArrayCount()            ->d
x       |(+)|   getDataArray(n_id)          ->DataArray         {data_array.getGroup(block().getDataArray(n_id).id())}
x       |(+)|   getDataArray(idx)           ->DataArray
-         +     addDataArray(DataArray)     <-v // v=void       backend.addDataArray(DataArray.id())
x       |(+)|   addDataArray(n_id)          <-v                 {data_array.createLink(block().getDataArray(n_id).group(), block().getDataArray(n_id).id())}    // H5Lcreate_hard
-         +     removeDataArray(DataArray)  <-b                 backend.removeDataArray(DataArray.id())
x       |(+)|   removeDataArray(n_id)       <-b                 {data_array.removeGroup(block().getDataArray(n_id).id())}                   // H5Gunlink
x         +     dataArrays(filter)          -><DataArray>
x         +     dataArrays()                -><DataArray>
x       |(+)|   dataArrays(<DataArray>)     <-v                 {for each addDataArray(DA.id())}

x       |(+)|   hasTag(n_id)                ->b                 {tag.hasGroup(block().getTag(n_id).id())}
-         +     hasTag(Tag)                 ->b                 backend.hasTag(Tag.id())
x       |(+)|   tagCount()                  ->d
x       |(+)|   getTag(n_id)                ->Tag               {tag.openGroup(block().getTag(n_id).id())
x       |(+)|   getTag(idx)                 ->Tag
x       |(+)|   addTag(n_id)                <-v                 {tag.createLink(block().getTag(n_id).group(), block().getTag(n_id).id()}    // H5Lcreate_hard
-         +     addTag(Tag)                 <-v                 backend.addTag(Tag.id())
x       |(+)|   removeTag(n_id)             <-b                 {tag.removeGroup(block().getTag(n_id).id())}                                // H5Gunlink
-         +     removeTag(Tag)              <-b                 backend.removeTag(Tag.id())
x         +     tags(filter)                -><Tag>
x         +     tags()                      -><Tag>
x       |(+)|   tags(<Tag>)                 <-v                 {for each addTag(Tag.id())}

x       |(+)|   hasMultiTag(n_id)           ->b                 {multi_tag.hasGroup(block().getMultiTag(n_id).id())}
-         +     hasMultiTag(MultiTag)       ->b                 backend.hasMultiTag(MultiTag.id())
x       |(+)|   multiTagCount()             ->d
x       |(+)|   getMultiTag(n_id)           ->MultiTag          {multi_tag.openGroup(block().getMultiTag(n_id).id())}
x       |(+)|   getMultiTag(idx)            ->MultiTag
x       |(+)|   addMultiTag(n_id)           <-v                 {multi_tag.createLink(block().getMultiTag(n_id).group(), block().getMultiTag(n_id).id())}   // H5Lcreate_hard
-         +     addMultiTag(MultiTag)       <-v                 backend.addMultiTag(MultiTag.id())
x       |(+)|   removeMultiTag(n_id)        <-b                 {multi_tag.removeGroup(block().getMultiTag(n_id).id())}                       // H5Gunlink
-         +     removeMultiTag(MultiTag)    <-b                 backend.removeMultiTag(MultiTag.id())
x         +     multiTags(filter)           -><MultiTag>
x         +     multiTags()                 -><MultiTag>
x       |(+)|   multiTags(<MultiTag>)       <-v                 {for each addMultiTag(MultiTag.id())}

        +Entity +NamedEntity +EntityWithMetaData +EntityWithSources
x (desc)|(+)|   id()                    ->s
x (desc)|(+)|   name()                  ->s
x (desc)|(+)|   type()                  ->s
x       |(+)|   type(s)                 <-v
x       |(+)|   definition(s)           <-v
x (desc)|(+)|   definition()            ->s
x       |(+)|   definition(n)           <-v
x       |(+)|   compare(NamedEntity)    ->d

x       |(+)|   metadata()              ->Section
-         +     metadata(Section)       <-v
x       |(+)|   metadata(n_id)          <-v
x       |(+)|   metadata(n)             <-v
x       |(+)|   sourceCount()           ->d
x       |(+)|   hasSource(n_id)         ->b
-         +     hasSource(Source)       ->b
x       |(+)|   getSource(n_id)         ->Source
x       |(+)|   getSource(idx)          ->Source
x         +     sources(filter)         -><Source>
x       |(+)|   sources(<Source>)       <-v
x       |(+)|   addSource(n_id)         <-v
-         +     addSource(Source)       <-v
x       |(+)|   removeSource(n_id)      <-b
-         +     removeSource(Source)    <-b


mx      DataArray.h (IDataArray) |DataArrayHDF5.h| {DataArrayHDF5.cpp}
        {label}                                     // H5G.a
        {unit}                                      // H5G.a
        {expansion_origin}                          // H5G.a
        {polynom_coefficients}                      // DataSet -> LocID -> H5Object
        ?{} dimension                               // H5Group. optGroup

x (desc)|(+)|   label()                             ->s
x       |(+)|   label(s)                            <-v
x       |(+)|   label(n)                            <-v
x (desc)|(+)|   unit()                              ->s
x       |(+)|   unit(s)                             <-v
x       |(+)|   unit(n)                             <-n
-       |(+)|   expansionOrigin()                   ->d
-       |(+)|   expansionOrigin(d)                  <-v
-       |(+)|   expansionOrigin(n)                  <-v
-       |(+)|   polynomCoefficients(<d>)            <-v                     {group().createData("polynom_coefficients")}    // H5Pcreate & H5Dcreate
x (desc)|(+)|   polynomCoefficients()               -><d>
-       |(+)|   polynomCoefficients(n)              <-v                     {group().removeData("polynom_coefficients")}    // H5Gunlink
-         +     dimensions(filter)                  -><Dimension>
x         +     dimensions()                        -><Dimension>
x       |(+)|   dimensionCount()                    ->d
x       |(+)|   getDimension(idx)                   ->Dimension
x         +     appendSetDimension()                <-SetDimension
x         +     appendRangeDimension(<d>)           <-RangeDimension
x         +     appendAliasRangeDimension()         <-RangeDimension
x         +     appendSampledDimension(d)           <-SampledDimension
x       |(+)|   createSetDimension(idx)             <-SetDimension          {createDimensionGroup(idx)}                         // H5Pcreate & H5Gcreate2
x       |(+)|   createRangeDimension(idx, <d>)      <-RangeDimension        {createDimensionGroup(idx)}                         // H5Pcreate & H5Gcreate2
x       |(+)|   createAliasRangeDimension()         <-RangeDimension        {createDimensionGroup(1)}                           // H5Pcreate & H5Gcreate2
x       |(+)|   createSampledDimension(idx, d)      <-SampledDimension      {createDimensionGroup(idx)}                         // H5Pcreate & H5Gcreate2
x       |(+)|   deleteDimensions()                  <-b                     {for every dimension: dimensions.removeGroup()}     // H5Gunlink
          +     getDataDirect(DataType, data, count, offset)    <-v         // {read(dtype, data, count, offset)}
          +     setDataDirect(DataType, data, count, offset)    <-v         // {write(dtype, data, count, offset)}
x (desc)|(+)|   dataExtent()                        ->d                     {group().openData("data").size()}
x       |(+)|   dataExtent(d)                       <-v                     {group().openData("data").setExtent}                // H5Dset_extent
x       |(+)|   dataType()                          ->DataType              {group().openData("data").dataType()}               // H5Dget_type
x         +     appendData(DataType, data, count, axis) <-v
          ~     ioRead
          ~     ioWrite
        +|()|   createData(DataType, size)          <-v                     {group().createData("data")}                        // H5Pcreate & H5Dcreate
        +|()|   hasData()                           ->b                     {group().hasData("data")}
        +|()|   write(dtype, data, count, offset)   <-v
        +|()|   read(dtype, buffer, count, offset)  <-v                     // read data into buffer
        -||     createDimensionGroup(idx)           <-H5Group               {dimension.openGroup(idx)}                          // H5Pcreate & H5Gcreate2

        +Entity +NamedEntity +EntityWithMetaData +EntityWithSources
x (desc)|(+)|   id()                    ->s
x (desc)|(+)|   name()                  ->s
x (desc)|(+)|   type()                  ->s
x       |(+)|   type(s)                 <-v
x       |(+)|   definition(s)           <-v
x (desc)|(+)|   definition()            ->s
x       |(+)|   definition(n)           <-v
x       |(+)|   compare(NamedEntity)    ->d

x       |(+)|   metadata()              ->Section
-         +     metadata(Section)       <-v
x       |(+)|   metadata(n_id)          <-v
x       |(+)|   metadata(n)             <-v

x       |(+)|   sourceCount()           ->d
x       |(+)|   hasSource(n_id)         ->b
-         +     hasSource(Source)       ->b
x       |(+)|   getSource(n_id)         ->Source
x       |(+)|   getSource(idx)          ->Source
x         +     sources(filter)         -><Source>
x       |(+)|   sources(<Source>)       <-v
x       |(+)|   addSource(n_id)         <-v
-         +     addSource(Source)       <-v
x       |(+)|   removeSource(n_id)      <-b
-         +     removeSource(Source)    <-b


mx      Source.h (ISource) |SourceHDF5.h| {SourceHDF5.c}
        -||     sources                     // H5Group

x       |(+)|   hasSource(n_id)             ->b
-         +     hasSource(Source)           ->b                 backend.hasSource(Source.id())
x       |(+)|   getSource(n_id)             ->Source            {sources.findGroupByNameOrAttribute("entity_id", n_id)}
x       |(+)|   getSource(idx)              ->Source
x       |(+)|   sourceCount()               ->d
x         +     sources(filter)             -><Source>
x         +     findSources(filter, depth)  -><Source>
x       |(+)|   createSource(name, type)    <-Source            {sources.openGroup(name)}                       // H5Pcreate & H5Gcreate2, created with name, not id
x       |(+)|   deleteSource(n_id)          <-b                 {sources.removeAllLinks(source.name())}         // H5Ldelete, also deletes all child sources by child.id()
-         +     deleteSource(Source)        <-b                 backend.deleteSource(Source.id())
x         +     parentSource()              ->Source

x         +     referringDataArray()        -><DataArray>
x         +     referringTags()             -><Tag>
x         +     referringMultiTags()        -><MultiTag>
          +|()| parentFile()                ->File
          +|()| parentBlock()               ->Block

          +Entity +NamedEntity +EntityWithMetaData
x (desc)|(+)|   id()                    ->s
x (desc)|(+)|   type()                  ->s
x       |(+)|   type(s)                 <-v
x (desc)|(+)|   name()                  ->s
x       |(+)|   definition(s)           <-v
x (desc)|(+)|   definition()            ->s
x       |(+)|   definition(n)           <-v
x       |(+)|   compare(NamedEntity)    ->d

x       |(+)|   metadata()              ->Section
-         +     metadata(Section)       <-v
x       |(+)|   metadata(n_id)          <-v
x       |(+)|   metadata(n)             <-v


mx      Tag.h (ITag) |TagHDF5.h| [IBaseTag] /BaseTagHDF5.h/ {BaseTagHDF5.c && TagHDF5.c}
        {units}                                     // DataSet -> LocID -> H5Object
        {position}                                  // DataSet -> LocID -> H5Object
        {extent}                                    // DataSet -> LocID -> H5Object
        -{} features, references                    // H5Group. optGroup

x (desc)|(+)|   units()                             -><s>                   {group().getData("units")}
x       |(+)|   units(<s>)                          <-v                     {group().setData("units")}                                  // H5Pcreate & H5Dcreate
x       |(+)|   units(n)                            <-v                     {group().removeData("units")}                               // H5Gunlink
x (desc)|(+)|   position()                          -><d>                   {group().getData("position")}
x       |(+)|   position(<d>)                       <-v                     {group().setData("position")}                               // H5Pcreate & H5Dcreate
x       |(+)|   extent(<d>)                         <-v                     {group().getData("extent")}
x (desc)|(+)|   extent()                            -><d>                   {group().setData("extent")}                                 // H5Pcreate & H5Dcreate
x       |(+)|   extent(n)                           <-v                     {group().removeData("extent")}                              // H5Gunlink

x       /[+]/   hasReference(n_id)                  ->b                     {references.hasGroup(block().getDataArray(n_id).id())}
-         +     hasReference(DataArray)             ->b                     backend.hasReference(DataArray.name())
x       /[+]/   referenceCount()                    ->d
x       /[+]/   getReference(n_id)                  ->DataArray             {references.openGroup(block().getDataArray(n_id).id())}
x       /[+]/   getReference(idx)                   ->DataArray
x       /[+]/   addReference(n_id)                  <-v                     {references.createLink(block().getDataArray(n_id).id())}    // H5Lcreate_hard
-         +     addReference(DataArray)             <-v                     backend.addReference(DataArray.name())
x       /[+]/   removeReference(n_id)               <-b                     {references.removeGroup(block().getReference(n_id).id())}   // H5Gunlink
-         +     removeReference(DataArray)          <-b                     backend.addReference(DataArray.name())
x         +     references(filter)                  -><DataArray>
x         +     references()                        -><DataArray>
x       /[+]/   references(<DataArray>)             <-v                     {for each DA references.createLink(block().getDataArray(DA.id()).id())} // H5Lcreate_hard

x       /[+]/   hasFeature(n_id)                    ->b
-         +     hasFeature(Feature)                 ->b                     backend.hasFeature(Feature.id())
x       /[+]/   featureCount()                      ->d
x       /[+]/   getFeature(n_id)                    ->Feature
x       /[+]/   getFeature(idx)                     ->Feature
x         +     features(filter)                    -><Feature>
-         +     createFeature(DataArray, LinkType)  <-Feature               backend.createFeature(Feature.id())
x       /[+]/   createFeature(n_id, LinkType)       <-Feature               {features.openGroup(util::createId())}                      // H5Pcreate & H5Gcreate2, n_id ... id/name of an existing DA
x       /[+]/   deleteFeature(n_id)                 <-b                     {features.removeGroup(block().getFeature(n_id).id())}       // H5Gunlink
-         +     deleteFeature(Feature)              <-b                     backend.deleteFeature(Feature.id())

x         +     retrieveData(ref_idx)               ->DataView
x         +     retrieveData(id)                    ->DataView
x         +     retrieveFeature(feat_idx)           ->DataView
x         +     retrieveFeature(id)                 ->DataView

        +Entity +NamedEntity +EntityWithMetaData +EntityWithSources
x (desc)|(+)|   id()                    ->s
x (desc)|(+)|   name()                  ->s
x (desc)|(+)|   type()                  ->s
x       |(+)|   type(s)                 <-v
x       |(+)|   definition(s)           <-v
x (desc)|(+)|   definition()            ->s
x       |(+)|   definition(n)           <-v
x       |(+)|   compare(NamedEntity)    ->d

x       |(+)|   metadata()              ->Section
-         +     metadata(Section)       <-v
x       |(+)|   metadata(n_id)          <-v
x       |(+)|   metadata(n)             <-v

x       |(+)|   sourceCount()           ->d
x       |(+)|   hasSource(n_id)         ->b
-         +     hasSource(Source)       ->b
x       |(+)|   getSource(n_id)         ->Source
x       |(+)|   getSource(idx)          ->Source
x         +     sources(filter)         -><Source>
x       |(+)|   sources(<Source>)       <-v
x       |(+)|   addSource(n_id)         <-v
-         +     addSource(Source)       <-v
x       |(+)|   removeSource(n_id)      <-b
-         +     removeSource(Source)    <-b


mx       MultiTag.h (IMultiTag) |MultiTagHDF5.h| [IBaseTag] /BaseTagHDF5.h/ {BaseTagHDF5.c && MultiTag.c}
        {units}                                         // DataSet -> LocID -> H5Object
        {positions}                                     // DataSet -> LocID -> H5Object
        {extents}                                       // DataSet -> LocID -> H5Object
        -{} features, references                        // H5Group. optGroup

x (desc)|(+)|   units()                                 -><s>                   {group().getData("units")}
x       |(+)|   units(<s>)                              <-v                     {group().setData("units")}                                  // H5Pcreate & H5Dcreate
x       |(+)|   units(n)                                <-v                     {group().removeData("units")}                               // H5Gunlink
x       |(+)|   positions()                             ->DataArray             {group().openGroup("positions")}
x       |(+)|   positions(n_id)                         <-v                     {group().createLink(block().getDataArray(n_id).group(), "positions")}   // H5Gunlink existing "positions", H5Lcreate_hard
-         +     positions(DataArray)                    <-v                     backend.positions(DataArray.id())
x       |(+)|   hasPositions()                          ->b                     {group().hasGroup("positions")}
x       |(+)|   extents(n_id)                           <-v                     {group().createLink(block().getDataArray(n_id).group(), "extents")}     // H5Gunlink existing "extents", H5Lcreate_hard
-         +     extents(DataArray)                      <-v                     backend.extents(DataArray.id())
x       |(+)|   extents()                               ->DataArray             {group().openGroup("extents")}
x       |(+)|   extents(n)                              <-v                     group().removeGroup("extents")                              // H5Gunlink

x       /[+]/   hasReference(n_id)                      ->b                     {references.hasGroup(block().getDataArray(n_id).id())}
-         +     hasReference(DataArray)                 ->b
x       /[+]/   referenceCount()                        ->d
x       /[+]/   getReference(n_id)                      ->DataArray             {references.openGroup(block().getDataArray(n_id).id())}
x       /[+]/   getReference(idx)                       ->DataArray
x       /[+]/   addReference(n_id)                      <-v                     {references.createLink(block().getDataArray(n_id).id())}    // H5Lcreate_hard
-         +     addReference(DataArray)                 <-v                     backend.addReference(DataArray.name())
x       /[+]/   removeReference(n_id)                   <-b                     {references.removeGroup(getReference(n_id).id())}           // H5Gunlink
-         +     removeReference(DataArray)              <-b                     backend.removeReference(DataArray.name())
x         +     references(filter)                      -><DataArray>
x         +     references()                            -><DataArray>
x       /[+]/   references(<DataArray>)                 <-v                     {for each DA references.createLink(block().getDataArray(DA.id()).id())} // H5Lcreate_hard
x         +     retrieveData(pos_idx, ref_idx)          ->DataView
-         +     retrieveData(pos_idx, n_id)             ->DataView

x       /[+]/   hasFeature(n_id)                        ->b
-         +     hasFeature(Feature)                     ->b                     backend.hasFeature(Feature.id())
x       /[+]/   featureCount()                          ->d
x       /[+]/   getFeature(n_id)                        ->Feature
x       /[+]/   getFeature(idx)                         ->Feature
x         +     features(filter)                        -><Feature>
-         +     createFeature(DataArray, LinkType)      <-Feature
x       /[+]/   createFeature(n_id, LinkType)           <-Feature               {features.openGroup(util::createId())}      // H5Pcreate & H5Gcreate2, n_id ... id/name of an existing DA
x       /[+]/   deleteFeature(n_id)                     <-b                     {features.removeGroup(block().getFeature(n_id).id())}       // H5Gunlink
-         +     deleteFeature(Feature)                  <-b                     backend.deleteFeature(Feature.id())

x         +     retrieveFeatureData(pos_idx, feat_idx)  ->DataView
-         +     retrieveFeatureData(pos_idx, n_id)      ->DataView

        +Entity +NamedEntity +EntityWithMetaData +EntityWithSources
x (desc)|(+)|   id()                    ->s
x (desc)|(+)|   name()                  ->s
x (desc)|(+)|   type()                  ->s
x       |(+)|   type(s)                 <-v
x       |(+)|   definition(s)           <-v
x (desc)|(+)|   definition()            ->s
x       |(+)|   definition(n)           <-v
x       |(+)|   compare(NamedEntity)    ->d

x       |(+)|   metadata()              ->Section
-         +     metadata(Section)       <-v
x       |(+)|   metadata(n_id)          <-v
x       |(+)|   metadata(n)             <-v

x       |(+)|   sourceCount()           ->d
x       |(+)|   hasSource(n_id)         ->b
-         +     hasSource(Source)       ->b
x       |(+)|   getSource(n_id)         ->Source
x       |(+)|   getSource(idx)          ->Source
x         +     sources(filter)         -><Source>
x       |(+)|   sources(<Source>)       <-v
x       |(+)|   addSource(n_id)         <-v
-         +     addSource(Source)       <-v
x       |(+)|   removeSource(n_id)      <-b
-         +     removeSource(Source)    <-b


mx       Feature.h (IFeature) |FeatureHDF5.h| {FeatureHDF5.c}
        {link_type}                 // H5G.a
        {data}                      // H5Group

x       |(+)|   linkType(LinkType)  <-v
x (desc)|(+)|   linkType()          ->LinkType                                                                                          // "tagged", "untagged", "indexed"
x       |(+)|   data(n_id)          <-v                         {group().createLink(block().getDataArray(n_id).group(), "data")}        // H5Lcreate_hard, removes existing "data" before creation
-         +     data(DataArray)     <-v                         backend.data(DataArray.id())
x       |(+)|   data()              ->DataArray                 {group().openGroup("data")}     // "data" means only one link per Feature

        +Entity
x (desc)|(+)|   id()                ->s

nix-mx has getLinkType ... since this already included in the description, could be removed.


mx       Section.h (ISection) |SectionHDF5.h| {SectionHDF5.c}
        {repository}                            // H5G.a
        {mapping}                               // H5G.a
        {link}                                  // H5Group

        -|| sections, properties                // optional H5Groups

x       |(+)|   repository(s)                   <-v
x (desc)|(+)|   repository()                    ->s             // URI
x       |(+)|   repository(n)                   <-v
x       |(+)|   link(n_id)                      <-v             {group().createLink(findSection(n_id).group(), "link")}             // H5Lcreate_hard
-         +     link(Section)                   <-v             backend.link(Section.id())
x       |(+)|   link()                          ->Section
x       |(+)|   link(n)                         <-v             {group().removeGroup("link")}                                       // H5Gunlink
x       |(+)|   mapping(s)                      <-v
x (desc)|(+)|   mapping()                       ->s
x       |(+)|   mapping(n)                      <-v
x       |(+)|   parent()                        ->Section
x       |(+)|   sectionCount()                  ->d
x       |(+)|   hasSection(n_id)                ->b
-         +     hasSection(Section)             ->b             backend.hasSection(Section.id())
x       |(+)|   getSection(n_id)                ->Section       {sections.findGroupByNameOrAttribute("entity_id", n_id)}
x       |(+)|   getSection(idx)                 ->Section
x         +     sections(filter)                -><Section>
x         +     findSections(filter, depth)     -><Section>
x         +     findRelated(filter)             -><Section>
x       |(+)|   createSection(name, type)       <-Section       {sections.openGroup(name)}                                          // H5Pcreate & H5Gcreate2, create with name, not id
x       |(+)|   deleteSection(n_id)             <-b             {sections.removeAllLinks(sections.getSection(n_id).name())}         // H5Ldelete, children removed by child.id()
-         +     deleteSection(Section)          <-b             backend.deleteSection(Section.id())
x       |(+)|   propertyCount()                 ->d
x       |(+)|   hasProperty(n_id)               ->b
-         +     hasProperty(Property)           ->b             backend.hasProperty(Property.id())
x       |(+)|   getProperty(n_id)               ->Property      {properties.findGroupByNameOrAttribute("entity_id", n_id)}
x       |(+)|   getProperty(idx)                ->Property
x         +     properties(filter)              -><Property>
x         +     inheritedProperties()           -><Property>
x       |(+)|   createProperty(name, type)      <-Property      {properties.createData(name)}                                       // H5Pcreate & H5Dcreate
-       |(+)|   createProperty(name, value)     <-Property
x       |(+)|   createProperty(name, <value>)   <-Property
x       |(+)|   deleteProperty(n_id)            <-b             {properties.removeData(properties.getProperty(n_id).name())}        // H5Gunlink
-         +     deleteProperty(Property)        <-b             backend.deleteProperty(Property.id())
-         +     referringDataArrays(Block)      -><DataArray>
x         +     referringDataArrays()           -><DataArray>
x         +     referringTags()                 -><Tag>
-         +     referringTags(Block)            -><Tag>
x         +     referringMultiTags()            -><MultiTag>
-         +     referringMultiTags(Block)       -><MultiTag>
x         +     referringSources()              -><Source>
-         +     referringSources(Block)         -><Source>
x         +     referringBlocks()               -><Block>
          -     findDownstream(filter)          -><Section>
          -     findUpstream(fiter)             -><Section>
          -     findSideways(filter)            -><Section>
          -     tree_depth()                    ->d
          +|()|	parentFile()

          +Entity +NamedEntity
x (desc)|(+)|   id()                    ->s
x (desc)|(+)|   name()                  ->s
x (desc)|(+)|   type()                  ->s
x       |(+)|   type(s)                 <-v
x       |(+)|   definition(s)           <-v
x (desc)|(+)|   definition()            ->s
x       |(+)|   definition(n)           <-v
x       |(+)|   compare(NamedEntity)    ->d


// Note Entity and NamedEntity methods on PropertyHDF5.h & PropertyHDF5.cpp are 
// re-implemented, because the Base of this Entity is H5Object and not H5Group!
// Can therefore be not inherited from EntityHDF5!
// why???

mx       Property.h (IProperty) |PropertyHDF5.h| {PropertyHDF5.c}
        {entity_id}                         // H5G.a
        {updated_at}                        // H5G.a
        {created_at}                        // H5G.a
        {name}                              // H5G.a
        {definition}                        // H5G.a
        {mapping}                           // H5G.a
        {unit}                              // H5G.a

x (desc)|(+)|   name()                      ->s             // explicitly reimplemented, see Note
x (desc)|(+)|   definition()                ->s
x       |(+)|   definition(s)               <-v
x       |(+)|   definition(n)               <-v
x       |(+)|   mapping(s)                  <-v
x (desc)|(+)|   mapping()                   ->s
x       |(+)|   mapping(n)                  <-v
x (desc)|(+)|   dataType()                  ->DataType
x       |(+)|   unit(s)                     <-v
x (desc)|(+)|   unit()                      ->s
x       |(+)|   unit(n)                     <-v
x       |(+)|   deleteValues()              <-v
x       |(+)|   valueCount()                ->d
x(works?|(+)|   values(<Value>)             <-v
x       |(+)|   values()                    -><Value>
x       |(+)|   values(n)                   <-v
x         +     compare(Property)           ->d
        +||     explicitly all Entity Methods               // explicitly reimplemented, see Note
        +||     fileTypeForValue(DataType)  ->DataType
        -||     dataSet()                   ->DataSet

        +Entity
x (desc)|(+)|   id()                    ->s


mx       Value.h

        uncertainty             d
        reference               s
        filename                s
        encoder                 s
        checksum                s

        data.set();.get()
-       type()                  ->DataType
-       supportsType(DataType)  ->b


mx       DataSet.h |H5DataSet.h|
          +     getData(T)
          +     getData(T, count, offset)
          +     getData(T, offset)
          +     setData(T, offset)
          +     getData(dtype, data, count, offset)
          +     setData(dtype, data, count, offset)

          +     dataExtent(d)       <-v
          +     dataExtent()        ->d
         |+|    dataType()          ->DataType
          ~     ioRead
          ~     ioWrite


mx      Dimensions.h (IDimensions) |DimensionHDF5.h| {DimensionHDF5.c}
        
        SampledDimension
        {label}                         // H5G.a
        {unit}                          // H5G.a
        {sampling_interval}             // H5G.a
        {offset}                        // H5G.a
        
-         +     index()                 ->d
x (desc) |+|    dimensionType()         ->DimensionType
x (desc)|(+)|   label()                 ->s
x       |(+)|   label(s)                <-v
x       |(+)|   label(n)                <-v
x (desc)|(+)|   unit()                  ->s
x       |(+)|   unit(s)                 <-v
x       |(+)|   unit(n)                 <-v
x (desc)|(+)|   samplingInterval()      ->d
x       |(+)|   samplingInterval(d)     <-v
x (desc)|(+)|   offset()                ->d
x       |(+)|   offset(d)               <-v
x       |(+)|   offset(n)               <-v
x         +     indexOf(d)              ->d
x         +     positionAt(d)           ->d
x         +     axis(count, start_idx)  -><d>


        SetDimension
        {labels}                        // H5G.a

-         +     index()                 ->d
x (desc) |+|    dimensionType()         ->DimensionType
x (desc)|(+)|   labels()                -><s>
x       |(+)|   labels(<s>)             <-v
x       |(+)|   labels(n)               <-v


        RangeDimension
        {label}                         // H5G.a
        {unit}                          // H5G.a
        {ticks}                         // DataSet -> LocID -> H5Object


x (desc)|(+)|   alias()                 ->b
-         +     index()                 ->d
x (desc) |+|    dimensionType()         ->DimensionType
x (desc)|(+)|   label()                 ->s
x       |(+)|   label(s)                <-v
x       |(+)|   label(n)                <-v
x (desc)|(+)|   unit()                  ->s
x       |(+)|   unit(s)                 <-v
x       |(+)|   unit(n)                 <-v
x (desc)|(+)|   ticks()                 -><d>
x       |(+)|   ticks(<d>)              <-v                 {group().setData("ticks")}          // H5Pcreate & H5Dcreate
x         +     tickAt(idx)             ->d
x         +     indexOf(position)       ->d
x         +     axis(count, start_idx)  -><d>


        Dimension
        {dimension_type}                // H5G.a

        ~|| group                       // H5Group
        ~|| dim_index                   // d

        |(+)|   index()                 ->d
          +     dimensionType()         ->DimensionType
          +     asSetDimension()        ->SetDimension
          +     asSampledDimension()    ->SampledDimension
          +     asRangeDimension()      ->RangeDimension
        ~||     setType()               <-v

