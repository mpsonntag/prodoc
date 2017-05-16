mx		Entity.h (IEntity) |EntityHDF5.h| {EntityHDF5.c}
        {entity_id}                     // H5G.a=H5Group.attribute
        {updated_at}                    // H5G.a
        {created_at}                    // H5G.a

		|(+)|	id()				    ->s             
		|(+)|	updatedAt()			    ->t=time
		|(+)|	createdAt()			    ->t
		|(+)|	setUpdatedAt()		    <-v=void
		|(+)|	forceUpdatedAt()	    <-v
		|(+)|	setCreatedAt()		    <-v
		|(+)|	forceCreatedAt(t)       <-v
		|(+)|	isValidEntity()		    ->b=bool          {group().referenceCount()>0}
		+||		group()                 ->H5Group
		~||		file()                  ->ptr


mx		NamedEntity.h (INamedEntity) |NamedEntityHDF5.h| {NamedEntityHDF5.c}
        {name}                          // H5G.a
        {type}                          // H5G.a
        {definition}                    // H5G.a

		|(+)|	type(s)					<-v
		|(+)|	type()					->s
		|(+)|	name()					->s
		|(+)|	definition(s)			<-v
		|(+)|	definition()			->s
		|(+)|	definition(n)			<-v // n=none, deleter
		|(+)|	compare(NamedEntity)	->d=int


mx		EntityWithMetaData.h (IEntityWithMetaData) |EntityWithMetaDataHDF5.h| {EntityWithMetaDataHDF5.c}
		|(+)|	metadata()			    ->Section           {group().openGroup("metadata")}
		  + 	metadata(Section)	    <-v                 backend.metadata(Section.id())
		|(+)|	metadata(id)		    <-v // id=s         {group().createLink(file().findSection(id).group(), "metadata")}     // H5Lcreate_hard
		|(+)|	metadata(n)			    <-v                 {group().removeGroup("metadata")}               // H5Gunlink


mx		EntityWithSources.h (IEntityWithSources) |EntityWithSourcesHDF5.h| {EntityWithSourcesHDF5.c}
        -||     sources                 // H5Group optGroup

		|(+)|	sourceCount()		    ->d // d=any int
		|(+)|	hasSource(n_id)		    ->b                 {sources.hasGroup(n_id)}
		  + 	hasSource(Source)	    ->b                 backend.hasSource(Source.id())
		|(+)|	getSource(n_id)		    ->Source            {sources.openGroup(block().findSources(n_id).id())}
		|(+)|	getSource(idx)		    ->Source //idx=d
		  + 	sources(filter)		    -><Source>
		|(+)|	sources(<Source>)	    <-v                 {for each Source call addSource(Source.id())}
		|(+)|	addSource(n_id)		    <-v                 {sources.createLink(block().findSources(n_id).group(), n_id)}       // H5Lcreate_hard
		  + 	addSource(Source)	    <-v                 backend.addSource(Source.id())
		|(+)|	removeSource(n_id)	    <-b                 {sources.removeGroup(n_id)}                                         // H5Gunlink
		  + 	removeSource(Source)    <-b                 backend.removeSource(Source.id())
		+||		block()                 ->ptr



// How does H5Group.openGroup actually create a new group with name and id? uses H5Pcreate & H5Gcreate2
// with the "name" it gets, but how is this also linked with an additionally created UUID?
// Why is this creation process different than createLink from EntityXXX (H5Lcreate_hard)

mx		File.h (IFile) |FileHDF5| {FileHDF5.c}
        {format}                            // H5G.a
        {version}                           // H5G.a
        {updated_at}                        // H5G.a
        {created_at}                        // H5G.a

        -||     root, metadata, data        // H5Groups "root" directory, containing one directory "data", one "metadata"

x		  +		open(name, mode, backend)	-> //name=s mode=FileMode, backend=...,HDF5
		|(+)|	flush()						->b // persist changes
		|(+)|	blockCount()				->d
