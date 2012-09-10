append-only-storage
===================

Learning task repository. Task is to create C++ library to deal with append-only storage.

Library functions:
1) store arbitrary large object and associate unique identifier with it;
2) read object associated with unique identifier;
3) delete object associated with unique identifier.

Storage usage pattern:
1) approximately 1 write per 10 reads;
2) single deletions are rare;
3) massive deletions are very rare;
4) object size is unpredictable (1KB - 100MB or more); tens of millions of small objects can be stored as well as hundreds of enormous one;
5) total amount of stored data is up to several TB.

Special requirements on storage structure:
1) it is possible to easily back up data without function pause;
2) it is possible to defragment storage after massive deletions;
3) safe for power interruptions.

Since task is of learning kind no multi-threading support is required.

