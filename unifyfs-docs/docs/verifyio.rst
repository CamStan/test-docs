=========================================
VerifyIO: Determine UnifyFS Compatibility
=========================================

--------
VerifyIO
--------

VerifyIO_ can be used to determine an application's compatability with UnifyFS
as well as help narrow down what an application may need to change to become
compatible with UnifyFS.

VerifyIO is a tool within the Recorder_ tracing framework that takes application
traces from Recorder and determines whether I/O synchronization is correct based
on the underlying file system semantics (e.g., POSIX, commit) and
synchronization semantics (e.g., POSIX, MPI).

Run VerifyIO with commit semantics on the application in question to determine
compatibility with UnifyFS.

----------

--------------
VerifyIO Guide
--------------

To use VerifyIO, the Recorder library needs to be installed. See the `Recorder
README`_ for full instructions on how to build and run Recorder.

Build
*****

Clone Recorder, using the ``pilgrim`` branch:

.. code-block:: Bash
    :caption: Clone

    $ git clone https://github.com/uiuc-hpc/Recorder.git

Determine the install locations of the MPI-IO and HDF5 libraries being used by
the application and pass those paths to Recorder at configure time.

.. code-block:: Bash
    :caption: Configure, Make, and Install

    $ deps_prefix="${mpi_install};${hdf5_install}"
    $ mkdir -p build install

    $ cd build
    $ cmake -DCMAKE_INSTALL_PREFIX=../install -DCMAKE_PREFIX_PATH=$deps_prefix ../Recorder
    $ make
    $ make install

    # Capture Recorder source code and install locations
    $ export RECORDER_SRC=/path/to/Recorder/source/code
    $ export RECORDER_ROOT=/path/to/Recorder/install

Python3 and the ``recorder-viz`` and ``networkx`` packages are also required to
run the final VerifyIO verification code.

.. code-block:: Bash
    :caption: Install Python Packages 

    $ module load python/3.x.x
    $
    $ pip3 install recorder-viz --user
    $ pip3 install networkx --user

Run
***

Before capturing application traces, it is recommended to disable data sieving
as VerifyIO will flag this as incompatible under commit semantics.

.. code-block:: Bash
    :caption: Disable Data Sieving

    echo -e "romio_ds_write disable\nromio_ds_read disable" > /path/to/romio_hints
    export ROMIO_HINTS=/path/to/romio_hints
    export ROMIO_PRINT_HINTS=1   #optional


Run the application with Recorder to capture the traces using the equivalent
``-env`` (mpirun) option for the available workload manager.

.. code-block:: Bash
    :caption: Capture Traces 

    srun -N $nnodes -n $nprocs --export=ALL,LD_PRELOAD=$RECORDER_ROOT/lib/librecorder.so example_app_executable

Recorder places the trace files in a folder within the current working directory
named ``hostname-username-appname-pid-starttime``.

If desired (e.g., for debugging), generate human-readable traces from the
captured trace files.

.. code-block:: Bash
    :caption: Generate Human-readable Traces 

    $RECORDER_ROOT/bin/recorder2text /path/to/traces &> recorder2text.out

This will generate text-format traces in the folder ``path/to/traces/_text``.

Next, run the Recorder conflict detector to capture **potential** conflicts. The
``--semantics=`` option needs to match the semantics provided by the underlying
file system. In the case of UnifyFS, use ``commit`` semantics.

.. code-block:: Bash
    :caption: Capture Potential Conflicts 

    $RECORDER_ROOT/bin/conflict_detector /path/to/traces --semantics=commit &> conflict_detector_commit.out

The potential conflicts will be recorded to the file
``path/to/traces/conflicts.txt``. 

Lastly, run VerifyIO with the traces and potential conflicts to determine
whether all I/O operations are properly synchronized under the desired standard
(e.g., POSIX, MPI).

.. code-block:: Bash
    :caption: Run VerifyIO 

    # Evaluate using POSIX standard
    python3 $RECORDER_SRC/tools/verifyio/verifyio.py /path/to/traces /path/to/traces/conflicts.txt --semantics=posix &> verifyio_commit_results.posix

    # Evaluate using MPI standard
    python3 $RECORDER_SRC/tools/verifyio/verifyio.py /path/to/traces /path/to/traces/conflicts.txt --semantics=mpi &> verifyio_commit_results.mpi

Interpreting Results
********************

In the event VerifyIO shows an incompatibility, or the results are not clear,
don't hesitate to contact the UnifyFS team `mailing list`_ for aid in
determining a solution.

Conflict Detector Results
^^^^^^^^^^^^^^^^^^^^^^^^^

When there are no potential conflicts, the conflict detector output simply
states as much:

.. code-block:: none

    [prompt]$ cat conflict_detector_commit.out
    Check potential conflicts under Commit Semantics
    ...
    No potential conflict found for file /path/to/example_app_outfile

When potential conflicts exists, the conflict detector prints a list of each
conflicting pair. For each operation within a pair, the output contains the
process rank, sequence ID, offset the conflict occured at, number of bytes
affected by the operation, and whether the operation was a write or a read.
This format is printed at the top of the output.