x		|(+)|	hasBlock(n_id)				->b // n_id=s
		  +		hasBlock(Block)				->b
x		|(+)|	getBlock(n_id)				->Block             {data.findGroupByNameOrAttribute("entity_id", n_id)}
		|(+)|	getBlock(idx)				->Block
x		|(+)|	createBlock(name, type)		<-Block             {data.openGroup(name)}          // H5Pcreate & H5Gcreate2
x		|(+)|	deleteBlock(n_id)			<-b                 {data.removeAllLInks(id)}       // H5Ldelete
		  +		deleteBlock(Block)			<-b
		  +		blocks(filter)				-><Block>
x		  +		blocks()					-><Block>
x		|(+)|	hasSection(n_id)			->b
		  +		hasSection(Section)			->b
x		|(+)|	getSection(n_id)			->Section           {metadata.findGroupByNameOrAttribute("entity_id", n_id)}
		|(+)|	getSection(idx)				->Section
		|(+)|	sectionCount()				->d
		  +		sections(filter)			-><Section>
x		  +		sections()					-><Section>
		  +		findSections(filter, depth)	-><Section>	// depth=int
		  +		findSections(depth)			-><Section>
x		|(+)|	createSection(name, type)	<-Section           {metadata.openGroup(name)}      // H5Pcreate & H5Gcreate2
x		|(+)|	deleteSection(n_id)			<-b                 // {} also deletes all child sections
		  +		deleteSection(Section)		<-b
x		|(+)|	version()					-><d>
x		|(+)|	format()					->s
x		|(+)|	location()					->s // URI
x		|(+)|	createdAt()					->t
x		|(+)|	updatedAt()					->t
		|(+)|	setUpdatedAt()				<-v
		|(+)|	forceUpdatedAt()			<-v
		|(+)|	setCreatedAt()				<-v
		|(+)|	forceCreatedAt(t)			<-v
		|(+)|	close()						<-v
		|(+)|	isOpen()					->b
		|(+)|	fileMode()					->FileMode
		  +		validate()					->??
        -||     fileExists()                ->b
        -||     openRoot()                  <-v
        -||     checkHeader(FileMode)       ->b
        -||     createHeader()              <-v


mx		Feature.h (IFeature) |FeatureHDF5.h| {FeatureHDF5.c}
        {link_type}                 // H5G.a

		|(+)|	linkType(LinkType)	<-v
		|(+)|	linkType()			->LinkType                  {"tagged", "untagged", "indexed"}
		|(+)|	data(n_id)			<-v                         {group().createLink("data")}    // H5Lcreate_hard
		  +		data(DataArray)		<-v
		|(+)|	data()				->DataArray                 {group().openGroup("data")}

        +Entity


mx		Source.h (ISource) |SourceHDF5.h| {SourceHDF5.c}
        -||     sources                     // H5Group

		|(+)|	hasSource(n_id)				->b
		  +		hasSource(Source)			->b
		|(+)|	getSource(n_id)				->Source            {sources.findGroupByNameOrAttribute("entity_id", n_id)}
		|(+)|	getSource(idx)				->Source
		|(+)|	sourceCount()				->d
		  +		sources(filter)				-><Source>
		  +		findSources(filter, depth)	-><Source>
		|(+)|	createSource(name, type)	<-Source            {sources.openGroup(name)}          // H5Pcreate & H5Gcreate2
		|(+)|	deleteSource(n_id)			<-b                 {sources.removeAllLInks(id)}       // H5Ldelete
		  +		deleteSource(Source)		<-b
		  +		parentSource()				->Source

		  +		referringDataArray()		-><DataArray>
		  +		referringTags()				-><Tag>
		  +		referringMultiTags()		-><MultiTag>
		  +|()|	parentFile()				->File
		  +|()|	parentBlock()				->Block
		  
		  +Entity +NamedEntity +EntityWithMetaData


