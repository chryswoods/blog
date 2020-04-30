==================
Return of the King
==================

This is part three of a `three part post <https://chryswoods.github.io/blog>`__
detailing my RSE work on the `MetaWards project <https://metawards.github.io>`__.
In `part one <https://chryswoods.github.io/blog/fellowship>`__
I talked about trust in software and data, and why my first
step was to port the `original code <https://github.com/ldanon/MetaWards>`__
from C to Python.

In `part two <https://chryswoods.github.io/blog/two_towers>`__
I introduced the two challenges of research software - robustness
and speed, and how I overcame these by walking back to C.

In this part I am going to talk about the most important aspect of
research software engineering, which distinguishes it from normal
software engineering. Namely, flexibility. Research software, by
its nature, must be highly flexible and adaptable. And, most
importantly, adaptable by the users, who are often not trained
software engineers.

Climbing Minas Anor
-------------------

Research software is different to normal software for three reasons;

1. We don't know the requirements of the software when it is written.
   There is a vague idea of what is required. But research, by it's nature,
   is an exploration of the unknown. The software thus has to adapt and
   evolve throughout that exploration. Adaptability and flexibility
   are thus key.

2. The researchers, who are the users of the software, are often also
   the software developers. They have little or no training in coding,
   let alone software engineering. Thus research software has to be
   adaptable and flexible for people with little coding knowledge.

3. Research software can live for **decades**. And what's worse, we often
   don't know which software is going to be useful in 10 years time.
   Often long-lived codes start as quickly written scripts of short
   C or Fortran programs. The original developer would never believe that
   the code would likely outlive their own research careers, and certainly
   didn't write it with that expectation.

This need for flexibility, coupled with the need for the code to be
understandable by a novice coder, makes good research software extremely
challenging to write. And worse, this need for "understandable flexibility"
is at odds with the other two requirements of robustness and speed.

Why is that? Robust codes necessarily have a limited scope of
applicability - as testing gives robustness, but you can't test
everything. And as flexibility increases, so to do all of the
edge case errors and integration errors. Plus, inexperienced
coders tend to cut corners or introduce bugs.

Similarly, optimised codes are written in complex languages and utilise
the full suite of expert knowledge to maximise flops and mops, often
at the expense of code readability. Optimised codes are extremely
intimidating for novice researcher developers, and are extremely
challenging to adapt and then debug.

This means that research software often turns into a poorly tested,
yet highly optimised monolith, full of the scars and hacks of
generations of PhD students and postdocs who were intimidated by
the sheer bulk of the code and the complexity of the build, test
and deployment systems that emerged from the cruft.

I did not look for it in a Ranger from the North
------------------------------------------------

Good research software engineering balances the three requirements of
flexibility, robustness and speed, while remembering that the primary
audience are user-developers with little coding experience, and the
secondary audience are the generations of researchers who will work
with the softare over the coming decade(s).

In my opinion, the best way to achieve this is to build software as
modular re-usable building blocks. And a really good language for that,
well that's Python.

Why do I say that? Surely other languages are better? C and Fortran
are surely faster? R is better for data, no? Rust is best for bug-free
code? Julia is much more understandable? Javascript has the best
package manager? C++ is simply the most poweful
language (and shotgun) ever invented? So why Python?

Python is not the best at anything, but it is the second best at
everything. And it brings everything together. It is a fantastic
glue that joins lots of things together, while at the same time not being
too proud to steal/copy the best ideas and APIs from other languages.

Python comes with fantastic documentation, testing and packaging
tools. The design of the language forces developers to write clean
code, and a strong adherence to coding standards using tools like
flake8 makes group-developed code more coherent.

Most importantly, the Python community develops and encourages the
development of shared modular libraries. Pandas, scikit-learn, numpy,
geopandas etc. just feed into a culture where researcher-developers
are encouraged to write code that is modular and shared.

The way is shut
---------------

Bringing this back to `metawards <https://metawards.github.io>`__, my
goal this last week was to improve flexibility for today, for the
crucial next few weeks/months, and then, importantly, for the next
few years as this area of research is likely to be extremely important.

I took two approaches to achieve this, which I will detail through
the rest of this blog;

1. I wrote a *really* detailed tutorial, and coupled it with a full
   project website.

2. I tried to anticipate how the code may need to be changed, and then
   used the dynamic typing and functional programming capabilities of
   Python to create a plugin interface that researchers should be able
   to understand.

Crucially, I made sure that the tutorial walked through creating plugins.
I then used the tutorial to communicate my progress and the state of
the code back to Leon. This validated that the tutorial had enough
detail and that the plugin interface was understandable and gave
sufficient flexibility.

-- Documentation - sphinx

Learned more about program by writing a good tutorial. Really important.
Fixed bugs too. Added nicer interfaces as tutorial really focussed my
mind on how to use this program as a user. Added nice-to-haves

Lurgy. How to modify the code to model and then investigate a lockdown.

-- Power of python, iterators, custom inputs, extractors, libraries