.. code-block:: none

    [prompt]$ cat conflict_detector_commit.out
    Check potential conflicts under Commit Semantics
    Format:
    Filename, io op1(rank-seqId, offset, bytes, isRead), io op2(rank-seqId, offset, bytes, isRead)

    /path/to/example_app_outfile, op1(0-244, 0, 800, write), op2(0-255, 0, 96, write)
    /path/to/example_app_outfile, op1(0-92, 4288, 2240, write), op2(0-148, 4288, 2216, read)
    /path/to/example_app_outfile, op1(1-80, 6528, 2240, write), op2(1-136, 6528, 2216, read)
    ...
    /path/to/example_app_outfile, op1(0-169, 18480, 4888, write), op2(3-245, 18848, 14792, read)
    /path/to/example_app_outfile, op1(0-169, 18480, 4888, write), op2(3-246, 18848, 14792, write)
    /path/to/example_app_outfile, op1(0-231, 18480, 16816, write), op2(3-245, 18848, 14792, read)
    /path/to/example_app_outfile, Read-after-write (RAW): D-2,S-5, Write-after-write (WAW): D-1,S-2

The final line printed contains a summary of all the potential conflicts.
This consists of the total number of read-after-write (RAW) and
write-after-write (WAW) operations performed by different processes or the same
process.

VerifyIO Results
^^^^^^^^^^^^^^^^

VerifyIO takes the traces and potential conflicts and checks if each conflicting pair is properly synchronized. Refer to the `VerifyIO README <VerifyIO>`_ for a
description on what determines proper synchronization for a conflicting I/O
pair.

Compatible with UnifyFS 
"""""""""""""""""""""""

In the event that there are no potential conflicts, or each potential conflict
was performed by the same rank, VerifyIO will report the application as being
properly synchronized and therefore compatible with UnifyFS.

.. code-block:: none

    [prompt]$ cat verifyio_commit_results.posix
    Rank: 0, intercepted calls: 79, accessed files: 5
    Rank: 1, intercepted calls: 56, accessed files: 2
    Building happens-before graph
    Nodes: 46, Edges: 84

    Properly synchronized under posix semantics


    [prompt]$ cat verifyio_commit_results.mpi
    Rank: 0, intercepted calls: 79, accessed files: 5
    Rank: 1, intercepted calls: 56, accessed files: 2
    Building happens-before graph
    Nodes: 46, Edges: 56

    Properly synchronized under mpi semantics

When there are potential conflicts from different ranks but the proper
synchronization has occured, VerifyIO will also report the application as being
properly synchronized.

.. code-block:: none

    [prompt]$ cat verifyio_commit_results.posix
    Rank: 0, intercepted calls: 510, accessed files: 8
    Rank: 1, intercepted calls: 482, accessed files: 5
    Rank: 2, intercepted calls: 481, accessed files: 5
    Rank: 3, intercepted calls: 506, accessed files: 5
    Building happens-before graph
    Nodes: 299, Edges: 685
    Conflicting I/O operations: 0-169-write <--> 3-245-read, properly synchronized: True
    Conflicting I/O operations: 0-169-write <--> 3-246-write, properly synchronized: True
    Conflicting I/O operations: 0-169-write <--> 3-246-write, properly synchronized: True

    Properly synchronized under posix semantics

Incompatible[*]_ with UnifyFS 
"""""""""""""""""""""""""""""

In the event there are potential conflicts from different ranks but the proper
synchronization has **not** occured, VerifyIO will report the application as not
being properly synchronized and therefore incompatible[*]_ with UnifyFS.

.. code-block:: none

    [prompt]$ cat verifyio_commit_results.mpi
    Rank: 0, intercepted calls: 510, accessed files: 8
    Rank: 1, intercepted calls: 482, accessed files: 5
    Rank: 2, intercepted calls: 481, accessed files: 5
    Rank: 3, intercepted calls: 506, accessed files: 5
    Building happens-before graph
    Nodes: 299, Edges: 427
    0-169-write --> 3-245-read, properly synchronized: False
    0-169-write --> 3-246-write, properly synchronized: False
    0-169-write --> 3-246-write, properly synchronized: False

    Not properly synchronized under mpi semantics

.. [*] Incompatible here does not mean the application cannot work with UnifyFS
   at all, just under the default configuration. There are
   :doc:`workarounds <limitations>` available that could very easily change this
   result (VerifyIO plans to have options to run under the assumption some
   workarounds are in place). Should your outcome be an incompatible result,
   please contact the UnifyFS `mailing list`_ for aid in finding a solution.

.. explicit external hyperlink targets

.. _mailing list: ecp-unifyfs@exascaleproject.org
.. _Recorder: https://github.com/uiuc-hpc/Recorder
.. _Recorder README: https://github.com/uiuc-hpc/Recorder/blob/pilgrim/README.md
.. _VerifyIO: https://github.com/uiuc-hpc/Recorder/tree/pilgrim/tools/verifyio#note-on-the-third-step
