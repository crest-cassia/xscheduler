(This repository is work in progress!)

# eXtensible Scheduler

A wrapper for job schedulers.
Job schedulers used in HPCs, such as Torque, often have its own I/O format.
Users have to change their scripts to conform with its dialect.

To solve this problem, we propose a unified I/O format for job scheduler.
This gem is one implementation that wraps Torque to have the proposed specification.
By extending this gem, you can use several kinds of schedulers using a unified format.

## Proposed Specification

Scheduler has three commands **xsub**, **xstat**, and **xdel**.
These correspond to qsub, qstat, and qdel of Torque.

It prints JSON to the standard output. The format shown later is the minimal one. You can add arbitrary fields if you want.

### xsub

submit a job to a scheduler

- **usage**: `xsub {job_script}`
- **options**:
  - "-d WORKDIR" : set working directory
    - this directory is used to make a temporary job file or to store scheduler log files.
    - when the job is executed, the current directory is set to this working directory.
  - "-p PARAMETERS" : set parameters required to submit a job
  - "-t" : show parameters to submit a job in JSON format. Job is not submitted.
  - "-l" : Log file path. If not this option is not given, the logs are printed to STDERR.

- **output format**:
  - when "-t" option is given, it prints JSON as follows.
    - it must have a "parameters" field.
    - Each parameter has "description" field and "default" field. "description" field is optional.
  ```json
  {
    "parameters": {
      "mpi_procs": {
        "description": "MPI process",
        "default": 1
      },
      "omp_threads": {
        "description": "OMP threads",
        "default": 1
      },
      "ppn": {
        "description": "Process per node",
        "default": 1
      },
      "elapsed": {
        "description": "Limit on elapsed time",
        "default": "1:00:00"
      }
    }
  }
  ```

  - when job is submitted, the output format looks like the following.
    - it must have a key "job_id". The value of job_id can be either a number or a string.
  ```json
  {
    "job_id": 21507
  }
  ```
  - When it succeeds, return code is zero. When it fails, return code is non-zero.

### xstat

show a status of a job

- **usage**: `xstat {job_id}` or `xstat`
  - when "job_id" is given, show the status of the job
  - when "job_id" is not given, show current the status of the scheduler

- **output format**:
  - when "job_id" is given, it prints JSON as follows.
  ```json
  {
    "status": "running",
    "log_paths": ["/foo/bar/scheduler_log.o12345", "/foo/bar/scheduler_log.e12345"]
  }
  ```
    - status field takes either "queued", "running", or "finished".
      - "queued" means the job is in queue.
      - "running" means the job is running.
      - "finished" means the job is finished or the job is not found.
    - "log_paths" fileds has an array of paths to scheduler log files.
  - when job_id is not given, the output format is arbitrary.
    - it usually prints the output of `qsub` command.

### xdel

delete a job

- **usage**: `xdel {job_id}`
  - cancel the specified job
    - if the job finished successfully, return code is zero.
    - if the job is not found, it returns non-zero.
  - output format is arbitrary.

## Installation

Add this line to your application's Gemfile:

    gem 'any_scheduler'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install any_scheduler

## Usage

TODO: Write usage instructions here

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
