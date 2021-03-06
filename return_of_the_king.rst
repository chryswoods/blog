==================
Return of the King
==================

This is part three of a `three part post <https://chryswoods.github.io/blog>`__
detailing my RSE work on the `MetaWards project <https://metawards.org>`__.
In `part one <https://chryswoods.github.io/blog/fellowship>`__
I talked about trust in software and data, and why my first
step was to port the `original code <https://github.com/ldanon/MetaWards>`__
from C to Python.

In `part two <https://chryswoods.github.io/blog/two_towers>`__
I introduced the two challenges of research software - robustness
and speed, and how I overcame these by walking back to C.

In this part three I will cover a lot of topics, as the last six weeks
have been very eventful. This is a very long post, so, in advance,
thank you if you reach the end.

I will cover both RSE-type topics, such as adding flexibility to code,
"tutorial-driven development", and "plugin-based design", before exploring
more general topics such as the role
of the RSE, my opinions on the Covid-Sim events, my opinion on the
behaviour of some connected to government, and finally, why I believe
a second wave is inevitable and what we as a culture can do to protect
ourselves.

Part 1: RSE-type topics
=======================

Climbing Minas Anor
-------------------

There is a big difference between normal software and research software.
This is why research software looks different to normal software, and
why what can be "good software" in academia can look like "bad software"
to a "professional software engineer".

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
   C or Fortran programs. Most developers would never believe that
   their code will outlive their own research careers, and certainly
   don't write it with that expectation.

This need for flexibility, coupled with the need for the code to be
understandable by a novice coder, makes good research software extremely
challenging to write. And worse, this need for "understandable flexibility"
is at odds with the other two requirements of robustness and speed.

But why is that? Why does flexibility make codes slower and less robust?
Robust codes can only have a limited scope of
applicability. Testing gives robustness, but you can't test
everything. And as flexibility increases, so to do all of the
edge case errors and integration errors. Plus, inexperienced
coders tend to cut corners or introduce bugs.

Similarly, optimised codes are written in complex languages and utilise
the full suite of expert knowledge to maximise FLOPs and MOPs, often
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
collection of modular and re-usable building blocks.
And what's a really good language for that - well that's Python.

Why do I say that? Surely other languages are better? C and Fortran
are surely faster? R is better for data, no? Rust is best for bug-free
code? Julia is much more understandable? Javascript has the best
package manager? C++ is simply the most poweful
language (and shotgun) ever invented? So why Python?

Python is not the best at anything, but it is the "second best at
everything". And it brings everything together. It is a fantastic
glue that joins lots of things together, while at the same time not being
too proud to steal/copy the best ideas and APIs from other languages.

Python comes with fantastic documentation, testing and packaging
tools. The design of the language forces developers to write clean
code, and a strong adherence to coding standards using tools like
`flake8 <https://flake8.pycqa.org/en/latest/>`__,
`black <https://black.readthedocs.io/en/stable/>`__ and
`autopep8 <https://pypi.org/project/autopep8/>`__ makes
group-developed code more coherent.

Most importantly, the Python community develops and encourages the
development of shared modular libraries. Pandas, scikit-learn, numpy,
geopandas etc. just feed into a culture where researcher-developers
are encouraged to write code that is modular and sharable. And because
it is a scripting language, distributing the source (often via highly
permissive open source licenses) is the default.

It's the jobs that's never started that takes longest to finish
---------------------------------------------------------------

Bringing this back to `MetaWards <https://metawards.org>`__, once I had
finished and optimised the Python port, my next goal was to add
flexibility to the code. This was not just so that the code could adapt
to the demands for the next few weeks/months, but more importantly, that the
code will be adaptable for the next
few years as this area of research is likely to remain extremely important.

I took two approaches to achieve this, which I will detail below;

1. I adopted the process of "tutorial-driven development". This involved
   writing a detailed tutorial for each new feature before or as it
   was developed.

2. I tried to anticipate how the code may need to be changed, and then
   used the dynamic typing and functional programming capabilities of
   Python to create a plugin interface that researchers should be able
   to understand.

Tutorial-driven development
---------------------------

