==============
The Two Towers
==============

This is part two of a `three part post <https://chryswoods.github.io/blog>`__
detailing my RSE work on the `MetaWards project <https://metawards.github.io>`__.
In `part one <https://chryswoods.github.io/blog/fellowship>`__
I talked about trust in software and data, and why my first
step was to port the `original code <https://github.com/ldanon/MetaWards>`__
from C to Python.

In this part I am going to discuss how I tackled the two major challenges
of research software; how to make it fast, while also keeping it correct.

Climbing Orthanc
================

The first tower to climb is that of robustness, reproducibility and provenance.
The most important requirement of any piece of research software is that it
is *correct*. But what does *correct* mean? In my opinion, correct codes
are ones which;

1. are **robust** - give the *correct* output for a given input,
2. are **reproducible** - give the same outputs for the same inputs,
3. report **provenance** - provide enough information in the output to
   enable someone else to rerun and reproduce it.

Against some, I have not yet been tested
----------------------------------------

Level one is robustness, which is tackled by testing. While you cannot
test everything, it is
really important to set up an infrastructure and workflow that makes
it easy to add tests as a project develops. To this end, the first thing
I did was set up `pytest <https://docs.pytest.org/en/latest/>`__, and made
sure that this was run regularly. I asked Lester in my team to set up
continuous integration using
`GitHub actions <https://github.com/metawards/MetaWards/actions>`__. He also
kindly added a status badge to the
`main website <https://metawards.github.io>`__ so that it was very clear
when something was broken. I also made sure that we adopted robust
development practices of using ``devel`` as the development branch,
``master`` as the release branch, and using feature branches for day-to-day
development. As this way of working would be new for some developers,
we also wrote a
`detailed developer guide <https://metawards.github.io/MetaWards/development.html>`__
that showed how to use feature branches, how to write tests, how to
use virtual environments, how to document code, together with a general
coding style guide that would make it easier for others to learn MetaWards.

Some things that should not have been forgotten were lost
---------------------------------------------------------

Level two is reproducibility, which is tackled using inputs.
First, you have to look through the
code and find all of the inputs. Once you've located them all, you then
run the code with those inputs, and verify that you always get the
same output.

Hunting for inputs is not as easy as you might think. Yes, there are obvious
inputs, like input files and command line arguments.
But there also hidden inputs that lurk, like goblins, in the shadows of
the code. These goblins include random number
seeds, time of day, number of parallel threads, compile-time constants,
and the size of the variables (floats or doubles). MetaWards had them all!

This became clear when I first tried to run MetaWards.
It's command line looks something like this;

.. code-block:: bash

  ./wards.o 15324 Testing/ncovparams.csv 0 1.0

A check in the code revealed that the first argument (15324) was a random number
seed, the second was the name of the input file, the third was the line
in the input file to read, and the last set a variable called ``UV``.

I ran MetaWards several times using these arguments, and each time, it
came back with a different answers. Something was definitely up...

Full of darkness and danger
---------------------------

A look in the code revealed that this was because the random number seed
was being `adjusted using the current time <https://github.com/ldanon/MetaWards/blob/9aecf26f0ec625398334ae0377d74b840b2b988e/Model/main_29_03.c#L36>`__.
On checking with Leon it turned out he was doing this because he was running
many jobs and was not confident that the bash ``$RANDOM`` variable that he
was using in his job script would have given him good random number seeds.

Because MetaWards is a stochastic program, this method of seeding meant that
it was impossible to reproduce any MetaWards output without knowing
exactly what time the program was run. I immediately
`changed this <https://github.com/metawards/MetaWards/blob/c74bc66f72c7b421f622cf4895f7d6105540b906/original/src/main_RepeatsNcov.c#L35>`__.

Now, every time I ran MetaWards I got the same output. This
`reproducible output <https://github.com/metawards/MetaWards/blob/devel/original/expected_test_output.txt>`__
was then used as the `integration test <https://github.com/metawards/MetaWards/blob/devel/tests/test_integration.py>`__
for my Python code.

To stop me chasing my tail, I double-checked that both C and Python gave
the same sequence of random numbers. I changed both the C and Python
codes to
`print the first five generated random numbers to the screen <https://github.com/metawards/MetaWards/blob/c74bc66f72c7b421f622cf4895f7d6105540b906/src/metawards/_network.py#L279>`__.
This way, I would see immediately if there were any differences
(there were not) and if anything changed.

