# FireSim Paper Workloads

* Untested as of 08/07/2024 *

Old FireSim software workloads used in the FireSim paper (see below).
This software needs the following to build:

1. Setup FireMarshal, FireSim's workload builder: https://github.com/firesim/FireMarshal. This was last tested with commit `74ac78a`.
2. Ensure you have all FireMarshal requirements on your path (i.e. if you are using Chipyard, ensure the `env.sh` is setup)
3. Symlink the FireMarshal repository to this repository like so:


```
ln -sf <PATH_TO_FIREMARSHAL> firemarshal-symlink
```

4. Build using `make allpaper` or `make linux-poweroff`

## Old Documentation

[DEPRECATED] ISCA 2018 Experiments
==================================

.. Note::

   These instructions are deprecated. Users interested in reproducing the ISCA 2018 experiments
   should build workloads using FireMarshal present in Chipyard.

This page contains descriptions of the experiments in our `ISCA 2018 paper
<https://sagark.org/assets/pubs/firesim-isca2018.pdf>`__ and instructions for
reproducing them on your own simulations.

One important difference between the configuration used in the ISCA
2018 paper and the open-source release of FireSim is that the ISCA
paper used a proprietary L2 cache design that is not open-source.
Instead, the open-source FireSim uses an LLC model that models the
behavior of having an L2 cache as part of the memory model. Even with
the LLC model, you should be able to see the same trends in these
experiments, but exact numbers may vary.

Each section below describes the resources necessary to run the experiment.
Some of these experiments require a large number of instances -- you should
make sure you understand the resource requirements before you run one of the
scripts.

**Compatiblity**: These were last tested with commit
``4769e5d86acf6a9508d2b5a63141dc80a6ef20a6`` (Oct. 2019) of FireSim. After this commit,
the Linux version in FireSim has been bumped past Linux 4.15. To reproduce workloads
that rely on OS behavior that has changed, like
``memcached-thread-imbalance``, you must use the last tested Oct. 2019 commit.


Prerequisites
-------------------------

These guides assume that you have previously followed the
single-node/cluster-scale experiment guides in the FireSim documentation. Note
that these are **advanced** experiments, not introductory tutorials.


Building Benchmark Binaries/Rootfses
--------------------------------------

We include scripts to automatically build all of the benchmark rootfs images
that will be used below. To build them, make sure you have already run
``./marshal build workloads/br-base.json`` in ``firesim/target-design/chipyard/software/firemarshal``, then run:

.. code-block:: bash

    cd firesim/target-design/chipyard/software/firesim-paper-workloads
    make allpaper


Figure 5: Ping Latency vs. Configured Link Latency
-----------------------------------------------------

Resource requirements:

.. code-block:: bash

    cat firesim/target-design/chipyard/software/firesim-paper-workloads/firesim-yaml-configs/ping-latency-config.yaml

To Run:

.. code-block:: bash

    cd firesim/target-design/chipyard/software/firesim-paper-workloads/firesim-yaml-configs
    ./run-ping-latency.sh withlaunch



Figure 6: Network Bandwidth Saturation
-----------------------------------------------

Resource requirements:

.. code-block:: bash

    cat firesim/target-design/chipyard/software/firesim-paper-workloads/firesim-yaml-configs/bw-test-config.yaml

To Run:

.. code-block:: bash

    cd firesim/target-design/chipyard/software/firesim-paper-workloads/firesim-yaml-configs
    ./run-bw-test.sh withlaunch


Figure 7: Memcached QoS / Thread Imbalance
-----------------------------------------------

Resource requirements:

.. include:: /../deploy/workloads/memcached-thread-imbalance-config.yaml
   :start-line: 3
   :end-line: 6
   :code: yaml

To Run:

.. code-block:: bash

    cd firesim/target-design/chipyard/software/firesim-paper-workloads/firesim-yaml-configs
    ./run-memcached-thread-imbalance.sh withlaunch


Figure 8: Simulation Rate vs. Scale
----------------------------------------

Resource requirements:

.. include:: /../deploy/workloads/simperf-test-scale-config.yaml
   :start-line: 3
   :end-line: 6
   :code: yaml


To Run:

.. code-block:: bash

    cd firesim/target-design/chipyard/software/firesim-paper-workloads/firesim-yaml-configs
    ./run-simperf-test-scale.sh withlaunch


A similar benchmark is also provided for supernode mode, see ``run-simperf-test-scale-supernode.sh``.


Figure 9: Simulation Rate vs. Link Latency
---------------------------------------------

Resource requirements:

.. include:: /../deploy/workloads/simperf-test-latency-config.yaml
   :start-line: 3
   :end-line: 6
   :code: yaml


To Run:

.. code-block:: bash

    cd firesim/target-design/chipyard/software/firesim-paper-workloads/firesim-yaml-configs
    ./run-simperf-test-latency.sh withlaunch


A similar benchmark for supernode mode will be provided soon. See https://github.com/firesim/firesim/issues/244


Running all experiments at once
-----------------------------------

This script simply executes all of the above scripts in parallel. One caveat
is that the bw-test script currently cannot run in parallel with the others,
since it requires patching the switches. This will be resolved in a future
release.

.. code-block:: bash

    cd firesim/target-design/chipyard/software/firesim-paper-workloads/firesim-yaml-configs
    ./run-all.sh

.. _gap-benchmark-suite:

GAP Benchmark Suite
=====================
You can run the reference implementation of the GAP (Graph Algorithm Performance)
Benchmark Suite. We provide scripts that cross-compile the graph kernels for RISCV.

For more information about the benchmark itself, please refer to the site:
http://gap.cs.berkeley.edu/benchmark.html

Some notes:

-  Only the Kron input graph is currently supported.
-  Benchmark uses ``graph500`` input graph size of 2^20 vertices by default. ``test`` input size has 2^10 vertices and can be used by specifying an argument into make: ``make gapbs input=test``
-  The reference input size with 2^27 verticies is not currently supported.

By default, the gapbs workload definition runs the benchmark multithreaded with number of threads equal to the number of cores. To change the number of threads, you need to edit ``firesim/deploy/workloads/runscripts/gapbs-scripts/gapbs.sh``. Additionally, the workload does not verify the output of the benchmark by default. To change this, add a ``--verify`` parameter to the json.

To Build Binaries and RootFSes:

.. code-block:: bash

    cd firesim/deploy/workloads/
    make gapbs

Run Resource Requirements:

.. include:: /../deploy/workloads/gapbs.yaml
   :start-line: 3
   :end-line: 6
   :code: yaml


To Run:

.. code-block:: bash

    ./run-workload.sh workloads/gapbs.yaml --withlaunch

Simulation times are host and target dependent. For reference, on a
four-core rocket-based SoC with a DDR3 + 1 MiB LLC model, with a 90
MHz host clock, ``test`` and ``graph500`` input sizes finish in a few minutes.