Tutorial-driven development is a process we've adopted at Bristol that has
grown out from our experiences from the
`BioSimSpace project <https://biosimspace.org>`__.
We realised during that project that it was a waste of our time to add
functionality that users didn't know existed, nor knew how to use. We thus
took to writing Jupyter-notebook-based workshops that demonstrated
key functionality. Tutorial-driven development was born as the development
of workshop material began to overlap with, and then precede actual code
development.

In MetaWards I had completed the Python port after ~2 weeks so that it was a
drop-in replacement for the C code. I then had the pressing need to add
new functionality to support the ideas being discussed by the epidemiologists
in their Slack group (more on that later). I needed to not just add new
functionality, but also teach people how to use it. Because a lot of the
flexibility came from a Python plugin interface, this also meant teaching
users how to use the MetaWards API to write plugins.

So I started to write tutorials that were included in the code as rst
files in the `doc/source/tutorial <https://github.com/metawards/MetaWards/tree/devel/doc/source/tutorial>`__
directory. They started as tutorials for the code as it was, and to be honest,
were more forcing me to learn enough epidemiology so that I could explain
what the code was doing to others. However, they quickly evolved to
describing functionality as I was writing it, to writing tutorials first
showing how the code *should* work, and how it *should* behave. This meant
that the tutorial inched forward in lock-step with the code releases.
Each new feature had an associated new tutorial, and the process of
writing the tutorial had forced me to adopt the role of the user and the
teacher. As a result, the code was deliberately written to make
functionality easy to use and easy to teach. As an added benefit, the
tutorials, by design, had almost full coverage of the code. Users who
followed the tutorials on their own machines were a great distributed
human continuous integration service. The errors they encountered and bugs
they surfaced were reported quickly, and in a way that I could easily
reproduce (we were, of course, all working from the same input files and
tutorial files). The strong Windows support for MetaWards is entirely
because one of our users works on Windows, and was very diligent and helpful
in raising issues for every problem they encountered. I sincerely thank
that user :-). (also, hint to everyone, add
`GitHub issue templates <https://help.github.com/en/github/building-a-strong-community/configuring-issue-templates-for-your-repository>`__
as they nudge people to submit useful issues, and keep out most of the spam
and trolls).

One issue with tutorial-driven development was that the tutorials were
always changing. I constantly refined tutorials based on suggestions from
the users, e.g. to add functionality that would make the code
easier to understand, or to make it easier for users to write plugins.
This was yet another benefit of tutorial-driven development - users made really
useful suggestions, as we were all speaking the language of the user
actually using the code!

To prevent confusing people, I had to version the
`https://metawards.org <https://metawards.org>`__ website. I did this by
adding a
`few small scripts <https://github.com/metawards/MetaWards/tree/devel/actions>`__
to the GitHub Actions pipeline that copied the Sphinx-generated docs into
a ``versions`` subdirectory depending on the Git version tag. I then ran
a `deduplication script <https://github.com/metawards/MetaWards/blob/devel/actions/deduplicate_website.py>`__
to remove duplicated website assets that were larger than 256KB, and then
a small bit of javascript so that a combo-box below the logo on the website
could access any version of a page.

.. image:: /images/versions.png
  :width: 80%
  :align: center
  :alt: Image showing the version combo box

Now users could follow the version of the tutorial that matched the
software. And we have a record of how the code has evolved and developed.
Reproducibility, provenance, user-centered design, strong testing and
distributed human CI. Tutorial-driven development forces adoption of a
workflow that brings huge benefits to research code.

It also solves the question that I asked Neil Chue-Hong when I first met
him as part of a `SSI project <https://software.ac.uk/resources/case-studies/getting-grips-molecules>`__
back in 2010: How do we document code for
users, and keep that up to date? At that time I had been working on
`ProtoMS <https://protoms.org>`__ and `Sire <https://siremol.org>`__, both
of which had good source-level documentation, and in ProtoMS's case,
a detailed user manual. However, writing user-level documentation and keeping
it in sync with code as it developed was really difficult. I rarely found
the time to write user-level docs, and when I did, they quickly became
outdated as I added new functionality or changed the design of the code.

