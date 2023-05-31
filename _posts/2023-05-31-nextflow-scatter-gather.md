---
tags:
  - nextflow
---

Sometimes when building a workflow you'll run into a situation where one of the steps in your workflow takes much longer than it should. If you are lucky the problematic task is ["embarassingly parallelizable"](https://en.wikipedia.org/wiki/Embarrassingly_parallel) you can easily accellerate the analysis by splitting the work performed by a single task over many tasks that run in parallel. 

![serial vs parallel processing](/assets/images/serial_vs_parallel.png)

One place this scatter/gather stragegy can come up in bioinformatics is when pre-processing on a large number of reads. For example, detecting rare events such as CRISPR off-target editing or translocations can require sequencing tens of millions of reads. To manage PCR duplicates library preparation strategies add UMIs to the DNA library. [UMI-tools](https://github.com/CGATOxford/UMI-tools) is a popular tool for working with UMIs, but as of the time of writing this it only supports single-thread processing. When processing millions of reads with a single thread UMI assignment becomes the most timeconsuming step in most workflows.

Luckily, to assign a UMI you only need the information in the read sequence, which means the process can be parallelized by splitting the FASTQ file into chunks and running the computation across many CPUs. As it turns out, it is only a few lines of code to convert a single-threaded process to one that executes in parallel using using Nextflow's built-in channel operators "flatten" and "reduce".

Given an initial workflow that processes FASTQ files using UMI-tools in a single thread

```
```

This workflow can be parallelized by adding logic for scatter and gather.

```
process map {

}

def reshape() {

}
```

If the problem you are working on is embarassingly parallel at the input (FASTQ) level this approach can be used to accelerate the computation without adding the complexity of multi-threading, allowing the application logic that performs the calculation to be kept simple and single threaded.

## Note on cost

One neat thing about the scatter/gather strategy is that it is cost-neutral; you pay for the same total number of CPU hours to run the workflow, they just run in parallel rather than sequentially. If you are running your workflow with on-prem infrastructure and are limited by the number of CPUs in the cluster will limit the amount of acceleration possible, while managed batch compute cloud services will take care of scaling your compute cluster to the size of your workload.

As long as the scatter and gather steps run quickly you will pay pennies of overhead for parallization in return for hours of wall-time shaved off your workflow executions.

## Unsolicited advice

Every line of code you add to a pipeline is another place that a bug could be hiding, increases the maintenance burden when making a change, and steepens the learning curve when on-board new team members to a project. While nextflow makes implementing map-reduce in a genomics pipeline very easy, I would recommend carefully evaluating whether the benefits of parallelization, and use judiciously. Avoid premature optimization!
