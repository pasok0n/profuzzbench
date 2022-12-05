Please carefully read the [main README.md](../../../README.md), which is stored in the benchmark's root folder, before following this subject-specific guideline.

# Fuzzing Modbus TCP server with AFLNet
Please follow the steps below to run and collect experimental results for M-bus.

## Step-1. Build a docker image
The following commands create a docker image tagged modbustcp. The image should have everything available for fuzzing and code coverage calculation.

```bash
cd $PFBENCH
cd subjects/MODBUS/M-bus
docker build . -t modbustcp
```

## Step-2. Run fuzzing
The following commands run 4 instances of AFLNet to simultaenously fuzz LightFTP in 60 minutes.

```bash
cd $PFBENCH
mkdir results-modbustcp

profuzzbench_exec_common.sh modbustcp 4 results-modbustcp aflnet out-modbustcp-aflnet "-P MODBUSTCP -D 10000 -q 3 -s 3 -E -R -K" 3600 5
```