Tutorial-driven development makes the user-level docs **the central output and focus of the software development process**. The code has to
stay in sync with the docs, not the other way around. New functionality does not
exist unless it is in the tutorial. Developers must put themselves in the shoes
of their users, and adopt the mindset of a teacher who has to explain how
this new functionality will work.

Now you may think that this would dramatically slow down the code writing
process. I hope that MetaWards, which in just two months has hit version
1.0 and been used
in production as part of `extensive studies <https://uq4covid.github.io>`__
proves otherwise. Much to my surprise, tutorial-driven development made
me faster and more productive. Writing the tutorial first gave me the time
and head space to properly design new functionality before writing it.
It prevented me from going down many blind alleys and rabbit holes.
The most valuable
outcome was that it made me actually *use the code* to run epidemiological
experiments.

For these, I created `the lurgy <https://en.wiktionary.org/wiki/lurgy>`__,
so that I could model scenarios in public without the risk of causing
panic or misinterpretation. As a user, I looked at MetaWards and asked
the question of how I would use this to, e.g. model different lockdown
scenarios. Or to model shielding, quarantines, hospital admissions etc.
I had reached a level of trust with the epidemiologists that they had
invited me into their Slack group. I knew what they wanted to model and
what approaches they wanted to take. So, I looked at MetaWards and tried
to work out the easiest and most easily explainable way to model what
they wanted - under obviously very tight timescales.

By writing the tutorial first, I was forced to re-use as much existing
functionality in MetaWards as possible. The best new code is no new code
(it has the fewest bugs). Re-using and re-purposing existing functionality
exposed and fixed edge-case bugs, but also prevented me from writing whole
new functions/classes for conceptually different, but what turned out to be
structurally related functionality.
Code written for, e.g. modelling different age demographics, was quickly
repurposed to represent dynamic and arbitrary groups, to model
shielding and quarantine. And then, as I wrote the tutorials and really
used the code, it gave me a deep level of understanding that enabled me
to make big contextual leaps, e.g. the
`creation of interaction matrices <https://metawards.org/tutorial/part05/03_custom.html>`__,
or using `different isolation groups for different days <https://metawards.org/tutorial/part06/02_duration.html>`__,
such that really
sophisticated features, like `modelling different transmissability of the
virus from carers to shielded versus shielded to carers <https://metawards.org/tutorial/part05/03_custom.html#adjusting-shielding-parameters>`__,
or
`modelling the impact of people breaking quarantine with different probabilities
for different numbers of days <https://metawards.org/tutorial/part06/05_quarantine.html#reflections-on-results>`__,
could all be implemented *in the tutorial*
with *no major changes to the code*!

Tutorial-driven development stopped me from "coding in a hole", and stopped
me writing overly complex or unnecessary functionality that would never
be used. It is the single most productivity-enhancing process I have
followed as a research software engineer, and I strongly encourage its
wider roll-out in the community.

Plugin focussed design
----------------------

Flexibility is paramount for research software. However too much flexibility
makes for untestable or spaghetti like code. Plugin-based designs, which
give users flexibility within small sandpits defined within the code are
a good compromise.

I've fully embraced a plugin-based design for MetaWards. We use
`iterate plugins <https://metawards.org/api/index_MetaWards_iterators.html>`__
to advance the outbreak on each model day. We use
`extract plugins <https://metawards.org/api/index_MetaWards_extractors.html>`__
to control how data is analysed and output to files (or databases etc.)
We use
`mix plugins <https://metawards.org/api/index_MetaWards_mixers.html>`__ to control
how the force of infection calculated for different demographics is merged
together, and
`move plugins <https://metawards.org/api/index_MetaWards_movers.html>`__ to
control how individuals are moved from one demographic to another
(e.g. from home to hospital, or from quarantine to released).

