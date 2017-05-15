mx		Entity.h (IEntity) |EntityHDF5.h| {EntityHDF5.cpp}

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
		|(+)|	isValidEntity()		    ->b=bool    // {group().referenceCount()>0}
		+||		group()                 ->H5Group
		~||		file()                  ->ptr


mx		NamedEntity.h (INamedEntity) |NamedEntityHDF5|
		|(+)|	type(s)					<-v
		|(+)|	type()					->s
		|(+)|	name()					->s
		|(+)|	definition(s)			<-v
		|(+)|	definition()			->s
		|(+)|	definition(n)			<-v // n=none, deleter
		|(+)|	compare(NamedEntity)	->d=int


mx		EntityWithMetaData.h (IEntityWithMetaData) |EntityWithMetaDataHDF5|
		|(+)|	metadata()			    ->Section
		|(+)|	metadata(Section)	    <-v
		|(+)|	metadata(id)		    <-v // id=s
		|(+)|	metadata(n)			    <-v


mx		EntityWithSources.h (IEntityWithSources) |EntityWithSourcesHDF5|
		|(+)|	sourceCount()		    ->d // d=any int
		|(+)|	hasSource(id)		    ->b
		  + 	hasSource(Source)	    ->b
		|(+)|	getSource(id)		    ->Source
		|(+)|	getSource(idx)		    ->Source //idx=d
		|(+)|	sources(filter)		    -><Source>
		|(+)|	sources(<Source>)	    <-v
		|(+)|	addSource(id)		    <-v
		  + 	addSource(Source)	    <-v
		|(+)|	removeSource(id)	    <-b
		  + 	removeSource(Source)    <-b
		+||		block()                 ->ptr


mx		File.h (IFile) |FileHDF5|
x		  +		open(name, mode, backend)	-> //name=s mode=FileMode, backend=...,HDF5
		|(+)|	flush()						-> // persist changes
		|(+)|	blockCount()				->d
x		|(+)|	hasBlock(n_id)				->b // n_id=s
		  +		hasBlock(Block)				->b
x		|(+)|	getBlock(n_id)				->Block
		|(+)|	getBlock(idx)				->Block
x		|(+)|	createBlock(name, type)		<-
x		|(+)|	deleteBlock(n_id)			<-
		  +		deleteBlock(Block)			<-
		  +		blocks(filter)				-><Block>
x		  +		blocks()					-><Block>
x		|(+)|	hasSection(n_id)			->b
		  +		hasSection(Section)			->b
x		|(+)|	getSection(n_id)			->Section
		|(+)|	getSection(idx)				->Section
		|(+)|	sectionCount()				->d
		  +		sections(filter)			-><Section>
x		  +		sections()					-><Section>
		  +		findSections(filter, depth)	-><Section>	// depth=int
		  +		findSections(depth)			-><Section>
x		|(+)|	createSection(name, type)	<-
x		|(+)|	deleteSection(n_id)			<-
		  +		deleteSection(Section)		<-
x		|(+)|	version()					->s
x		|(+)|	format()					->s
x		|(+)|	location()					->s // URI
x		|(+)|	createdAt()					->s
x		|(+)|	updatedAt()					->s
		|(+)|	setUpdatedAt()				<-
		|(+)|	forceUpdatedAt()			<-
		|(+)|	setCreatedAt()				<-
		|(+)|	forceCreatedAt()			<-
		|(+)|	close()						<-
		|(+)|	isOpen()					->b
		|(+)|	fileMode()					->FileMode
		  +		validate()					->??

mx		Feature.h (IFeature) |FeatureHDF5|
		|(+)|	linkType(LinkType)	<-
		|(+)|	linkType()			->LinkType
		|(+)|	data(n_id)			<-
		  +		data(DataArray)		<-
		|(+)|	data()				->DataArray
		
mx		Property.h (IProperty) |PropertyHDF5|
		|(+)|	name()			->s
		|(+)|	definition()	->s
		|(+)|	definition(s)	<-
		|(+)|	definition(n)	<-
		|(+)|	mapping(s)		<-
		|(+)|	mapping()		->s
		|(+)|	mapping(n)		<-
		|(+)|	dataType()		->DataType
		|(+)|	unit(s)			<-
		|(+)|	unit()			->s
		|(+)|	unit(n)			<-
		|(+)|	deleteValues()	<-
		|(+)|	valueCount()	->d
		|(+)|	values(<Value>)	<-
		|(+)|	values()		-><Value>
		|(+)|	values(n)		<-
		  +		compare(Value)	->b
		+||		explicitly all Entity Methods (weird)
		+||		fileTypeForValue()
		+||		dataSet()
		