mx		Group.h (IGroup) |GroupHDF5.h| {GroupHDF5.c}
        -||     data_array, tag, multi_tag  // H5Group

		|(+)|	hasDataArray(n_id)			->b                 {data_array.hasGroup(id)}           // specifically gets id() for lookup
		  +		hasDataArray(DataArray)		->b
		|(+)|	dataArrayCount()			->d                 
		|(+)|	getDataArray(n_id)			->DataArray         {data_array.getGroup(id)}           // specifically gets id() for lookup
		|(+)|	getDataArray(idx)			->DataArray
		  +		addDataArray(DataArray)		<-v // v=void
		|(+)|	addDataArray(id)			<-v                 {data_array.createLink(id)}         // H5Lcreate_hard; specifically uses id(), not name to create link
		  +		removeDataArray(DataArray)	<-b
		|(+)|	removeDataArray(n_id)		<-b                 {data_array.removeGroup}            // H5Gunlink, specifically gets id() for unlink
		  +		dataArrays(filter)			-><DataArray>
		  +		dataArrays()				-><DataArray>
		|(+)|	dataArrays(<DataArray>)		<-v                 {for each i addDataArray above}

		|(+)|	hasTag(n_id)    			->b                 {tag.hasGroup(id)}                  // specifically gets id() for lookup
		  +		hasTag(Tag)					->b
		|(+)|	tagCount()					->d
		|(+)|	getTag(n_id)				->Tag               {tag.getGroup(id)}                  // specifically gets id() for lookup
		|(+)|	getTag(idx)					->Tag
		|(+)|	addTag(id)					<-v                 {tag.createLink(id)}                // H5Lcreate_hard; specifically uses id(), not name to create link
		  +		addTag(Tag)					<-v
		|(+)|	removeTag(n_id)				<-b                 {tag.removeGroup}                   // H5Gunlink, specifically gets id() for unlink
		  +		removeTag(Tag)				<-b
		  +		tags(filter)				-><Tag>
		  +		tags()						-><Tag>
		|(+)|	tags(<Tag>)					<-v                 {for each i addTag above}

		|(+)|	hasMultiTag(n_id)			->b                 {multi_tag.hasGroup(id)}            // specifically gets id() for lookup
		  +		hasMultiTag(MultiTag)		->b
		|(+)|	multiTagCount()				->d
		|(+)|	getMultiTag(n_id)			->MultiTag          {multi_tag.getGroup(id)}            // specifically gets id() for lookup
		|(+)|	getMultiTag(idx)			->MultiTag
		|(+)|	addMultiTag(n_id)			<-v                 {multi_tag.createLink(id)}          // specifically uses id(), not name to create link
		  +		addMultiTag(MultiTag)		<-v
		|(+)|	removeMultiTag(n_id)		<-b                 {multi_tag.removeGroup}             // H5Gunlink, specifically gets id() for unlink
		  +		removeMultiTag(MultiTag)	<-b
		  +		multiTags(filter)			-><MultiTag>
		  +		multiTags()					-><MultiTag>
		|(+)|	multiTags(<MultiTag>)		<-v                 {for each i addMultiTag above}
		
		+Entity +NamedEntity +EntityWithMetaData +EntityWithSources