Python is a functional language, so plugins are super-easy to write and
work with. I make liberal use of ``**kwargs`` keyword arguments and
use the tutorials to strongly encourage users to embrace a very
Pythonic way of writing software. Initial discussions with a new user
with a C++ and C# background was illuminating as it really showed that
the object-orientated way of thinking encouraged by those languages is
very constraining. A lot of people who come to Python from C++ end up
writing "C++ in Python", and don't adapt to the freedom and flexibility
that full Python provides. I feel it is very similar to when C developers
first moved to C++ - they just wrote "C in C++" and didn't take
advantage of the new freedom of a much more expressive language. We are
at a similar tipping point now, and a new generation of programmers are
coming in who will truly embrace functional-style development.
Such codes are fast (MetaWards Python is *faster* than MetaWards C), but
are also significantly faster to develop, easier to maintain, easier to test,
and give an expressive freedom that mean that complex and sophisticated
ideas can be implemented intuitively using little code (and remember,
less code means less bugs).

If you are a Fortran, C, or C++-trained RSE,
then please take a week to learn Python from scratch. Put aside your
pre-conceptions of object-orientated programming. Forget inheritance.
Embrace functions as objects, duck typing, keyword arguments and forgiveness
is better than permission. Yes, it will be
different, and you will initially feel lost and confused. But, once you
get it, once you see that this style of programming is very different
to what has come before, you will grow as a developer. And have a new
tool to complement the collection that you should build up through
your career.

To motivate you, the best way I can explain the difference is that writing
C++ is like writing a technical manual - it is fully detailed and
each page tells a different story. In contrast, writing Python is like
writing a haiku. There are a small number of lines and, on first look,
it is deceptively simple. However, there are many layers of meaning. Calling
these lines in different situations with different types of objects
gives ever deeper and more sophisticated behaviour. Each time you read
those lines you can get deeper meaning and greater understanding. And tied
into what I said above about re-using code in the tutorials, those same
lines of code can be repurposed for completely different behaviour, but
are still easily comprehendable for both of those different situations.
Since 2000 C++ was my favorite programming language, with Python gradually
rising up into a solid second place. Writing MetaWards has given me a
deep and new appreciation of Python that has now elevated it clearly
to the status of my favourite programming language. It is the new king.

Part 2 - More general topics
============================

What is the role of the RSE?
----------------------------

In addition to making me reflect on how to program and how to teach,
this project has also made me think deeply about the role of the RSE.

It has taken two months to get to MetaWards 1.0. I adopted a parallel code
strategy, leaving the epidemiologists to work in a (slightly cleaned up)
version of the C code while I spent the first two weeks completing the initial
refactor and testing. They then started moving across to the new code
for speed, but developing new functionality in C. I mirrored that into the
Python code, until we eventually reached a point where they were happy
enough to do everything in Python. So, while it has been 2 months, they
were productively working with the code from about 2 weeks. Most importantly,
they were never slowed down by my work, and the move across to Python,
supported by the tutorials, has been relatively smooth.

RSEs need to be empowered to make these kind of large changes to a codebase.
But to do this, they must make the effort to learn enough about the field
they are working in so that they can fully understand the code and gain
the trust of researchers to make these suggestions. While I was working
on MetaWards I was hearing about what was happening with Microsoft and
Covid-Sim. No disrespect intended, but software engineers who don't
understand the science behind a code are limited to making tiny changes, e.g.
adding integration tests that just compare outputs, or pointing out
duplicated code that could possibly be refactored, or running static
analysis tools to point out potential bugs. These are useful things,
no doubt, but **this is not research software engineering!!!**.
I was disappointed to see the trolling, attacks and general mess that
resulted from the publication of Covid-Sim. And the general hypocrisy
of "professional" software engineers saying that academia has a culture
of mediocrity regarding software, when we can all point to many examples
of truly terrible business software, or ridiculously buggy software
from major commercial software providers (iOS 13 anyone?).

I was also annoyed to read that RSEs merely "bring the tools of industry
into academia". Version control has been used in academia since the days
of VCS and CVS. If you want, you can look through the full history of
my 20+ years of academic software development, from the CVS-to-SVN
of `ProtoMS <https://code.google.com/archive/p/protoms/source/default/source>`__,
through the SVN to Git of `Sire <https://github.com/michellab/Sire>`__,
via `BioSimSpace <https://github.com/michellab/BioSimSpace>`__ and now
to `MetaWards <https://github.com/metawards/MetaWards/>`__. Testing,
documentation, continuous integration and continuous deployment can be
seen to be evolving through all of these codes, reaching a pinnacle
now with MetaWards, which embodies every tool and technique that my
years of experience have given.