His gaze pierces cloud, shadow, earth and flesh
-----------------------------------------------

A deeper look at the C code showed that there a significant number of inputs
which weren't specified on the command line. These included
compile time flags to `set the disease being modelled <https://github.com/metawards/MetaWards/blob/c74bc66f72c7b421f622cf4895f7d6105540b906/original/src/globals.h#L9>`__
or to switch on or off control measures, such as
`self isolation <https://github.com/metawards/MetaWards/blob/c74bc66f72c7b421f622cf4895f7d6105540b906/original/src/globals.h#L12>`__
or whether the model `treated weekends differently <https://github.com/metawards/MetaWards/blob/c74bc66f72c7b421f622cf4895f7d6105540b906/original/src/globals.h#L14>`__.

There were also definitions of disease parameters
`specified in code <https://github.com/metawards/MetaWards/blob/c74bc66f72c7b421f622cf4895f7d6105540b906/original/src/wards_lib.c#L1579>`__,
as well as lots of `hard coded parameters <https://github.com/ldanon/MetaWards/blob/9aecf26f0ec625398334ae0377d74b840b2b988e/Model/wards_lib.c#L1652>`__.

The program also relied on reading data from extra files that were bundled
with the repository, and it looked for those files in a
`hard coded directory <https://github.com/ldanon/MetaWards/blob/9aecf26f0ec625398334ae0377d74b840b2b988e/Model/wards_lib.c#L1777>`__
(e.g. ``$HOME/GitHub/MetaWards/2011Data``).

This reveals something that I find generally true when I read other people's
code - how it is written gives me a window into how the developer works and how
they think. In this case, the code is written to work well on the developers
own computer only. They make changes to inputs in the source code,
compile a new MetaWards binary,
and then run that to get an output. There was a little bit of tracking of
versions by saving ``main`` functions into
`files with different names <https://github.com/ldanon/MetaWards/tree/master/Model>`__,
but it is clear that the code was being changed, recompiled and re-run, with
inputs, executables and outputs likely becoming jumbled together.
Maintaining reproducibility with such a workflow may be challenging.

It is hardly possible to separate you
-------------------------------------

To encourage adoption of a reproducible workflow my next step was to
separate all data inputs into a `different GitHub repository <https://github.com/metawards/MetaWardsData>`__.
I divided the inputs into three classes, depending on whether they were
used to describe a `disease <https://github.com/metawards/MetaWardsData/blob/master/diseases/ncov.json>`__,
a `model <https://github.com/metawards/MetaWardsData/tree/master/model_data/2011Data>`__
or `simulation parameters <https://github.com/metawards/MetaWardsData/blob/master/parameters/march29.json>`__.

I used JSON as the format for any new files I created, as this was both
very easy for a human to read and edit, and for a computer to load. I then
changed my Python code to use these inputs, and verified that my integration
test still passed. I then removed the hard-coded paths, and instead supplied
either a clear command line option or an environment variable to enable the
user to specify the location of their MetaWardsData clone.
Once this worked, I discussed this with Leon and we agreed
to make similar changes to the C code (which Chris in my team quickly did).

We want a map, not a Gollum
---------------------------

With the second level of reproducibility climbed, the last level was to
embed provenance. To understand why provenance is important, it is useful
to consider the hobbits' journey and the role of Gollum.

Gollum is an interesting character because the hobbits needed him to travel
with them, despite the danger he poses, and the huge personal cost he
suffers.

Gollum was needed as he had been to Mordor before, and so could use
his memory of that journey to act as a guide. If he'd kept a detailed map, then the
hobbits could have simply used that to recreate Gollum's steps, and much
misadventure and perilous encouters with spiders would have been avoided.

In much the same way, the path to reproducibility is rocky if we have
to enlist the original researcher to manually
guide us through the process of recreating their outputs. It costs the
researcher time that they likely don't have, plus risks adventure if their
memory is imperfect, or the terrain (operating system, libraries or
software) has changed since their last visit.

The solution is that the software itself must create a map of how it was
used. This map must be automatically generated, and contain
sufficient detail to guide others in the future.

Original or Extended Edition?
-----------------------------