mx		Block.h (IBlock) |BlockHDF5.h| {BlockHDF5.c}
        ?{}     data_arrays, tags, multi_tags, sources, groups      // H5Groups, optional

		|(+)|	hasSource(n_id)				->b
		  +		hasSource(Source)			->b
		|(+)|	getSource(n_id)				->Source                {sources.findGroupByNameOrAttribute("entity_id", n_id)}
		|(+)|	getSource(idx)				->Source
		|(+)|	sourceCount()				->d
		  +		sources(filter)				-><Source>
		  +		findSources(filter, depth)	-><Source>
		|(+)|	createSource(name, type)	<-Source                {sources.openGroup(name)}               // H5Pcreate & H5Gcreate2
		|(+)|	deleteSource(n_id)			<-b                     {sources.removeAllLInks(name)}          // H5Ldelete, remove specifically by name()
		  +		deleteSource(Source)		<-b

		|(+)|	hasDataArray(n_id)			->b
		  +		hasDataArray(DataArray)		->b
		|(+)|	getDataArray(n_id)			->DataArray             {data_arrays.findGroupByNameOrAttribute("entity_id", n_id)}
		|(+)|	getDataArray(idx)			->DataArray
		  +		dataArrays(filter)			-><DataArray>
		|(+)|	dataArrayCount()			->d
		|(+)|	createDataArray(name, type, DataType, size)		<-DataArray     {data_arrays.openGroup(name) + group().createData("data")}  // H5Pcreate & H5Gcreate2 + H5Pcreate & H5Dcreate
		  +		createDataArray(name, type, <data>, DataType)	<-DataArray
		|(+)|	deleteDataArray(n_id)		<-b                     {data_arrays.removeAllLInks(name)}      // H5Ldelete, remove specifically by name()
		  +		deleteDataArray(DataArray)	<-b

		|(+)|	hasTag(id)					->b
		  +		hasTag(Tag)					->b
		|(+)|	getTag(id)					->Tag                   {tags.findGroupByNameOrAttribute("entity_id", n_id)}
		|(+)|	getTag(idx)					->Tag
		  +		tags(filter)				-><Tag>
		|(+)|	tagCount()					->d
		|(+)|	createTag(name, type, position)	<-Tag // position=<double>  {tags.openGroup(name)}          // H5Pcreate & H5Gcreate2
		|(+)|	deleteTag(n_id)				<-b                     {tags.removeAllLInks(name)}             // H5Ldelete, remove specifically by name()
		  +		deleteTag(idx)				<-b

		|(+)|	hasMultiTag(id)				->b
		  +		hasMultiTag(Tag)			->b
		|(+)|	getMultiTag(n_id)			->MultiTag              {multi_tags.findGroupByNameOrAttribute("entity_id", n_id)}
		|(+)|	getMultiTag(idx)			->MultiTag
		  +		multiTags(filter)			-><MultiTag>
		|(+)|	multiTagCount()				->d
		|(+)|	createMultiTag(name, type, positions)	<-MultiTag // positions=DataArray   {multi_tags.openGroup(name)}          // H5Pcreate & H5Gcreate2
		|(+)|	deleteMultiTag(n_id			<-b                     {multi_tags.removeAllLInks(name)}       // H5Ldelete, remove specifically by name()
		  +		deleteMultiTag(idx)			<-b

		|(+)|	hasGroup(n_id)				->b
		  +		hasGroup(Group)				->b
		|(+)|	getGroup(n_id)				->Group                 {groups.findGroupByNameOrAttribute("entity_id", n_id)}
		|(+)|	getGroup(idx)				->Group
		  +		groups(filter)				-><Group>
		|(+)|	groupCount()				->d
		|(+)|	createGroup(name, type)		<-Group                 {groups.openGroup(name)}                // H5Pcreate & H5Gcreate2
		|(+)|	deleteGroup(n_id)			<-b                     {groups.removeAllLInks(name)}           // H5Ldelete, remove specifically by name()
		  +		deleteGroup(Group)			<-b
		+||		block()						->ptr
		
		+Entity +NamedEntity +EntityWithMetaData


mx		DataArray.h (IDataArray) |DataArrayHDF5.h| {DataArrayHDF5.cpp}
        {label}                                     // H5G.a
        {unit}                                      // H5G.a
        {expansion_origin}                          // H5G.a
        {polynom_coefficients}                      // DataSet -> LocID -> H5Object
        ?{} dimension                               // H5Group. optGroup

		|(+)|	label()						        ->s
		|(+)|	label(s)					        <-v
		|(+)|	label(n)					        <-v
		|(+)|	unit()						        ->s
		|(+)|	unit(s)						        <-v
		|(+)|	unit(n)						        <-n
		|(+)|	expansionOrigin()			        ->d
		|(+)|	expansionOrigin(d)			        <-v
		|(+)|	expansionOrigin(n)			        <-v
		|(+)|	polynomCoefficients(<d>)	        <-v                     {group().createData("polynom_coefficients")}    // H5Pcreate & H5Dcreate
		|(+)|	polynomCoefficients()		        -><d>
		|(+)|	polynomCoefficients(n)		        <-v                     {group().removeData("polynom_coefficients")}    // H5Gunlink
		  +		dimensions(filter)			        -><Dimension>
		  +		dimensions()				        -><Dimension>
		|(+)|	dimensionCount()			        ->d
		|(+)|	getDimension(idx)			        ->Dimension
		  +		appendSetDimension()		        <-SetDimension
		  +		appendRangeDimension(<d>)	        <-RangeDimension
		  +		appendAliasRangeDimension()	        <-RangeDimension
		  +		appendSampledDimension(d)	        <-SampledDimension
		|(+)|	createSetDimension(idx)		        <-SetDimension          {createDimensionGroup(idx)}                         // H5Pcreate & H5Gcreate2
		|(+)|	createRangeDimension(idx, <d>)	    <-RangeDimension        {createDimensionGroup(idx)}                         // H5Pcreate & H5Gcreate2
		|(+)|	createAliasRangeDimension()	        <-RangeDimension        {createDimensionGroup(1)}                           // H5Pcreate & H5Gcreate2
		|(+)|	createSampledDimension(idx, d)	    <-SampledDimension      {createDimensionGroup(idx)}                         // H5Pcreate & H5Gcreate2
		|(+)|	deleteDimensions()			        <-b                     {for every dimension: dimensions.removeGroup()}     // H5Gunlink
		  +		getDataDirect(DataType, data, count, offset)	<-v         // {read(dtype, data, count, offset)}
		  +		setDataDirect(DataType, data, count, offset)	<-v         // {write(dtype, data, count, offset)}
		|(+)|	dataExtent()				        ->d                     {group().openData("data").size()}
		|(+)|	dataExtent(d)				        <-v                     {group().openData("data").setExtent}                // H5Dset_extent
		|(+)|	dataType()					        ->DataType              {group().openData("data").dataType()}               // H5Dget_type
		  +		appendData(DataType, data, count, axis)
		  ~		ioRead
		  ~		ioWrite
		+|()|	createData(DataType, size)	        <-v                     {group().createData("data")}                        // H5Pcreate & H5Dcreate
		+|()|	hasData()					        ->b                     {group().hasData("data")}
		+|()|	write(dtype, data, count, offset)   <-v
		+|()|	read(dtype, buffer, count, offset)  <-v                     // read data into buffer
        -||     createDimensionGroup(idx)           <-H5Group               {dimension.openGroup(idx)}                          // H5Pcreate & H5Gcreate2
		
		+Entity +NamedEntity +EntityWithMetaData +EntityWithSources


mx		Tag.h (ITag) |TagHDF5.h| [IBaseTag] /BaseTagHDF5.h/ {BaseTagHDF5.c && TagHDF5.c}
        {units}                                     // DataSet -> LocID -> H5Object
        {position}                                  // DataSet -> LocID -> H5Object
        {extent}                                    // DataSet -> LocID -> H5Object
        -{} features, references                    // H5Group. optGroup

		|(+)|	units()						        -><s>                   {group().getData("units")}
		|(+)|	units(<s>)					        <-v                     {group().setData("units")}                  // H5Pcreate & H5Dcreate
		|(+)|	units(n)					        <-v                     {group().removeData("units")}               // H5Gunlink
		|(+)|	position()					        -><d>                   {group().getData("position")}
		|(+)|	position(<d>)				        <-v                     {group().setData("position")}               // H5Pcreate & H5Dcreate
		|(+)|	extent(<d>)					        <-v                     {group().getData("extent")}
		|(+)|	extent()					        -><d>                   {group().setData("extent")}                 // H5Pcreate & H5Dcreate
		|(+)|	extent(n)					        <-v                     {group().removeData("extent")}              // H5Gunlink

		/[+]/	hasReference(n_id)			        ->b                     {references.hasGroup(block().getDataArray(n_id).id())}
		  +		hasReference(DataArray)		        ->b                     backend.hasReference(DataArray.id())
		/[+]/	referenceCount()			        ->d
		/[+]/	getReference(n_id)			        ->DataArray             {references.openGroup(block().getDataArray(n_id).id())}
		/[+]/	getReference(idx)			        ->DataArray
		/[+]/	addReference(id)			        <-v                     {references.createLink(block().getDataArray(n_id).id())}    // H5Lcreate_hard
		  +		addReference(DataArray)		        <-v                     backend.addReference(DataArray.name())
		/[+]/	removeReference(n_id)		        <-b                     {references.removeGroup(getReference(n_id).id())}           // H5Gunlink
		  +		removeReference(DataArray)	        <-b                     backend.addReference(DataArray.name())
		  +		references(filter)			        -><DataArray>
		  +		references()				        -><DataArray>
		/[+]/	references(<DataArray>)		        <-v                     {for each DA references.createLink(block().getDataArray(DA.id()).id())} // H5Lcreate_hard

		/[+]/	hasFeature(id)				        ->b
		  +		hasFeature(Feature)			        ->b                     backend.hasFeature(Feature.id())
		/[+]/	featureCount()				        ->d
		/[+]/	getFeature(id)				        ->Feature
		/[+]/	getFeature(idx)				        ->Feature
		  +		features(filter)			        -><Feature>
		  +		createFeature(DataArray, LinkType)	<-Feature               backend.createFeature(Feature.id())
		/[+]/	createFeature(n_id, LinkType)		<-Feature               {features.openGroup(util::createId())}      // H5Pcreate & H5Gcreate2, n_id ... id/name of an existing DA
		/[+]/	deleteFeature(n_id)			        <-b                     {features.removeGroup(block().getFeature(n_id).id())}       // H5Gunlink
		  +		deleteFeature(Feature)		        <-b                     backend.deleteFeature(Feature.id())

		  +		retrieveData(ref_idx)		        ->DataView
		  +		retrieveData(id)			        ->DataView
		  +		retrieveFeature(feat_idx)	        ->DataView
		  +		retrieveFeature(id)			        ->DataView
		
		+Entity +NamedEntity +EntityWithMetaData +EntityWithSources


mx		MultiTag.h (IMultiTag) |MultiTagHDF5.h| [IBaseTag] /BaseTagHDF5.h/ {BaseTagHDF5.c && MultiTag.c}
        {units}                                         // DataSet -> LocID -> H5Object
        {position}                                      // DataSet -> LocID -> H5Object
        {extent}                                        // DataSet -> LocID -> H5Object
        -{} features, references                        // H5Group. optGroup

		|(+)|	units()						            -><s>                   {group().getData("units")}
		|(+)|	units(<s>)					            <-v                     {group().setData("units")}                  // H5Pcreate & H5Dcreate
		|(+)|	units(n)					            <-v                     {group().removeData("units")}               // H5Gunlink
		|(+)|	positions()					            ->DataArray             {group().openGroup("positions")}
		|(+)|	positions(n_id)     		            <-v                     {group().createLink(block().getDataArray(n_id).group(), "positions")}   // H5Gunlink existing "positions", H5Lcreate_hard
		  + 	positions(DataArray)		            <-v                     backend.positions(DataArray.id())
		|(+)|	hasPositions()				            ->b                     {group().hasGroup("positions")}
		|(+)|	extents(n_id)   			            <-v                     {group().createLink(block().getDataArray(n_id).group(), "extents")}     // H5Gunlink existing "extents", H5Lcreate_hard
		  + 	extents(DataArray)			            <-v                     backend.extents(DataArray.id())
		|(+)|	extents()					            ->DataArray             {group().openGroup("extents")}
		|(+)|	extents(n)         			            <-v                     group().removeGroup("extents")                              // H5Gunlink

		/[+]/	hasReference(n_id)			            ->b                     {references.hasGroup(block().getDataArray(n_id).id())}
		  +		hasReference(DataArray)		            ->b
		/[+]/	referenceCount()			            ->d
		/[+]/	getReference(n_id)			            ->DataArray             {references.openGroup(block().getDataArray(n_id).id())}
		/[+]/	getReference(idx)			            ->DataArray
		/[+]/	addReference(n_id)			            <-v                     {references.createLink(block().getDataArray(n_id).id())}    // H5Lcreate_hard
		  +		addReference(DataArray)		            <-v                     backend.addReference(DataArray.name())
		/[+]/	removeReference(n_id)		            <-b                     {references.removeGroup(getReference(n_id).id())}           // H5Gunlink
		  +		removeReference(DataArray)	            <-b                     backend.removeReference(DataArray.name())
		  +		references(filter)			            -><DataArray>
		  +		references()				            -><DataArray>
		/[+]/	references(<DataArray>)		            <-v                     {for each DA references.createLink(block().getDataArray(DA.id()).id())} // H5Lcreate_hard
		  +		retrieveData(pos_idx, ref_idx)          ->DataView
		  +		retrieveData(pos_idx, n_id)             ->DataView

		/[+]/	hasFeature(n_id)			            ->b
		  +		hasFeature(Feature)			            ->b                     backend.hasFeature(Feature.id())
		/[+]/	featureCount()				            ->d
		/[+]/	getFeature(n_id)			            ->Feature
		/[+]/	getFeature(idx)				            ->Feature
		  +		features(filter)			            -><Feature>
		  +		createFeature(DataArray, LinkType)	    <-Feature
		/[+]/	createFeature(n_id, LinkType)		    <-Feature               {features.openGroup(util::createId())}      // H5Pcreate & H5Gcreate2, n_id ... id/name of an existing DA
		/[+]/	deleteFeature(n_id)			            <-b                     {features.removeGroup(block().getFeature(n_id).id())}       // H5Gunlink
		  +		deleteFeature(Feature)		            <-b                     backend.deleteFeature(Feature.id())

		  +		retrieveFeatureData(pos_idx, feat_idx)  ->DataView
		  +		retrieveFeatureData(pos_idx, n_id)      ->DataView
		
		+Entity +NamedEntity +EntityWithMetaData +EntityWithSources


mx		Section.h (ISection) |SectionHDF5.h| {SectionHDF5.c}
        {repository}                            // H5G.a
        {link}                                  // H5G.a
        {mapping}                               // H5G.a

        -|| sections, properties                // optional H5Groups
        
		|(+)|	repository(s)		            <-v
		|(+)|	repository()		            ->s // URI
		|(+)|	repository(n)		            <-v
		|(+)|	link(id)			            <-v
		  + 	link(Section)		            <-v
		|(+)|	link()				            ->Section
		|(+)|	link(n)				            <-v
		|(+)|	mapping(s)			            <-v
		|(+)|	mapping()			            ->s
		|(+)|	mapping(n)			            <-v
		|(+)|	parent()			            ->Section
		|(+)|	sectionCount()		            ->d
		|(+)|	hasSection(n_id)	            ->b
		  +		hasSection(Section)	            ->b
		|(+)|	getSection(n_id)	            ->Section       {sections.findGroupByNameOrAttribute("entity_id", n_id)}
		|(+)|	getSection(idx)		            ->Section
		  +		sections(filter)	            -><Section>
		  +		findSections(filter, depth)	    -><Section>
		  +		findRelated(filter)	            -><Section>
		|(+)|	createSection(name, type)	    <-Section       {sections.openGroup(name)}          // H5Pcreate & H5Gcreate2
		|(+)|	deleteSection(n_id)			    <-b             {sections.removeAllLInks(n_id)}     // H5Ldelete
		  +		deleteSection(Section)		    <-b
		|(+)|	propertyCount()				    ->d
		|(+)|	hasProperty(n_id)			    ->b
		  +		hasProperty(Property)		    ->b
		|(+)|	getProperty(n_id)			    ->Property      {properties.findGroupByNameOrAttribute("entity_id", n_id)}
		|(+)|	getProperty(idx)			    ->Property
		  + 	properties(filter)			    -><Property>
		  +		inheritedProperties()		    -><Property>
		|(+)|	createProperty(name, type)	    <-Property      {properties.createData(name)}       // H5Pcreate & H5Dcreate
		|(+)|	createProperty(name, value)	    <-Property
		|(+)|	createProperty(name, <value>)	<-Property
		|(+)|	deleteProperty(n_id)			<-b             {properties.removeData(n_id)}       // H5Gunlink, removes by name, not ID
		  +		deleteProperty(Property)		<-b
		  +		referringDataArrays(Block)		-><DataArray>
		  +		referringDataArrays()			-><DataArray>
		  +		referringTags()					-><Tag>
		  +		referringTags(Block)			-><Tag>
		  +		referringMultiTags()			-><MultiTag>
		  +		referringMultiTags(Block)		-><MultiTag>
		  +		referringSources()				-><Source>
		  +		referringSources(Block)			-><Source>
		  +		referringBlocks()				-><Block>
		  -		findDownstream(filter)			-><Section>
		  -		findUpstream(fiter)				-><Section>
		  -		findSideways(filter)			-><Section>
		  -		tree_depth()					->d
		  +|()|	parentFile()
		  
		  +Entity +NamedEntity


// Note Entity and NamedEntity methods on PropertyHDF5.h & PropertyHDF5.cpp are 
// re-implemented, because the Base of this Entity is H5Object and not H5Group!
// Can therefore be not inherited from EntityHDF5!
// why???

mx		Property.h (IProperty) |PropertyHDF5.h| {PropertyHDF5.c}
        {entity_id}                         // H5G.a
        {updated_at}                        // H5G.a
        {created_at}                        // H5G.a
        {name}                              // H5G.a
        {definition}                        // H5G.a
        {mapping}                           // H5G.a
        {unit}                              // H5G.a

		|(+)|	name()			            ->s             // explicitly reimplemented, see Note
		|(+)|	definition()	            ->s
		|(+)|	definition(s)	            <-v
		|(+)|	definition(n)	            <-v
		|(+)|	mapping(s)		            <-v
		|(+)|	mapping()		            ->s
		|(+)|	mapping(n)		            <-v
		|(+)|	dataType()		            ->DataType
		|(+)|	unit(s)			            <-v
		|(+)|	unit()			            ->s
		|(+)|	unit(n)			            <-v
		|(+)|	deleteValues()	            <-v
		|(+)|	valueCount()	            ->d
		|(+)|	values(<Value>)	            <-v
		|(+)|	values()		            -><Value>
		|(+)|	values(n)		            <-v
		  +		compare(Value)	            ->b
		+||		explicitly all Entity Methods               // explicitly reimplemented, see Note
		+||		fileTypeForValue(DataType)  ->DataType
		-||		dataSet()                   ->DataSet

        +Entity


mx		Value.h

		uncertainty				d
		reference				s
		filename				s
		encoder					s
		checksum				s

		data.set();.get()
		type()					->DataType
		supportsType(DataType)	->b


mx		DataSet.h |H5DataSet.h|
		  +		getData(T)
		  +		getData(T, count, offset)
		  +		getData(T, offset)
		  +		setData(T, offset)
		  +		getData(dtype, data, count, offset)
		  +		setData(dtype, data, count, offset)
		  
		  +		dataExtent(d)		<-v
		  +		dataExtent()		->d
		 |+|	dataType()			->DataType
		  ~		ioRead
		  ~		ioWrite


