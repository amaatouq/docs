> Much of this fantastic tutorial is due to the excellent work of [Lili Dworkin](http://lilianne.me/).

xs
TurkServer is designed around the Meteor framework. There are many
reasons that Meteor is especially powerful for web development; see
[this post](http://www.quora.com/Should-I-use-Meteor-Why) for a
summary. But more importantly, there are
[tons of learning resources](https://www.meteor.com/learn) for new
users to get started. The design philosophy of TurkServer is to stick
to standard Meteor as much as possible, while minimizing the need to
use custom APIs. This means that most of the outstanding Meteor
documentation on the Internet will be useful, and that most of the
required knowledge is not specific to TurkServer.

All the code for the project that you will build during the tutorial can be found [here](https://github.com/VirtualLab/tutorial). If you have questions, comments, or suggestions, please add a Github issue to that repo.
- **HIT**: A task that workers can complete on Mechanical Turk. A HIT
  can be completed by multiple workers, but each worker can only
  complete a particular HIT once.
- **External Question**: A HIT with an
  [external question](http://docs.aws.amazon.com/AWSMechTurk/latest/AWSMturkAPI/ApiReference_ExternalQuestionArticle.html)
  is hosted on your *own website* and displayed in an iframe in the
  Worker's web browser. TurkServer facilitates the process of building
  external HITs.
In using TurkServer, every Assignment of a HIT will go through one of three states (not necessarily in a linear fashion):
1. **Lobby** -- Where a worker goes right after accepting the HIT, and also between participating in experiment instances.
2. **Experiment Instance** -- Where the worker actually completes the
   task. Experiments are a logical way of setting up interaction
   between a group of workers and can consist of any number of
   workers, including just one.
> Each participant can be assigned to one instance at a time, but can
> participate in multiple instance over the course of a HIT, so
> that each instance can have one or more simultaneous
> participants. This supports various synchronous and longitudinal
> experiment designs.

**The developer using TurkServer is responsible for building the user
  interface for each of these stages.** In other words, TurkServer
  assumes you already have a Meteor app that controls the experience
  workers have after accepting your HIT and before submitting
  it. TurkServer is just the middleman between your app and Mechanical
  Turk. The framework will take care of connecting workers to your app
  when they accept the HIT, and submitting their work to Mechanical
  Turk when they are done with your HIT. TurkServer also allows you to
  easily monitor your experiment while in progress. But you first need
  to build the app that allows workers to complete the desired
  task. So let's do that now.
Next create a settings.json file, which needs to contain a password
for the TurkServer admin interface (from which you will monitor your
experiment) as well as your Amazon key and secret pair. (If you don't
have those, or don't remember where to find them, go
[here](http://docs.aws.amazon.com/AWSMechTurk/latest/AWSMechanicalTurkGettingStartedGuide/SetUp.html)).
The file should look like this:
We have now used the TurkServer API for the first time. The function
`submitExitSurvey()` will automatically submit a HIT to Mechanical
Turk. The function takes arbitrary JSON as an argument, so you can
pass in any data you want. The data will viewable in the TurkServer
admin console in real time. A natural thing to do is put a form in
your exit survey that asks the worker some follow-up questions about
the HIT. For example, modify your template as follows:
There are a few things worth noting about this code. We have made use of two boolean reactive variables provided by TurkServer: `TurkServer.inExperiment()` and `TurkServer.inExitSurvey()` (unsurprisingly, there is also `TurkServer.inLobby()`). When the state of the user changes, the value of these variables will update accordingly. The `Tracker.autorun` block will re-run whenever the variables change, which means that whenever the state of a user changes, our app will load the appropriate route, and therefore show the appropriate template.

> `Tracker.autorun` is the standard reactive programming approach in
> Meteor. If you are confused about this, take a minute and read
> [this](http://docs.meteor.com/#/basic/tracker).
  'click button#clickMe': function () {
Now that we have a batch and an assigner, our HIT flow will work
properly, so we can finally test it all out. TurkServer allows you to
login as a fake Mechanical Turk user, which is very useful for testing
without having to create HITs on your MTurk sandbox account. Navigate
to `http://localhost:3000/` and you'll see the popup from
before. TurkServer is asking us which batch we want to test out. We
only have one ("main"), which should be selected by default.
When you are ready to move to the exit survey, click "Exit Survey."
Now the client calls the `goToExitSurvey()` method, which ends the
experiment, so you go back to the lobby. The SimpleAssigner ensures
that when you enter the lobby for the second time, you go straight to
the exit survey stage. So `TurkServer.inExitSurvey()` becomes true,
which results in our router loading up the `'/survey'` route, and the
"survey" template is displayed.
It's not crucial to understand all of what's happening here, but the
general idea is important. A key concept to Turkserver's functionality
is the idea of "partitioned" Mongo collections. A partitioned
collection has the property that *every document belongs to exactly
one experiment*. When a user is in an experiment, he can only see
documents in the collection that belong to that experiment.
> The partitioning of collections is a key abstraction that allows
> experiments in TurkServer to be designed at the instance level. This
> in turn allows for many different types of experiment designs, while
> differing very little from Meteor's standard publish-subscribe
> model. In many cases, multi-person interactions are as simple to
> design as single-person interactions.

> Advanced users might mix partitioned and non-partitioned collections
> in Meteor code. Partitioned collections will show different data to
> participants in each experiment instance, while non-partitioned
> collections must be separately controlled by your code.

> See the
> [docs for partitioner](https://github.com/mizzao/meteor-partitioner)
> for more details on how this works.

A re-subscription triggers the publication code on the server. The
`Meteor.publish` code we added looks standard, but because the user is
currently in an experiment, the `Clicks.find()` part will *only return
documents that also belong to this experiment*. This way, a user only
subscribes to the documents that are relevant to his experiment,
automatically dividing real-time data between groups. (If this was
confusing and you need a refresher on Meteor's publish/subscribe
protocol, take a look at the
[docs](http://docs.meteor.com/#/basic/pubsub).)
We don't need to add any other fields to the Click document besides
`count`. This is because Clicks is partitioned, so whenever we add a
document, it automatically gets "tied" to the current experiment. More
precisely, behind the scenes, a hidden field called `_groupId` is
added. The value of this field is the id of the current experiment
(i.e. the id of the Mongo document that corresponds to the current
experiment -- more on this shortly). So later when we analyze our
data, we can search for a Click document with `_groupId: <our
experiment Id>`, and we'll find the right document and be able to tell
how many times we clicked the button. We'll go through that exercise
later and it will be clearer then.
First we have to deploy our app. There are two good options:
[Modulus](https://modulus.io/) (very beginner-friendly) and
[DigitalOcean](https://modulus.io/) (less beginner-friendly, but much
more flexible). A fantastic tutorial that covers both options is given
[here](http://meteortips.com/deployment-tutorial/). We'll only be
covering Modulus in this tutorial. If you choose to deploy on
DigitalOcean (or any other cloud server) instead, simply skip ahead to
the next section. (But first be sure to install an SSL certificate on
your server, because otherwise Mechanical Turk will block the external
HIT.)
The next step is to create a new HIT type. We can do this in the admin
console. Click on "MTurk" in the admin console navigation pane and go
through the process of filling out the form on the right side. First
we have to choose a batch for the HIT type. All HITs with this HIT
type will belong to the batch we choose. This important because the
only way TurkServer knows which **assigner** to use on a particular
HIT is by checking which **batch** the HIT belongs to. If TurkServer
doesn't know what assigner to use, then the app won't know where to
send a user after he enters the lobby, and the user will end up in a
limbo state.
Finally, let's test this out as a sandbox worker. Go to the
[worker sandbox](https://workersandbox.mturk.com/mturk/) *in a
different browser or Chrome incognito tab*. Click the HITs tab and
then search for your HIT (using either your requester name or the name
of the HIT). When you see it, click the "View a HIT in this group"
link in the upper right of the box. If all went well, your app will be
loaded into an iFrame at the default route `'/'`, which shows the
"home" template and prompts you to accept the HIT:
## Other Features - **UNFINISHED**

That concludes the main section of this tutorial; in this section I
will briefly point out some other features of TurkServer that are
worth knowing about.

### Batches and Treatments

TurkServer uses the concept of **batches** to logically group
instances of experiments together. Each batch limits repeat
participation.

Batch controls the assignment of incoming users to
**treatments**. Treatments can have structured data which are made
available to the front-end app under `TurkServer.treatment()`, making
them a useful way to control the display of different parts of the
user interface or app behavior. They can be defined for batches,
users, or worlds.

Batches and treatments can be viewed and edited from the
administration interface.

###

Meteor already makes it pretty easy to design a reactive and responsive user interface, but you may find some of the following packages useful.

- [Bootstrap](http://getbootstrap.com/), a CSS framework for front-end development.
- [Tutorials](https://github.com/mizzao/meteor-tutorials), a Meteor-specific tutorials package that I wrote for providing interactive and concise instructions for web apps. Very useful for delivering easily digestible experiment instructions.

## Online Experiment Methodology

While TurkServer provides much of the software components necessary
for deploying web-based experiments, there is still a great deal that
is helpful to know to ensure that your study goes smoothly and workers
leave happy. The lab at
[Microsoft Research NYC](http://research.microsoft.com/en-us/labs/newyork/)
aims to pioneer the development of web-based behavioral experiments,
and you should feel free to contact
[Andrew Mao](http://www.andrewmao.net/) or
[Sid Suri](http://www.sidsuri.com/) with questions about your
experimental design.

As design of web-based experiments is a quickly changing landscape, we
hope to update this section in the future with links to a more
comprehensive tutorial.

## API Reference
See [turkserver.meteor.com](https://turkserver.meteor.com).