mx		Section.h (ISection) |SectionHDF5|
		|(+)|	repository(s)		<-
		|(+)|	repository()		->s // URI
		|(+)|	repository(n)		<-
		|(+)|	link(id)			<-
		|(+)|	link(Section)		<-
		|(+)|	link()				->Section
		|(+)|	link(n)				<-
		|(+)|	mapping(s)			<-
		|(+)|	mapping()			->s
		|(+)|	mapping(n)			<-
		|(+)|	parent()			->Section
		|(+)|	sectionCount()		->d
		|(+)|	hasSection(n_id)	->b
		  +		hasSection(Section)	->b
		|(+)|	getSection(n_id)	->Section
		|(+)|	getSection(idx)		->Section
		  +		sections(filter)	-><Section>
		  +		findSections(filter, depth)	-><Section>
		  +		findRelated(filter)	-><Section>
		|(+)|	createSection(name, type)	<-
		|(+)|	deleteSection(n_id)			<-
		  +		deleteSection(Section)		<-
		|(+)|	propertyCount()				->d
		|(+)|	hasProperty(n_id)			->b
		  +		hasProperty(Property)		->b
		|(+)|	getProperty(n_id)			->Property
		|(+)|	getProperty(idx)			->Property
		  + 	properties(filter)			-><Property>
		  +		inheritedProperties()		-><Property>
		|(+)|	createProperty(name, type)	<-
		|(+)|	createProperty(name, value)	<-
		|(+)|	createProperty(name, <value>)	<-
		|(+)|	deleteProperty(n_id)			<-
		  +		deleteProperty(Property)		<-
		  +		referringDataArrays(Block)		-><DataArray>
		  +		referringDataArrays()			-><DataArrays>
		  +		referringTags()					-><Tags>
		  +		referringTags(Block)			-><Tags>
		  +		referringMultiTags()			-><MultiTags>
		  +		referringMultiTags(Block)		-><MultiTag>
		  +		referringSources()				-><Source>
		  +		referringSources(Block)			-><Source>
		  +		referringBlocks()				-><Block>
		  -		findDownstream(filter)			-><Section>
		  -		findUpstream(fiter)				-><Section>
		  -		findSideways(filter)			-><Section>
		  -		tree_depth()					->d
		  +|()|	parentFile()
		  
		  + Entity + NamedEntity
	
mx		Source.h (ISource) |SourceHDF5|
		|(+)|	hasSource(n_id)				->b
		  +		hasSource(Source)			->b
		|(+)|	getSource(n_id)				->Source
		|(+)|	getSource(idx)				->Source
		|(+)|	sourceCount()				->d
		  +		sources(filter)				-><Source>
		  +		findSources(filter, depth)	-><Source>
		|(+)|	createSource(name, type)	<-
		|(+)|	deleteSource(n_id)			<-
		  +		deleteSource(Source)		<-
		  +		parentSource()				->Source
		  +		referringDataArray()		-><DataArray>
		  +		referringTags()				-><Tag>
		  +		referringMultiTags()		-><MultiTag>
		  +|()|	parentFile()				->File
		  +|()|	parentBlock()				->Block
		  
		  +Entity +NamedEntity +EntityWithMetaData
		  
mx		Group.h (IGroup) |GroupHDF5|
		|(+)|	hasDataArray(id)			->b
		  +		hasDataArray(DataArray)		->b
		|(+)|	dataArrayCount()			->d
		|(+)|	getDataArray(id)			->DataArray
		|(+)|	getDataArray(idx)			->DataArray
		  +		addDataArray(DataArray)		<-v // v=void
		|(+)|	addDataArray(id)			<-
		  +		removeDataArray(DataArray)	<-b
		|(+)|	removeDataArray(id)			<-b
		  +		dataArrays(filter)			-><DataArray>
		  +		dataArrays()				-><DataArray>
		|(+)|	dataArrays(<DataArray>)		<-v
		|(+)|	hasTag(id)					->b
		  +		hasTag(Tag)					->b
		|(+)|	tagCount()					->d
		|(+)|	getTag(id)					->Tag
		|(+)|	getTag(idx)					->Tag
		|(+)|	addTag(id)					<-v
		  +		addTag(Tag)					<-v
		|(+)|	removeTag(id)				<-b
		  +		removeTag(Tag)				<-b
		  +		tags(filter)				-><Tag>
		  +		tags()						-><Tag>
		|(+)|	tags(<Tag>)					<-v
		|(+)|	hasMultiTag(id)				->b
		  +		hasMultiTag(MultiTag)		->b
		|(+)|	multiTagCount()				->d
		|(+)|	getMultiTag(id)				->MultiTag
		|(+)|	getMultiTag(idx)			->MultiTag
		|(+)|	addMultiTag(id)				<-v
		  +		addMultiTag(MultiTag)		<-v
		|(+)|	removeMultiTag(id)			<-b
		  +		removeMultiTag(MultiTag)	<-b
		  +		multiTags(filter)			-><MultiTag>
		  +		multiTags()					-><MultiTag>
		|(+)|	multiTags(<MultiTag>)		<-v
		
		+Entity +NamedEntity +EntityWithMetaData +EntityWithSources
		
