AriesNCL v1.0
=============

**Aries** **N**etwork Performance **C**ounters Monitoring **L**ibrary

AriesNCL is a library to monitor and record network router tile performance counters on the Aries router of Cray’s Cascade/XC30 platform.

### Build

```
make
```

Make sure module papi is loaded before compiling. You may also need to unload
the darshan module.

### C API

Call the function below to initialize PAPI and set up the counters. It expects
a file called 'counters.txt' in the same directory as the executable with a
newline-delimited list of counter names to record:
```
void InitAriesCounters(char *progname, int my_rank, int reporting_rank_mod,
int* AC_event_set, char*** AC_events, long long** AC_values, int* AC_event_count)
```

Start recording counters:
```
void StartRecordAriesCounters(int my_rank, int reporting_rank_mod, int*
AC_event_set, char*** AC_events, long long** AC_values, int* AC_event_count)
```

Stop recording counters. Stores counters in memory until FinalizeAriesCounters called:
```
void EndRecordAriesCounters(int my_rank, int
reporting_rank_mod, int* AC_event_set, char*** AC_events,
long long** AC_values, int* AC_event_count)
```

Writes out all the counters to YAML files, cleans up memory, stops PAPI:
```
void FinalizeAriesCounters(MPI_Comm *mod16_comm, int my_rank, int reporting_rank_mod,
int* AC_event_set, char*** AC_events, long long** AC_values, int* AC_event_count)
```

### Test

For an example test, see the tests folder. The test is currently configured for the
KNL nodes on Cori, and it should be run on 2 nodes with 64 ranks per node:
```
cd tests
make
srun -n 128 ./regions
```

To setup the variables required by the function calls look at tests/regions.c or the
following (this example is for 16 MPI ranks per node):

	int AC_event_set;
	char** AC_events;
	long long * AC_values;
	int AC_event_count;
	int numtasks;
	MPI_Comm_size(MPI_COMM_WORLD, &numtasks);
	MPI_Group mod16_group, group_world;
	MPI_Comm mod16_comm;
	int members[(numtasks-1)/16 + 1];
	int rank;
	for (rank=0; rank<((numtasks-1)/16 + 1); rank++)
	{
	members[rank] = rank*16;
	}
	MPI_Comm_group(MPI_COMM_WORLD, &group_world);
	MPI_Group_incl(group_world, ((numtasks-1)/16 + 1), members, &mod16_group);
	MPI_Comm_create(MPI_COMM_WORLD, mod16_group, &mod16_comm);

Since every node will record the same counter information, we only record it on
one rank per node (we could have up to 3 redudant records on Edison but we
cannot easily figure out how many nodes we own on a router). The
reporting_rank_mod is the number of ranks per node.

These must be run on nodes with papi support (i.e. Cray's NPU component). All of
Edison's compute nodes have this.

Running the test will output YAML files mpitest.nettiles.1.yaml and
mpitest.proctiles.1.yaml, containing the counter data for NIC tiles and processor tiles,
respectively.

### Release

Copyright (c) 2014, Lawrence Livermore National Security, LLC.
Produced at the Lawrence Livermore National Laboratory.

Written by:
```
    Dylan Wang <dylan.wang@gmail.com>
    Staci Smith <smiths949@email.arizona.edu>
    Abhinav Bhatele <bhatele@llnl.gov>
```

LLNL-CODE-678960. All rights reserved.

This file is part of AriesNCL. For details, see:
https://github.com/LLNL/ariesncl.
Please also read the LICENSE file for our notice and the LGPL.
