A lot of life is about regret minimization.  Here are some tips to not be too sad in the future.

Index

* [Asking Questions](#questions)
* [Deployments](#deployments)
* [Experiments](#experiments)
* [Papers](#papers)
* [Jobs](#jobs)
* [Appendix](#appendix)

# What is Research?

tbd

# Questions to Answer When Picking Problems

All of the following questions eventually need to be answered in a publication, and ideally before fully jumping into a project.
Often you want to run small experiments in order to answer these questions.

All credit goes to remcochang@tufts

* In once sentence, what new thing do you want to show/achieve?  

      _e.g., by using my threading library, systems run faster_

* Is this an application, a simulation, a prototype, etc?  _application/library_
* Who are the users whose lives would be improved by the research?  
  _people that user multi-threaded libraries.  Developers to a lesser extent_
  * What causes their frustrations in the current state of the art?   _programs are too slow.  Some take 10 whole seconds to run!_
  * If your project succeeds, how does that change their lives?  _programs will be fast.  completely changes how people interact with programs_
  * If your project is not done, what happens?  _lives are wasted waiting for slow programs_
* If the project works out in the *best case*, what results does that entail?  _multi-threaded programs run 30x faster_
* What steps are needed to make this happen?  
  _identify key bottlenecks in existing programs.  use technique XXX to remove the bottlenecks._
  _Write a library.  Modify existing programs to use library and run experiments_
* What risks could derail the project?  
  * **Major**: would kill the project: _technique XXX doesn't work_ 
  * **Minor**: slow the project down:  _programs are super complicated.  takes longer to switch to my library_
* Top 5 papers related to this project (previous attempts, alternatives, assumptions, etc)
  1. _library Y[1] proposed technique Y to run programs faster.  However they don't work for cases A, B, C which are important cases that affect the users described above._
  2. _project W[2] say that multi-threading is not needed if developers use a new chip.  However the chip doesn't actually exist yet_
  3. _developers make their programs faster by using tricks D, E, F.[3,4,5]  Turns out they are special cases of our library_
  4. _projects L, K [6,7] have used variations of our technique XXX in other domains and look, they worked.  However, they are different in these ways_
* How do you know if the project succeeded?  _the mail, webserver and database programs run at least 2x faster when using our library_
  * What do you need to be measuring (aka the x and y axes of a graph)  _latency, development costs_
  * How much do these metrics need to change before "success"?



# Asking for help

Asking for help is important and getting feedback often is important.    Better to be embarrassed early on than when you try to publish!
From [AMA with Google Brain team](https://www.reddit.com/r/MachineLearning/comments/4w6tsv/ama_we_are_the_google_brain_team_wed_love_to/d6diast)

        I often tell new team members about the 15 min rule (I didn't come up with it): 
        when you're stuck on something (e.g. getting a script to run), you have to try to 
        solve the problem all by yourself for 15 min, but then when the 15 minutes are up 
        you have to ask for help. Failure to do the former wastes people's time, 
        failure to ask for help wastes your time.

        (the value 15 may vary)

When asking questions about code, hit the following points:

1. what piece of code 
2. what you think it currently does and why, 
3. what you think needs to be changed to get the desired behavior 
4. what went wrong in terms of how the problem manifests itself and 
5. what you think the issue is
6. Remember, the helper may not have dealt with your specific instance of this problem



Other resources

* http://stackoverflow.com/help/how-to-ask


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


## Releasing Code

Releasing working code is a very good idea.  If you are writing a python application, make it `pip` installable.  The following repo provides a skeleton and simple example code for structuring a python package so it can be uploaded to pypi.

* [https://github.com/cudbg/pipexample](https://github.com/cudbg/pipexample)



## Websites

It is important to create and maintain a website of your project -- this is the primary way the world will learn about what you are doing!

* Create a github organization for your project.  Let's say it's called `yourproject`
* Create repo called `yourproject.github.io` in your organization.  
* Copy over and edit the [example website scaffolding](./src/website_scaffold/) where all you need to do is edit markdown documents and run a build script to create your site pages
* See [github's pages instructions](https://pages.github.com/) for more details


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


## Setup

Managing your papers

* I like to keep one repo per project, but makes releasing code more of a pain:

        SUPERPROJECT/
          docs/
            sigmod_16/
            vldb_16/
            README.md            -- all project notes
          src/
            allmycode/

* See [/docs/example16](./docs/example16)
* David Blei's lab: https://github.com/blei-lab/publications

## Writing papers

It helps to see how other papers evolve.  I've built a tool called `latexsnapshots` that will generate snapshots of a paper as it evolves.  

* [See how my VLDB 2014 paper evolved.](http://eugenewu.net/latexsnapshots/html/)

Responding to reviewer comments

* [A sarcastic guide](http://errantscience.com/blog/2016/10/19/how-to-reply-to-reviewers-a-terrible-guide-that-no-one-should-follow/)

## Reviewing Papers


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
