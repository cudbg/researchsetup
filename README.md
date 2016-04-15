


# Running Experiments

You want your experiments to be reproducible because they will fail the first (and second, third) times, so you will want an easy way to examine and debug them.  The following are some basic instructions.


Put things in a database (say, SQLite3).  You probably want ta schema similar to the following

        experiment_name   // what experiment are you running?  
        run_id            // you probably want to run multiple times and compute mean/std/CI statistics
        dataset(s)        // if you are using different datasets 
        seed              // if there is ANY source of randomness in your experiment, use a seed so you can reproduce it later
        parameter(s)      // what are you varying?  Each should be an attribute.  If unused, set the value to NULL
        measure(s)        // what you are measuring.  precision, recall, latency etc.
                          // store the RAW measures that can be used to compute aggregate measures
        timestamp(s)      // in systems work, you should be collecting timestamps.  Store them!

Some rules of thumb:

* Parameters go on the x axis
* Metrics are computed from the measures, and go on the y axis
  * Never _ever_ compute aggregate statistics in your code and log only those.  Always log all of the data and compute statistics later!
* _Always_ use a seed and record what it is
* If the above table gets too wide, it's ok to denormalize.

If you want to go whole hog, try [ReproZip](https://vida-nyu.github.io/reprozip/)


# Plotting experiments

* For the vast majority of plots, `ggplot2` in `R` is the way to go.  [Find a tutorial](https://www.google.com/?q=ggplot2%20tutorial) and follow it.
* If you use python for everything like I do, try the [pygg](https://github.com/sirrice/pygg) library.  It gives you ggplot2 syntax in python.  It can't handle multiple layers, which you probably shouldn't be doing anyways.