Embedding `provenance <https://en.wikipedia.org/wiki/Provenance>`__ of
every input and piece of software used in a calculation is a way to
start the automatic generation of such a map. To this end, Lester in my team
used `versioneer <https://github.com/warner/python-versioneer>`__ to
automatically add version tags to MetaWards Python. We also had them added
to the files in the
`MetaWardsData <https://github.com/metawards/MetaWardsData/blob/master/version>`__
directory. We made sure that these versions,
together with all inputs and random number seeds, were written to the outputs
produced by MetaWards, for example::


              ***********************
              metawards version 0.6.0
              ***********************

              -- Source information --
  repository: https://github.com/metawards/MetaWards
                   branch: devel
  revision: fc15e6280c288f6e445780bf04f2feb67de384e2
      last modified: 2020-04-09T21:40:51+0100

is printed at the top of every run. Similarly these versions (and values) for
parameter and disease data were also printed, e.g.::

  Using disease
  Disease ncov
  loaded from /Users/chris/GitHub/MetaWardsData/diseases/ncov.json
  repository: https://github.com/metawards/MetaWardsData
  repository_branch: master
  repository_version: 0.2.0

  beta = [0.0, 0.0, 0.95, 0.95, 0.0]
  progress = [1.0, 0.1923, 0.909091, 0.909091, 0.0]
  too_ill_to_move = [0.0, 0.0, 0.0, 0.0, 0.0]
  contrib_foi = [1.0, 1.0, 1.0, 1.0, 0.0]

This meant that someone reading the outputs would know exactly which versions
of the code and data we used. We even added in a check to see if the
code had not been committed to Git. This let us alert the user that
the code was ``dirty``, and that
`any outputs may not be reproducible <https://metawards.github.io/MetaWards/usage.html#metawards-program>`__.

Climbing Barad-dûr
==================

Now that the Python and C codes were reproducible and in-agreement, the next
tower to climb was to fix the
`mûmakil <https://lotr.fandom.com/wiki/Mûmakil>`__
in the room, namely that the Python code was horribly slow.

Level one was that I needed something magical that would quickly speed
up the Python. The bulk
of the code was iterating through the ``Node`` data structures that represented
an electoral ward, and the ``Link`` structs that connected them. I briefly
considered using an existing graph library, but put that on hold as it would
have taken too long (weeks+) to completely refactor. What I thought would be
a quick win would be refactoring the Python list of ``Node`` and list of
``Link`` objects into a ``Nodes`` and ``Links`` structs of lists (e.g.
`an array of structs into a struct of arrays <https://en.wikipedia.org/wiki/AoS_and_SoA>`__
transformation). This would allow me to switch to iterating over numpy arrays.

I quickly made the change using `test_integration.py <https://github.com/metawards/MetaWards/blob/devel/tests/test_integration.py>`__
as a guardrail. Feeling duly proud of myself I ran the test and the code was
**four times slower!** I switched the numpy arrays for standard
`Python arrays <https://docs.python.org/3/library/array.html>`__ and now the
code was just *two times slower*. How could Python lists be quicker?

I checked the memory usage and Python lists were quicker, but consumed
1.5GB compared to the ~95MB using numpy or arrays. I couldn't stick with
lists because the memory consumption would have been awful, so I double-checked
what was going on with numpy and arrays. Another profile showed that arrays
were two times slower at indexing for reading and writing compared to lists,
while numpy was two times slower for reading, and four times slower for
writing. A bit of searching on the internet revealed the obvious reason -
arrays and numpy are not designed for random access. They have to do lot of
boilerplate to verify the index is correct, then convert the raw ``float`` or
``int`` into a Python ``float`` or ``int`` object.

I need eagles
-------------

At this point I realised that I needed something to come to the rescue.
In short, I needed to find a way to compile the code that performed
random array access so that I could avoid the costs of moving
between raw numbers and Python objects. I tried `numba <https://numba.pydata.org>`__,
but it didn't work. The code was still painfully slow. I needed something
more powerful. I needed `cython <https://cython.org>`__.

Now is the hour! Riders of Rohan!
---------------------------------

