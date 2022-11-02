**SnapFuzz currently only works with LightFTP, Live555, TinyDTLS, Dnsmasq and Dcmtk!**

# SnapFuzz

SnapFuzz is a novel fuzzing framework for network applications. SnapFuzz offers a robust architecture that transforms slow asynchronous network communication into fast synchronous communication, snapshots the target at the latest point at which it is safe to do so, speeds up file operations by redirecting them to a custom in-memory filesystem, and removes the need for many fragile modifications, such as configuring time delays or writing clean-up scripts.

For more information about SnapFuzz, please check the repository at <https://github.com/srg-imperial/SnapFuzz>

# Running SnapFuzz with ProFuzzBench

ProFuzzBench comes with Dockerfiles and scripts to run SnapFuzz with the benchmark.
Every target includes a `Dockerfile-snapfuzz` that builds the SnapFuzz fuzzer, and a re-builds the target to be run with SnapFuzz. This Dockerfile builds on top of the default Dockerfile for the target.

To build a target for SnapFuzz:
```bash
cd $PFBENCH
cd subjects/FTP/LightFTP
docker build . -t lightftp
docker build . -f Dockerfile-snapfuzz -t lightftp-snapfuzz
```

<!--To build all targets for all fuzzers, you can run the script [profuzzbench_build_all.sh](scripts/execution/profuzzbench_build_all.sh). To run the fuzzers on all targets, you can use the script [profuzzbench_exec_all.sh](scripts/execution/profuzzbench_exec_all.sh).-->

You can fuzz an individual target with SnapFuzz in the same way of other fuzzers. For example:
```bash
cd $PFBENCH
mkdir results-lightftp

profuzzbench_exec_common.sh lightftp-snapfuzz 4 results-lightftp snapfuzz/aflnet out-lightftp-snapfuzz "-P FTP -D 10000 -q 3 -s 3 -E -K -m none -t 1000" 3600 5
```

Please see the main [README.md](README.md) for more information about how to run and analyze experiments with ProFuzzBench.

