# Conditional channel execution 

## Problem 

Two different tasks need to be executed in a mutually exclusive manner, 
then a third task should post-process the results of the previous execution.
This is an alternative approach to `conditional-process`.

## Solution

Conditionally create the input channels normally (with data) or as 
[empty](https://www.nextflow.io/docs/latest/channel.html#empty) channels. 
The process consuming the individual input channels will only execute if 
the channel is populated. Each process still declares its own output channel.

Then use the [mix](https://www.nextflow.io/docs/latest/operator.html#mix) operator to create 
a new channel that will emit the outputs produced by the two processes and use it as the input
for the third process.

## Code 

```nextflow
params.flag = false

if (params.flag) {
    Channel.empty().set{foo_inch}
    Channel.from(1,2,3).set{bar_inch}
} else {
    Channel.from(4,5,6).set{foo_inch}
    Channel.empty().set{bar_inch}
}

process foo {

  input:
  val(f) from foo_inch

  output:
  file 'x.txt' into foo_ch

  script:
  """
  echo $f > x.txt
  """
}

process bar {
  input:
  val(b) from bar_inch

  output:
  file 'x.txt' into bar_ch

  script:
  """
  echo $b > x.txt
  """
}

process omega {
  echo true
  input:
  file x from foo_ch.mix(bar_ch)

  script:
  """
  cat $x
  """
}
```

## Run it

Use the the following command to execute the example:

    nextflow run main.nf

The processes `foo` and `omega` are executed. Run the same command 
with the `--flag` command line option. 

    nextflow run main.nf --foo 

This time the processes `bar` and `omega` are executed.