Cython is a fantastic mix of C and Python that I have now grown to love
(it is my current `hammer <https://en.wikipedia.org/wiki/Law_of_the_instrument>`__).
You write ``.pyx`` files that merge Python and C. These are compiled to
C, from where they are then compiled into machine code. The basic idea
is that you use ``cdef`` statements to type your objects. For example,
this Python code...

.. code-block:: python

  a = 0.5
  b = 10.2
  total = 0.0

  for i in range(0, 10);
      total += a * b

becomes this Cython code

.. code-block:: cython

  cdef int i = 0

  cdef double a = 0.5
  cdef double b = 10.2
  cdef double total = 0.0

  for i in range(0, 10):
      total += a * b

(I know that this is a pretty silly example - take a look at the excellent
`cython documentation <https://cython.readthedocs.io/en/latest/>`__ and
`basic tutorial <https://cython.readthedocs.io/en/latest/src/tutorial/cython_tutorial.html>`__
for some more real-world examples)

There is a `great template package <https://github.com/FedericoStra/cython-package-example>`__
that makes it easy to write a Python ``setup.py`` for a Cython project,
which I used and have adapted. After some quick tests that showed that
Cython would really work, I then set about the process of Cythonizing
my code. It was a lot of fun :-)

I know what hunts you
---------------------

One of the reasons I enjoyed cythonizing was because I really enjoy the
chase of optimising code. Software is slow for unexpected and oftentimes
invisible reasons. You have to use profiling to actually find where a code
is slow, and then use your good understanding of how computers work to
fix the bottleneck and make it quicker. It is the ultimate
experiment-hypothesis-implement-test design cycle, but repeated dozens of
times as you move through each bottleneck. It is the best crossword
puzzle mixed with the best scientific experiment, with enough dark art
and magic to make it a creative as well as an intellectual endeavour.

Eyes always watching
--------------------

I like profilers and use them a lot. But they are blunt tools which
disrupt the workflow of running a program. I much prefer in-code
instrumenting, whereby a simple profiler is embedded in code and
switched on or off with a command line argument.

Simple profilers are very easy to write. They are just calls to
a timing function either side of the code you want to profile.
The difference between your two timing calls is the time it
took to run your code. Timers on computers now have nanosecond
resolutions, and obtaining the time is essentially free, meaning
that profiling adds no overhead to your code runtime. This means
that you can leave profiling switched on, and thus always
be able to know how fast your code is running, regardless of which
machine it is running on. Indeed, (eventually!) I will write some
performance regression tests to add to the suite that will
use the output of profiling to make sure that I don't
inadvertently slow down the code.

To make embedded profiling easier to use, I created a very simple
`Profiler <https://github.com/metawards/MetaWards/blob/devel/src/metawards/utils/_profiler.py>`__
class that recursively measured times of instrumented blocks, e.g.

.. code-block:: python

  p = Profiler()

  p = p.start("loop")

  for i in range(0, 10):
      p = p.start(f"iteration_{i}")

      # do stuff

      p = p.stop()

  p = p.stop()

  print(p)

This gave fantastically useful output that guided my cythonizing, e.g.::

   157 406
  S: 55833958    E: 215    I: 126    R: 247778    IW: 11   TOTAL POPULATION 56081862

  Total time: 121.805 ms (121.805 ms)
    \-timing for iteration 157: 121.805 ms (121.723 ms)
        \-additional_seeds: 0.008 ms
        \-iterate: 86.709 ms (86.687 ms)
            \-iterate: 86.687 ms (86.586 ms)
                \-setup: 0.035 ms
                \-loop_over_classes: 11.721 ms (11.310 ms)
                    \-work_0: 2.565 ms
                    \-play_0: 0.044 ms
                    \-work_1: 2.573 ms
                    \-play_1: 0.665 ms
                    \-work_2: 2.426 ms
                    \-play_2: 0.226 ms
                    \-work_3: 2.562 ms
                    \-play_3: 0.249 ms
                \-recovery: 19.200 ms
                \-fixed: 21.143 ms
                \-play: 34.487 ms
        \-extract_data: 17.452 ms (17.434 ms)
            \-extract_data: 17.434 ms (17.354 ms)
                \-loop_over_classes: 17.291 ms
                \-write_to_files: 0.063 ms
        \-extract_data_for_graphics: 17.554 ms (17.542 ms)
            \-extract_data_for_graphics: 17.542 ms (17.515 ms)
                \-loop_over_n_inf_classes: 17.515 ms