There is a narrative that academic code is rubbish, and commercial code
is great. This is patently false. There is another narrative that
scientists and researchers cannot learn software engineering because
it is too complicated, so should
leave this to the professional software engineers. This is also false.

I remain a scientist. I remain an academic. And I am employed specifically
to write software in a university, so yes, I am a professional software
engineer (yes, I only just realised that *I am* a professional software
engineer!). Watching ten years of hard work improving the reputation
of academic software be blown away in a few days was very painful.
But the past is the past. We don't waste time making excuses for it, but
instead must all redouble our efforts to build a better future.

What should RSEs do?
--------------------

So what can RSEs do? MetaWards took two months from a very experienced
professional RSE working full time, 6-day weeks with no holidays or breaks.
This is clearly not a sustainable way of working. I estimate that if this
was a "normal" project, then this work would have taken three months.
And the epidemiologists's codes are only a few thousand lines. As a rough
guide I'd say you need 1 months RSE time per 1500 lines of code.

We cannot possibly add months of professional RSE time to every
research project that produces some software. Nor can we expect that every
researcher will learn enough software engineering to raise their
code up to a "professional" level. But we also cannot let the current
situation continue, where poorly engineered academic code is elevated
to "national-security-level" with no research software engineering support.

While we cannot make every academic code immediately "crisis-ready",
we equally cannot ask the virus to stop infecting people while we improve the
software. So, we need to somehow make sure that all academic codes that
could potentially be elevated to "national-security-level" have enough
basic scaffolding in place to support rapid productionalising and scale up.
And we need a national RSE capability who are ready in place to perform
that scale-up in partnership with the academics.

To this end, the `EPSRC RSE Fellowship <https://epsrc.ukri.org/funding/calls/rsefellowships/>`__
was fantastic because it provided
a free-floating RSE capability in the UK. I was funded to work flexibly
on academic research software projects, and so was able to drop
everything to work exclusively on MetaWards. I was able to take the decision
to do this as I only had to answer to the EPSRC
(I am sure that my REF and impact statement next year will look great ;-)).
As a nation, we spend hundreds of millions a year funding huge
instruments and supercomputers to support "normal" research. These
capability investments
were quickly repurposed for some excellent Covid-research (particularly
Diamond, which speaking as a one-time computational chemist, their
outputs and bound ligand structures have been incredible - a truly
fantastic example of why we must invest in capability).

We need to invest in people capability as well as equipment. A pool of highly
skilled RSEs who can be trusted to parachute into "national-security-level"
projects to support productising and scale-up of academic codes in times of
national crisis is absolutely needed. Equally we need to change the incentive
structure so that academics add the scaffolding that is necessary to support rapid
scale-up. At a minimum, academics must publicly release their software,
ideally as it is being developed. If they need to protect the software, then
they should release a "reference code" that can generate the same outputs.
I cannot publish a maths paper where I state the result of a proof but
do not publish the actual proof or any of my working, so why is it acceptable
to publish a graph without the source code? Even if "there is enough detail
in the paper to reproduce the result", the reality is that there isn't. And
there is no guarantee that the code that produced the graph was not full of
bugs, and hence the results are not reproducible. We therefore need a culture
change whereby it is unacceptable to publish the output from a piece
of custom software without publishing the software itself (or at least how
to achieve the same results from a reference implementation).

Equally, all researchers who write software should be taught the basics
of the craft. Again, I would not dare to publish a maths proof if I
had no training in basic mathematics. Equally, it should be unacceptable
for academics to write and publish software without a basic training
in software engineering. This means version control; writing documentation and
tutorials; and writing tests. And given nearly all researchers need to
write software, teaching in software engineering should be as ubiquitous
in school and university programmes as teaching maths.

And researchers must understand and demonstrate reproducibility. Software
must produce the same outputs for the same inputs (and random number seed),
or, if the code is non-reproducible because it is parallel, then the
code must reproduce results according to a statistical model.

