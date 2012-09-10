append-only-storage
===================

Learning task repository. Task is to create C++ library to deal with append-only storage.


Library functions:
------------------

* store arbitrary large object and associate unique identifier with it;
* read object associated with unique identifier;
* delete object associated with unique identifier.


Storage usage pattern:
----------------------

* approximately 1 write per 10 reads;
* single deletions are rare;
* massive deletions are very rare;
* object size is unpredictable (1KB - 100MB or more); tens of millions of small objects can be stored as well as hundreds of enormous one;
* total amount of stored data is up to several TB.


Special requirements on storage structure:
------------------------------------------

* it is possible to easily back up data without function pause;
* it is possible to defragment storage after massive deletions;
* safe for power interruptions.


Since task is of learning kind no multi-threading support is required.


Solution design
===============

* objects content and stored objects index are separated logically and on filesystem level;
* on logic level objects content is a data stream where new information is appended to the end; no piece of stream can be actually modified or deleted; only append operations allowed;
* on file system level content data stream is a sequence of files, one of which represents end of stream and grows, other remaining static; sometimes growing stops and a new file is created and afterwards represents end of stream;
* on logic level index is modifiable table that stores correspondence: object unique identifier -> (object content begin position in data stream, object content length, special mark bit for deleted objects);
* unique identifier is a natural number that grows by one whenever new object is stored;
* on file system level index is somewhat similar to data stream; it is ordered sequence of files, of which only last one is edited and stores information about storage modifications (new objects stored and some deletions commited); all other files of index remain static; file order is important because different files may contain different deletion mark bit for the same object identifier; to resolve this contradictions library considers information of next files as superseding any information of previous files.


Why storage design is overcomplicated? What purpose do you split data stream and index into several files for? Short answer: this is to fulfil special requirements 1 and 2 -- backup and defragmentation support.


Backup procedure
----------------

* stop processing any writes and deletions for a moment;
* create new empty data stream file, create new empty storage index file;
* continue processing writes and deletions, storing new objects content to new data stream file, saving writes and deletions information to new index file;
* back up all static files of storage (that is all files except two newly created at second step in background while storage is functioning in normal mode.

Note: since files remain static forever backup procedure can be implemented in incremental way.


Background defragmentation procedure
------------------------------------

* create storage backup as described above;
* iterate through all index files of backup and figure out which objects are deleted and which are not;
* create new append-only storage; copy all non-deleted objects from backup to new storage; use old unique identifiers; delete backup; now you have new storage that contain the same objects with the same indentifiers; new storage has zero fragmentation.

Note: you may defragment storage parts independently; any sequence of objects that were stored between backups (between synchronous creation of new data stream file and new index file) can be backed up and defragmented independently of other objects.


How to replace fragmented hot storage part with defragmented one in background
--------------------------------------------------------------------------

* copy defragmented layout data stream and index files next to fragmented ones;
* stop common library operation for a moment;
* consider ordered sequence of index files; locate consequtive interval of index files of part that you want replace with defragmented layout; replace this interval with index file of defragmented layout;
* continue library operation;
* now old layout index files are unnecessary since they are not included in index sequence; fragmented data stream files are unnecessary since no index file in use point to their content; delete unnecessary files in background while library functions in normal mode.

Note: you can perform defragmentatio on cold backup server in order to save hot producation servers disk usage for request processing.