All we have to decide is what to do with the time that is given us
------------------------------------------------------------------

Now I knew where the code was slow, level two of the tower was to
decode on how I could make it faster. Optimising
Python code using cython is a whole blog post in itself as
there are a lot of tips and tricks that I learned. The main advice
I'd give is to disable all of the indexing and other cleverness
that cython uses to make C behave like Python. Do this by adding
the following to the top of your ``.pyx`` files;::

  #cython: boundscheck=False
  #cython: cdivision=True
  #cython: initializedcheck=False
  #cython: cdivision_warnings=False
  #cython: wraparound=False
  #cython: binding=False
  #cython: initializedcheck=False
  #cython: nonecheck=False
  #cython: overflowcheck=False

.. note::

  Update: I now add these checks to my
  `setup.py script <https://github.com/metawards/MetaWards/blob/035e9eacaba23b18ddce6788fe9222e04147fe33/setup.py#L196>`__

While clever, these cython checks add in extra functions around your code
that prevent vectorisation, increase indirect memory lookup, and just
add unnecessary bloat if you already trust that your Python code is
looping through arrays correctly.

Another piece of advice is to make sure you ``cdef`` everything that is
in a loop. Even forgetting a few variables
`has a dramatic impact on performance <https://github.com/metawards/MetaWards/commit/8487eb7d94cdfc3f362432ea2b4efca8abe23a15>`__.

The other advice I have is to not use memory views or other abstractions,
but to instead just use raw C pointers for contiguous arrays. When I originally
used memory views like this;::

  cdef double [:] a = python_array

the code was really slow. This was because every index lookup involved
cython trying to work out how to convert index ``i`` into the index
in ``python_array`` as cython didn't know that the array was contiguous.
Changing this to;::

  cdef double [::1] a = python_array

gave a big speed up, as now cython knew that this was a one-dimensional
contiguous array. However, when looking at the C code generated, it was
clear that cython was still adding in weird pointer redirections and
type conversions that were confusing the compiler. In the end, I switched
to;::

  cdef double * get_double_array_ptr(double_array):
      """Return the raw C pointer to the passed double array which was
         created using create_double_array
      """
      cdef double [::1] a = double_array
      return &(a[0])

  cdef double *a = get_double_array_ptr(python_array)

Cython now saw the array as just a double pointer, and so the resulting
C code was exactly if I had written it manually.

Where many paths and errands meet. And whither then? I cannot say
-----------------------------------------------------------------

With the code optimised, and now faster than the C, the third and final
level was to parallelise the code. MetaWards is a single-threaded C code,
but inspection of the loops within them suggested that they should
be parallelisable.

The great thing about cython is that is trivial to use
`OpenMP <https://openmp.org>`__ to
parallelise code. Again, how to do this and how to optimise
parallel code in cython is worth another blog post (or, indeed workshop),
but in a few days a serial code that took minutes was now a code that
scaled over 32-64 cores and took seconds. Indeed, OpenMP was so good that
loops that took 10s-100s of milliseconds were sped up to only take 3-5ms.
It is unlikely, given parallel overhead, that I could make them much faster.

Key pointers for performance are to drop the
`gil <https://wiki.python.org/moin/GlobalInterpreterLock>`__ (use ``nogil``),
double-check you've *dropped the
gil*, and then take a look at the C code to triple-check that there isn't
anything that you or cython have implicitly added that causes
something to be generated in the C code that **takes the #!!#!! gil!**

.. note::

   Update: Since writing I've learned that it is easier to put performance
   code into a ``with nogil:`` section. This causes a easily-readable
   compiler error if I accidentally take the gil, and then add a
   ``with gil:`` when I really want the gil.

Also, any variables you assign to are automatically ``lastprivate``,
which means that this doesn't do what you expect;

.. code-block:: cython

  cdef int i
  cdef broken = 0

  with nogil, parallel():
      for i in prange(0, 10):
          if i > 10:
              broken = 1
              break

  if broken:
      print("This will be printed as 'broken' has become undefined")

This means that if you want to save a value, then you have to put it into
an array. Also, increment operators, like ``val += something``, are automatically
converted into OpenMP reductions, which is wonderful except they are not
supported when you use ``parallel()``. Hence I had lots of code to convert
back into ``val = val + something``.

