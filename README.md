A lot of life is about regret minimization.  Here are some tips to not be too sad in the future.


# Deployments

You'll likely setup many application/server deployments as part of research.  Automate your deployment

SSH

* google for "passwordless ssh"
* http://www.linuxproblem.org/art_9.html

Fabric

* create a `fabfile.py` in your project directory

        from fabric.api import run, env, local

        env.hosts = ['clic.cs.columbia.edu']

        def taskA():
            run('ls')    # runs on your env.hosts machines

        def taskB():
            local('ls')  # runs on your local machine

* Type the following in the same directory as `fabfile.py` to list the commands

        fab -l

* documentation
  * http://www.fabfile.org/
  * http://docs.fabfile.org/en/1.11/tutorial.html
  * http://docs.fabfile.org/en/1.11/usage/execution.html#execution-strategy



# Experiments

## Running Experiments

        tldr; put all the raw data into a database

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


## Plotting experiments

Simple advice

* Did you compute aggregate values (e.g., mean latency etc)?  Show the standard deviation, or more meaningfully, bootstrapped confidence intervals.  See the appendix for UDF code to compute it in PostgreSQL


Flow chart for picking plots

* TBD


Tools

* For the vast majority of plots, `ggplot2` in `R` is the way to go.  [Find a tutorial](https://www.google.com/?q=ggplot2%20tutorial) and follow it.
* If you use python for everything like I do, try the [pygg](https://github.com/sirrice/pygg) library.  It gives you ggplot2 syntax in python.  It can't handle multiple layers, which you probably shouldn't be doing anyways.


## Presenting Experiments

There are simple steps to better understand what's going on.  These are also the steps for presenting an experiment.  Do the following

1. List your hypothesis
1. What ideal plots will help validate or debunk your hypothesis?
   1. Draw out what the plots should look like if your hypothesis is correct and if incorrect
   1. include x and y axes, shape of curves
1. What will you do to generate these plots?
1. Why do the plots look this way?  
   1. Which parts confirm your hypothesis?  
   1. What's surprising/does not confirm your hypothesis?  Why?
1. What are the next steps to
   1. answer 4ii
   1. use what you learned in your system
1. Go to step 1


# Papers

Managing your papers

* I like to keep one repo per project, but makes releasing code more of a pain:

        SUPERPROJECT/
          docs/
            sigmod_16/
            vldb_16/
            README.md            -- all project notes
          src/
            allmycode/

* David Blei's lab: https://github.com/blei-lab/publications

# Writing papers

It helps to see how other papers evolve.  I've built a tool called `latexsnapshots` that will generate snapshots of a paper as it evolves.  

* [See how my VLDB 2014 paper evolved.](http://eugenewu.net/latexsnapshots/html/)


# Reviewing Papers


        tldr; think like an engineer and look for things to take

Fill in the following form, with an emphasis on the **positive** aspects of the paper:

    # Positives 

    ## What is the key idea in this paper?  Why is it a good idea?

    ## Are there other reasons why this could be a good idea not mentioned in the text?  Suggest them.

    ## Are there other applications where you could use these ideas?  Describe them

    ## Which of the experiments helped you understand the paper?

    ## What else did you like about this paper?

    # Improvements

    ## What writing improvements could have made it easier to understand the paper?

    ## What techniques would make the ideas _even better_?  In what ways

    ## What experiments/plots would help you better understand how well the ideas work?  
    ## Are they _possible_? (remember: getting private data is hard)



         


What other people say about reviewing:

* [https://github.com/jtleek/reviews](https://github.com/jtleek/reviews)



# Jobs

### Faculty Jobs

List of useful links to people's cover letters/cvs/etc

* http://eugenewu.net/job.html
* http://pgbovine.net/faculty-job-application-materials.htm

# Appendix

## Useful Code


PostgreSQL code for defining `95%` confidence intervals


        DROP LANGUAGE plpythonu;
        CREATE LANGUAGE plpythonu;

        DROP FUNCTION IF EXISTS ci_final(numeric[]) cascade;
        CREATE  FUNCTION ci_final(vs numeric[])
        RETURNS numeric[]
        as $$
          vs = args[0]
          sortedvs = sorted(vs)
          from scikits import bootstrap
          return bootstrap.ci(sortedvs, alpha=0.05)
        $$ language plpythonu;

        DROP AGGREGATE IF EXISTS ci(numeric);
        CREATE AGGREGATE ci (numeric) (
          SFUNC = array_append,
          STYPE = numeric[],
          initcond = '{}',
          FINALFUNC = ci_final
        );


        -- an example query

        SELECT ci(measure)[0] AS lower_bound,
              ci(measure)[1] AS upper_bound
        FROM dataset
