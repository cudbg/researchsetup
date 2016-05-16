A lot of life is about regret minimization.  Here are some tips to not be too sad in the future.




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


# Papers

Managing your papers

* David Blei's lab: https://github.com/blei-lab/publications
* I like to keep one repo per project, but makes releasing code more of a pain:

        SUPERPROJECT/
          docs/
            sigmod_16/
            vldb_16/
            README.md            -- all project notes
          src/
            allmycode/


# Reviewing Papers


        tldr; don't be a dick.  

Fill in the following form, with an emphasis on the **positive** aspects of the paper:

      # Positives 

      ## What is the key idea in this paper?  Why is it a good idea?

      ## What are other reasons why this could be a good idea that are not mentioned in the text?  Suggest them.

      ## Are there other applications where you could use these ideas?  Describe them

      ## What else did you like about this paper?

      # Improvements

      ## What writing improvements could have made it easier to understand the paper?

      ## What techniques would make the ideas _even better_?  In what ways

      ## What experiments/plots would help you better understand how well the ideas work?  Are they _possible_? (remember: getting private data is hard)

         


What other people say about reviewing:

* [https://github.com/jtleek/reviews](https://github.com/jtleek/reviews)


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