Welcome, my lords, to Isengard
==============================

So two towers climbed, surely I have now finished? Unfortunatley, not yet.

Remember I said that there were hidden inputs to programs, like floating
point precision and number of threads. Well, the journey to optimise the
Python code exposed these inputs are sources of reproducibility error.

The first error was when I was optimising the code. Before cythonising,
the Python version was so slow that my integration test only compared outputs
up to the 20th day of the simulated outbreak. After half-cythonising, I was
able to compare a full outbreak and saw that the results were really different.

Line-by-line comparison showed that a few extra simulated agents were
infected on day 28 in the C code who were not infected in the Python code.
This propogated over the outbreak to give a very different result.

To understand why this was the case I added an *if* statement to both
the C and Python codes to print the values of all variables on day 27
and 28. From this it was clear that one value was very slightly different.
This made me realise that I had a precision error. I had mistakenly
changed

.. code-block:: python

  float(something) / float(something_else)

into

.. code-block:: cython

  <float>(something) / <float>(something_else)

when cythonizing. This was an error as  ``float`` in Python is
a ``double``, but ``float`` in cython is a ``float``.

I quickly changed all ``<float>`` statements into ``<double>``, amended the test
to run more days, and then `everything agreed <https://github.com/metawards/MetaWards/commit/f25f6091a491a4e66840df565733c6b7ec4abb8a#diff-7eb47fc18d4a6196f0940784484a5bf3>`__.

March of the Ents
-----------------

But this was not all... Once the code was parallelised I found another
reproducibility problem. I had made the random number generator thread-safe
by giving each thread its own generator, which was deterministically seeded
based on the main random number generator seeded by the user. I had also
been very careful to use techniques that ensured a deterministic order of
operation of calculations, plus order of assignment of work to threads
(e.g. using ``schedule="static"``).
This meant that, in theory, MetaWards would give the same answer for the
same input, when using the same  random number seed,
and running on same number of threads.

However, when I ported MetaWards onto
`Catalyst <https://www.bristol.ac.uk/news/2018/april/supercomputer-collaboration.html>`__,
one of the University's large ARM64 supercomputers, I was able to run
large numbers of parallel
jobs and quickly saw that a small percentage of jobs were giving different
outputs.

Again, I saw that the changes occured at a specific day of the simulated
outbreak. Suspecting this was a parallelisation problem I systematically
disabled parallelisation for different parts of the code by changing

.. code-block:: cython

  with nogil, parallel(num_threads=num_threads):

to

.. code-block:: cython

  with nogil, parallel(num_threads=1):

This showed that the difference was occuring on a specific day within
`one of the parallel loops <https://github.com/metawards/MetaWards/blob/c74bc66f72c7b421f622cf4895f7d6105540b906/src/metawards/utils/_iterate.pyx#L255>`__.

The problem was that the ``add_to_buffer`` function I was using to manage
the large number of reductions in the loop was adding the buffer contents
when each thread had filled it up. As different threads
filled the buffer at different times, this meant that the order of addition
was different for different runs. This was not a problem for most reductions,
as MetaWards is predominantly an integer code. But the variables that
hold the *force of infection (foi)* are doubles, and so the order of these
reductions did matter. While for >95% of runs the order was the same, there
were a few runs where the difference caused small numerical differences
that caused one or two more agents to become infected. These small differences
then propogated through the model outbreak to create different results.

For now, I've had to leave this section of the code in serial, and so
sacrifice a little speed for reproducibility. I've worked out how to
do the reduction safely and reproducibly, and plan to fix this problem
later this week.

.. note::

   This is now fixed and the function runs in parallel.


What's next?
============

Well done for getting this far. I said at the beginning that this blog
was going to be a long read. So now we have walked from a C code
to a Python code, and then via cython we have walked MetaWards back
to being a very C-like code. But it has now been transformed into
something that is much faster, more modular,
more reproducible, more robust and more usable. In the
`next (and final) part <../return_of_the_king>`__,
I will show why I have worked so hard to translate MetaWards to Python.
It is the King of Languages, and its power is now enabling me to
turn MetaWards into an adaptable *Minas Tirith* that will cope with anything
that the next few weeks could bring.