mx		Bock.h (IBlock) |BlockHDF5|
		|(+)|	hasSource(n_id)				->b
		  +		hasSource(Source)			->b
		|(+)|	getSource(n_id)				->Source
		|(+)|	getSource(idx)				->Source
		|(+)|	sourceCount()				->d
		  +		sources(filter)				-><Source>
		  +		findSources(filter, depth)	-><Source>
		|(+)|	createSource(name, type)	<-
		|(+)|	deleteSource(n_id)			<-
		  +		deleteSource(Source)		<-
		|(+)|	hasDataArray(n_id)			->b
		  +		hasDataArray(DataArray)		->b
		|(+)|	getDataArray(n_id)			->DataArray
		|(+)|	getDataArray(idx)			->DataArray
		  +		dataArrays(filter)			-><DataArray>
		|(+)|	dataArrayCount()			->d
		|(+)|	createDataArray(name, type, DataType, size)		<-
		  +		createDataArray(name, type, <data>, DataType)	<-
		|(+)|	deleteDataArray(n_id)		<-
		  +		deleteDataArray(DataArray)	<-
		|(+)|	hasTag(id)					->b
		  +		hasTag(Tag)					->b
		|(+)|	getTag(id)					->Tag
		|(+)|	getTag(idx)					->Tag
		  +		tags(filter)				-><Tag>
		|(+)|	tagCount()					->d
		|(+)|	createTag(name, type, position)	<-Tag // position=int
		|(+)|	deleteTag(n_id)				<-
		  +		deleteTag(idx)				<-
		|(+)|	hasMultiTag(id)				->b
		  +		hasMultiTag(Tag)			->b
		|(+)|	getMultiTag(id)				->MultiTag
		|(+)|	getMultiTag(idx)			->MultiTag
		  +		multiTags(filter)			-><MultiTag>
		|(+)|	multiTagCount()				->d
		|(+)|	createMultiTag(name, type, position)	<-MultiTag // position=int (probably positionS)
		|(+)|	deleteMultiTag(n_id			<-
		  +		deleteMultiTag(idx)			<-
		|(+)|	hasGroup(n_id)				->b
		  +		hasGroup(Group)				->b
		|(+)|	getGroup(n_id)				->Group
		|(+)|	getGroup(idx)				->Group
		  +		groups(filter)				-><Group>
		|(+)|	groupCount()				->d
		|(+)|	createGroup(name, type)		<-Group
		|(+)|	deleteGroup(n_id)			<-b
		  +		deleteGroup(Group)			<-b
		+||		block()						->ptr
		
		+Entity +NamedEntity +EntityWithMetaData
		
mx		DataArray.h (IDataArray) |DataArrayHDF5|
		|(+)|	label()						->s
		|(+)|	label(s)					<-v
		|(+)|	label(n)					<-v
		|(+)|	unit()						->s
		|(+)|	unit(s)						<-v
		|(+)|	unit(n)						<-n
		|(+)|	expansionOrigin()			->d
		|(+)|	expansionOrigin(d)			<-v
		|(+)|	expansionOrigin(n)			<-v
		|(+)|	polynomCoefficients(<d>)	<-v
		|(+)|	polynomCoefficients()		-><d>
		|(+)|	polynomCoefficients(n)		<-v
		  +		dimensions(filter)			-><Dimension>
		  +		dimensions()				-><Dimension>
		|(+)|	dimensionCount()			->d
		|(+)|	getDimension(idx)			->Dimension
		  +		appendSetDimension()		<-SetDimension
		  +		appendRangeDimension(<d>)	<-RangeDimension
		  +		appendAliasRangeDimension()	<-RangeDimension
		  +		appendSampledDimension(d)	<-SampledDimension
		|(+)|	createSetDimension(idx)		<-SetDimension
		|(+)|	createRangeDimension(idx, <d>)	<-RangeDimension
		|(+)|	createAliasRangeDimension()	<-RangeDimension
		|(+)|	createSampledDimension(idx, d)	<-SampledDimension
		|(+)|	deleteDimensions()			<-b
		  +		getDataDirect(DataType, data, count, offset)	<-v // ???
		  +		setDataDirect(DataType, data, count, offset)	<-v // ???
		|(+)|	dataExtent()				->d
		|(+)|	dataExtent(d)				<-v
		|(+)|	dataType()					->DataType
		  +		appendData(DataType, data, count, axis)
		  ~		ioRead
		  ~		ioWrite
		+|()|	createData(DataType, size)	<-v
		+|()|	hasData()					->b
		+|()|	write(dtype, data, count, offset)
		+|()|	read(dtype, buffer, count, offset)
		
		+Entity +NamedEntity +EntityWithMetaData +EntityWithSources
		
mx		Tag.h (ITag) |TagHDF5| [IBaseTag] {BaseTagHDF5}
		|(+)|	units()						-><s>
		|(+)|	units(<s>)					<-v
		|(+)|	units(n)					<-v
		|(+)|	position()					-><d>
		|(+)|	position(<d>)				<-v
		|(+)|	extent(<d>)					<-v
		|(+)|	extent()					-><d>
		{[+]}	hasReference(id)			->b
		  +		hasReference(DataArray)		->b
		{[+]}	referenceCount()			->d
		{[+]}	getReference(n_id)			->DataArray
		{[+]}	getReference(idx)			->DataArray
		{[+]}	addReference(id)			<-v
		  +		addReference(DataArray)		<-v
		{[+]}	removeReference(id)			<-b
		  +		removeReference(DataArray)	<-b
		  +		references(filter)			-><DataArray>
		  +		references()				-><DataArray>
		{[+]}	references(<DataArray>)		<-v
		{[+]}	hasFeature(id)				->b
		  +		hasFeature(Feature)			->b
		{[+]}	featureCount()				->d
		{[+]}	getFeature(id)				->Feature
		{[+]}	getFeature(idx)				->Feature
		  +		features(filter)			-><Feature>
		  +		createFeature(DataArray, LinkType)	<-Feature
		{[+]}	createFeature(id, LinkType)			<-Feature
		{[+]}	deleteFeature(id)			<-b
		  +		deleteFeature(Feature)		<-b
		  +		retrieveData(ref_idx)		->DataView
		  +		retrieveData(id)			->DataView
		  +		retrieveFeature(feat_idx)	->DataView
		  +		retrieveFeature(id)			->DataView
		
		+Entity +NamedEntity +EntityWithMetaData +EntityWithSources

mx		MultiTag.h (IMultiTag) |MultiTagHDF5| [IBaseTag] {BaseTagHDF5}
		|(+)|	units()						-><s>
		|(+)|	units(<s>)					<-v
		|(+)|	units(n)					<-v
		|(+)|	position()					-><d>
		|(+)|	position(<d>)				<-v
		|(+)|	hasPosition()				->b
		|(+)|	extent(<d>)					<-v
		|(+)|	extent()					-><d>
		{[+]}	hasReference(id)			->b
		  +		hasReference(DataArray)		->b
		{[+]}	referenceCount()			->d
		{[+]}	getReference(n_id)			->DataArray
		{[+]}	getReference(idx)			->DataArray
		{[+]}	addReference(id)			<-v
		  +		addReference(DataArray)		<-v
		{[+]}	removeReference(id)			<-b
		  +		removeReference(DataArray)	<-b
		  +		references(filter)			-><DataArray>
		  +		references()				-><DataArray>
		{[+]}	references(<DataArray>)		<-v
		{[+]}	hasFeature(id)				->b
		  +		hasFeature(Feature)			->b
		{[+]}	featureCount()				->d
		{[+]}	getFeature(id)				->Feature
		{[+]}	getFeature(idx)				->Feature
		  +		features(filter)			-><Feature>
		  +		createFeature(DataArray, LinkType)	<-Feature
		{[+]}	createFeature(id, LinkType)			<-Feature
		{[+]}	deleteFeature(id)			<-b
		  +		deleteFeature(Feature)		<-b
		  +		retrieveData(ref_idx)		->DataView
		  +		retrieveData(id)			->DataView
		  +		retrieveFeature(feat_idx)	->DataView
		  +		retrieveFeature(id)			->DataView
		  -		checkDimensions(DataArray, DataArray)		// ???
		
		+Entity +NamedEntity +EntityWithMetaData +EntityWithSources
		
mx		Value.h
		uncertainty				d
		reference				s
		filename				s
		encoder					s
		checksum				s

		data.set();.get()
		type()					->DataType
		supportsType(DataType)	->b
		
mx		DataSet.h |H5DataSet|
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