We also need to remove the cult of personality / attitude of the expert
whereby people accept that a code works because an expert says "I've run
it and the graph looks right". I have had countless bugs in MetaWards
that produced graphs that looked right. Some were so small that one in
fifteen graphs were wrong, and it was only because I was running large
ensemble jobs for the tutorial, and taking care to query why some results
"looked weird" that I found and fixed those bugs (via unit tests, which
remain in the code to catch regressions). Equally, the most
insidious bug we've had only occurs when running in parallel on Windows.
It was caught because I use lots of run-time testing, and where
possible will calculate
values using two different methods and compare the results. In this case,
for 1 in 3 runs in Windows, this run-time test was triggered for parallel
runs and the code exits. If I didn't have this test, then the graph would
look right on this Windows user's machine, they would potentially publish
that result, but it would be wrong. It doesn't matter that the graph looks
(and is) right on my computer. It is wrong on theirs! As mentioned,
I am a professional research software engineer with 20+ years experience.
My code is full of bugs. But I apply a process (testing) to mitigate the
risk of those bugs affecting production results. Anyone who thinks that
they can write code that doesn't have bugs, and that don't need testing
to mitigate those bugs, is either naive, deluded or arrogant. And, I am
sorry, the middle of a crisis is exactly the time to start following
proper procedure, even if it does cost you a week to set up an
appropriate testing infrastructure (spoiler-alert, setting up a good testing
infrastructure shouldn't take a professional more than a day, and won't
stop anyone else from continuing to work).

Equally, we need to recognise that a bad testing infrastructure is worse
than no testing infrastructure. You should be writing tests as you are
writing code, and tests should exercise a good portion of the code
and run quickly (seconds). Slow tests should be partitioned off and run
during merges or before releases. For MetaWards, we use
`pytest.mark <https://metawards.org/development.html#custom-attributes>`__
to mark ``slow`` and ``veryslow`` tests. Then ``make quicktest`` runs
all of the quick tests in less than 5 seconds, while ``make test`` takes
about 5 minutes on my 2016 1.1 GHz Macbook. If your code uses lots of data,
then create a small input testing data set that allows your code to run
on developers laptops, so that developers can run tests easily and quickly
as part of their normal workflow.

Again, I know, won't this slow you down? Academics need to
understand that testing *speeds you up*, because the up-front investment
is more than paid off by increased productivity. And the slowest thing
you can do is to give the wrong advice because your code was wrong.

Reflections on the outbreak
---------------------------

I have spent two months watching thousands of simulated outbreaks of the lurgy
spread around the UK, and have been reading lots of papers and reports
about the spread of the real virus.  I am not an epidemiologist.
However, a *research* software engineer has a duty
to learn enough about the research area in which they are working so that
they are able to speak the language of the researchers and make meaningful
contributions that go beyond running static analysis tools, adding
basic "compare the output hash" integration tests, putting code on
GitHub or creating a pretty website or GUI.

I put the effort into reading the papers and understanding the maths.
I exposed my ignorance by asking the epidemiologists questions, and always
"putting my hand up" when I didn't understand something or got confused.
I continually revisited my assumptions and corrected my misinterpretations
(see the change in the tutorial regarding the description of "IW").

I reached a point where my suggestions of how to change the model were
valuable, and indeed novel (once things are quieter I may have a proper
epidemiology paper on interaction matrices and multi-demographic networks).
I gained enough trust that I was invited onto the epidemiologists' slack,
which is a lot like the RSE slack. And from this, I have learned a lot.

In my opinion, **a second wave this winter is almost inevitable** (and that was
before the latest mess in government that has all-but destroyed trust in
the lockdown, and likely the tracking app too).

To caveat, I have only modelled the lurgy, but have run thousands of
simulations. To this model I added a infectious asymptomatic stage, whereby
individuals with no or low symptoms were able to continue with their
normal behaviour, but were silently infecting others. This stage
made the lurgy extremely difficult to control. It is like a fire, and
infected individuals are like sparks. If they hit a group of susceptibles,
then the outbreak ignites. Concepts like R0 are basically irrelevant at
these early stages, as one or two people suddenly infect dozens or hundreds.
And cases like the `Itaewon cluster <https://mothership.sg/2020/05/itaewon-cluster-korea/>`__
in South Korea, the `Church Choir cluster <https://www.sciencemag.org/news/2020/05/why-do-some-covid-19-patients-infect-many-others-whereas-most-don-t-spread-virus-all>`__,
and others, show that prediction of the outbreak in these early stages
is essentially impossible. You cannot know when or if a single infected
individual will travel to a new city, or interact with a diverse group,
and thus create a new, large and quickly spreading cluster.

