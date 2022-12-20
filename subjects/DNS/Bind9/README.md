Please carefully read the [main README.md](../../../README.md), which is stored in the benchmark's root folder, before following this subject-specific guideline.

**StateAFL can't compile bind9**

# Fuzzing Bind9 server with AFLNet and AFLnwe
Please follow the steps below to run and collect experimental results for Bind9, which is a lightweight DNS server.

## Step-1. Build a docker image
The following commands create a docker image tagged Bind9. The image should have everything available for fuzzing and code coverage calculation.

```bash
cd $PFBENCH
cd subjects/DNS/Bind9
docker build . -t bind9
```

## Step-2. Run fuzzing
The following commands run 4 instances of AFLNet and 4 instances of AFLnwe to simultaenously fuzz Bind9 in 60 minutes.

```bash
cd $PFBENCH
mkdir results-bind9

profuzzbench_exec_common.sh bind9 4 results-bind9 aflnet out-bind9-aflnet "-P DNS -D 10000 -K -R -q 3 -s 3 -m none" 3600 5 &
profuzzbench_exec_common.sh bind9 4 results-bind9 aflnwe out-bind9-aflnwe "-P DNS -D 10000 -K -R -q 3 -s 3 -m none" 3600 5
```

## Step-3. Collect the results
The following commands collect the  code coverage results produced by AFLNet and AFLnwe and save them to results.csv.

```bash
cd $PFBENCH/results-bind9

profuzzbench_generate_csv.sh bind9 4 aflnet results.csv 0
profuzzbench_generate_csv.sh bind9 4 aflnwe results.csv 1
```

## Step-4. Analyze the results
The results collected in step 3 (i.e., results.csv) can be used for plotting. Use the following command to plot the coverage over time and save it to a file.

```
cd $PFBENCH/results-Bind9

profuzzbench_plot.py -i results.csv -p bind9 -r 4 -c 60 -s 1 -o cov_over_time.png
```