No matter how I modelled lockdown, every time it was released you would have
one spark find its way to a susceptible group and reignite the outbreak.
Yes, potentially contact tracing these small fires could work, but modelling
self-isolation suggests that the time period you have between someone thinking
they may be ill, and tracing and isolating all those they may have infected
for the lurgy was extremely short - i.e. potentially 1-2 days.
And *everybody* had to comply with self-isolation. If just 10% of individuals
didn't take it seriously, or made up their own rules, then the outbreak
spread massively. I do not believe we will reach the speed and availability
of testing, nor the level of adoption
or compliance necessary to make contact tracing or self-isolation work,
and the social and economic cost of these is too high for something that
will not be effective.

"Every little helps" was one response I had when I tweeted this. Unfortunately this
reply shows how little we humans understand complex networks. Yes, you
self-isolating will break your own path in the network. But it only takes
a few open paths for the entire network to be overwhelmed. Or, to use
an analogy, there is no point installing sprinklers in your 5th storey
apartment if no-one else has installed sprinklers and the students in the
basement flat are having a bonfire party. Indeed, you wasting time installing
sprinklers has a huge `opportunity cost <https://en.wikipedia.org/wiki/Opportunity_cost>`__,
as you really should be using your time and energy
to move house or find an alternative location for the students to hold
their party...

The sad truth for the lurgy, and I suspect also for covid, is that most of
the transmission of the virus is by people who are infected and either
asymptomatic or don't really notice they are ill. Once they become ill
enough to notice and stay at home, and to trigger an alert on a contact
tracing app, then it is too late - they have already infected others.
The only pathway I see is that we need to accept that coronavirus
is now endemic in the human population. It will never go away, just
as the "common cold" or flu have never gone away. We need to adapt
our society and our culture to adopt practices and building designs
that minimise transmission and protect the vulnerable.

So what changes do I think are needed? First, it is becoming clear that
this is an airborne virus. The
`hamsters and face masks <https://fightcovid19.hku.hk/hku-hamster-research-shows-masks-effective-in-preventing-covid-19-transmission/>`__
study, which I am still waiting to see published, so "not peer-reviewed yet
warning", raises two important observations;

1. Hamsters in the uninfected cage became infected when exposed to the air
   from the infected cage. This means that the virus can spread via breathing
   only. The hamsters were not touching each other, coughing or sneezing
   over each other, or touching infected surfaces. If this is confirmed by
   other groups then this is a very significant observation. And no,
   I probably won't then want to get on a plane, or sit in a crowded bus,
   or be in a small meeting room for an hour, etc. etc...

2. The level and severity of infection was significantly reduced when a
   simple surgical face mask was placed in the tube connecting the two
   hamster cages together. This means that even basic face masks block
   a good percentage of the viral particles, and thus reduce the number
   of hamsters infected, and importantly, reduce the viral load at the
   time of infection (thereby giving the immune system more time to
   learn how to beat the virus before the body is overwhelmed).

I don't think it is a co-incidence that SE Asian nations have been spared
the worst of the virus. They have a strong culture of good hygeine and
mask wearing. I spent a long time in Thailand and am married to Thai
national (we got together when she was using my software to study the
flu outbreak in 2010-11). In Thailand, if you are ill and come into an office,
you had better be wearing a mask. You do not cough on people. You don't
sneeze on people. You don't blow your nose in public. You don't shake hands.

Equally, there is a culture of shaming if you become ill. You work out who
could have infected you and you go and talk to them and tell them off.
The Thai experience of bird and swine flu mean that the public know what
H5N1 and H1N1 means,
and
`air conditioners <https://www.samsung.com/sa_en/air-conditioners/wall-mount-ar18hvsdewk/>`__
and hand sanitisers are marketed as "killing all
viruses inluding H1N1".

Thai culture, like most SE Asian cultures, responded
to these outbreaks by developing practices that naturally minimise spread of
airborne diseases. To quote the
`report from the lessons we could learn from Thailand <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3376790/>`__
from the H1N1 flu outbreak;

"The use of good hand hygiene practices, social distancing measures,
and face masks was emphasized in national policy and prevention guidelines.
These measures were widely promoted and implemented, particularly during the
first pandemic wave, when awareness and anxiety levels were high."

We need a similar culture change to routinely adopt Thai (and
other SE Asian) practices in the UK:

1. Assume everyone, including yourself, is infected. Wear a mask in enclosed
   environments. Wash your hands regularly. Do not shake hands, but instead
   bob your head or `wai <https://en.wikipedia.org/wiki/Thai_greeting>`__.

2. Put cleaning materials on all tables (or carry your own) and adopt the
   practice of wiping the table clean before you sit down and eat
   or have a meeting.

3. Put cleaning materials in the toilets (or carry your own) and adopt
   the practice of cleaning the toilet seat and surroundings before you use it.

4. Boiling water should be placed next to cutlery in cafeterias so that
   you can wash the cutlery before eating. Eating implements (chopsticks,
   straws, cups) are individually wrapped in plastic (sorry World) so that
   there is no chance of cross-contamination.

5. Design rooms with vertical airflow and use good ventilation to either
   exhange inside with outside air or use good virus-catching filters.

6. Have all employees, particularly those involved in food production,
   wear masks and/or face guards (like
   `these here from Korea and Singapore <https://en-gb.facebook.com/pg/mascare.sg/posts/>`__.
   These were routinely used from before the outbreak).

7. Reduce face to face meetings or large gatherings to only those that
   are necessary. If something can be done from home or from a small
   personal office, or if something can be done online, then it should.
   Good bye hot desking!

8. Most importantly, do not go out if you have any symptoms of a cold,
   flu or coronavirus style illness (which includes driving four hours
   to another city). Do not go to work. Do not return until you are 100%
   better. And this last point is hard because many cannot take this amount
   of time off. Hence, we need legal safeguards and the culture change
   to protect those that stay home, and to shame those that come in.

These cultural changes are not difficult. We have, perhaps 4-5 months
before the second wave hits. I am a software engineer, and not an
epidemiologist, so trust me on this topic as much as you would trust
an epidemiologist when they write or talk about software.

But do look at the empirical evidence. Japan - 16.5k cases, 825 deaths.
Singapore - 16.5k cases, 825 deaths. Thailand - 3k cases, 57 deaths.
Vietnam - 325 cases, 0 deaths. Myanmar - 201 cases, 6 deaths.

Even though reporting is different, testing is different, "they may
be hiding their cases" etc., UK - 260k cases, 36.8k deaths (and the UK has
about the same population and geographic size as Thailand).

As a developer, the one rule that binds them all is
`keep it simple, stupid - KISS <https://en.wikipedia.org/wiki/KISS_principle>`__.
Complex bluetooth-based tracing apps,
complex lockdown messages, complex rules that even those at the top
interpret differently and don't follow are the antithesis to KISS.

In Thailand, the malls are open and everyone is enjoying shopping and eating
in restaurants again. A simple QR-code reading app counts how many are
inside each building and limits their numbers. You can see on a map which
shops have space and where you would have to queue. If an outbreak came,
they have the details of who was in which shops. Restaurants have
`used mascots <https://fortune.com/2020/05/22/how-to-reopen-restaurants-social-distancing-coronavirus/>`__
or `toy pandas <https://www.independent.co.uk/life-style/food-and-drink/lockdown-thailand-restaurant-pandas-social-distancing-a9514971.html>`__
to block seats and socially distance customers. Handwash and masks are freely
available and required to be used, with guards on the doors making customers
follow the rules.

They are not perfect, and I am sure there is much they have and will do
wrong. But it is clear to me that culture and collective community
action are `our most effective tools to fight this virus <https://xkcd.com/2287/>`__.
Success is not who has the best technology or spends the most money,
as the rich and technologically advanced West
has demonstrated. Put simply, which culture do you think is better placed
to survive the second wave? A culture who wear masks because they offer
*some* protection to others, or the culture where people argue that
masks are useless because they don't offer 100% protection to me?
