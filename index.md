[TurkServer](https://github.com/HarvardEconCS/turkserver-meteor) is a framework based on the JavaScript app platform [Meteor](https://www.meteor.com/) that makes it easy to build interactive web-based user experiments for deployment on [Amazon Mechanical Turk](https://www.mturk.com/mturk/welcome).
2. **Knows the basics of the JavaScript app platform [Meteor](https://www.meteor.com/).** If you have never used Meteor before, that's fine -- it's quite easy to learn. Before continuing with this tutorial, [install](https://www.meteor.com/install) Meteor and then complete the [to-do app tutorial](https://www.meteor.com/tutorials/blaze/creating-an-app).
All the code for the project that you will build during the tutorial can be found [here](https://github.com/ldworkin/turkserver-tutorial). If you have questions, comments, or suggestions, please add a Github issue to that repo.
For easiest integration with TurkServer, you will need to add some basic routing to your app. This how you connect different urls ("routes") to different templates. We will start with two routes:

1. The default path `/`, which will get shown when a worker is previewing your HIT.
2. A second path `/experiment`, which will get shown after the worker accepts the HIT.
    Please accept the HIT to continue.
</template>

<template name="experiment">
We have done two things here: we wrapped the existing HTML in a template called "experiment", and we added another template called "home." We want to show the "home" template when a worker is previewing the HIT, and the "experiment" template after they have accepted.
Router.route('/', function() {
  this.render('home');
});

Router.route('/experiment', function() {
  this.render('experiment');
For a more detailed explanation of this code and  a better understanding how routing works, see the [iron-router docs](http://iron-meteor.github.io/iron-router/). Essentially, we have told the app that when we navigate to the default route `'/'`, we should see the "home" template, and when we navigate to the route `'/experiment'`, we should see the "experiment" template.
Finally, run your app with `meteor --settings settings.json`. When you navigate to `http://localhost:3000/` (the default route `'/'`), you should see your "home" template (which prompts you to accept the HIT), and now you should also see a popup that prompts you to select a "batch:"
- When a user accepts the HIT, he goes into the lobby.
- When a user finishes an experiment, he goes back to the lobby.
- When a user goes into the lobby for the first time (i.e. after he accepts the HIT), he immediately gets put into a *joint* experiment with all other users who have also accepted the HIT.
- When a user goes to the lobby for the second time (i.e. when he is done with the experiment), he immediately gets put into the exit survey.
Let's assume that when a user is in the experiment, we want him to see the "Click Me" page. And when the user is in the exit survey, we want him to see a submit button that will allow him to submit the HIT. We already have the "Click Me" page (in the "experiment" template), so let's design the exit survey.
Add the following block to routes.js:
Router.route('/survey', function() {
  this.render('survey');
Finally, we need to tell our app that when a user is in the experiment state, we should load the `'/experiment'` route (which is connected to the "experiment" template), and when a user is in the exit survey state, we should load the `'/survey'` route (which is connected to the "survey" template). Add the following inside the `Meteor.isClient` block at the top of demo.js:
    Router.go('/experiment');
And define the new method `goToExitSurvey()` within the `Meteor.isServer` block (but outside of the `Meteor.startup()` block) at the bottom of demo.js:
In our new method `goToExitSurvey()`, we first grab the JavaScript object corresponding to the user's current experiment. (TurkServer uses the terms "instance" and "experiment" interchangeably; I will try to stick to "experiment.) Then we call the `teardown()` method of this object, which ends the experiment. So this point the user goes back to the lobby, and then the SimpleAssigner sends him to the exit survey, so he can submit the HIT.
To add the SimpleAssigner to your new "main" batch, add the following code within the `Meteor.startup` block of the `Meteor.isServer` block at the bottom of demo.js:
A few more TurkServer API calls here. First we grab the JavaScript object corresponding to the batch that we created. Then we call its method `setAssigner()` to tell our app that all HITs in this batch should use the SimpleAssigner.
After "accepting" the HIT, you go straight into the lobby. The code sees that you are in batch "main", and that the assigner on this batch is a SimpleAssigner. The SimpleAssigner ensures that when you enter the lobby for the first time, you go straight into the experiment stage. So `TurkServer.inExperiment()` becomes true and the `Tracker.autorun` block in demo.js calls `Router.go('/')`. The routing code in routes.js then loads the "hello" template, so you see the "Click Me" page.
- The **Instances** field shows one square blue icon, which corresponds to the one experiment you did in this assignment. If you click on that icon, you will see a popup with the id of that experiment, as well as a list of the users who participated:
Now go to demo.js, and within the `Meteor.isClient` block, add the following:
Next, within `Meteor.isServer` block, add:
## Setting Payment

First this code grabs the JavaScript object corresponding to the user's current assignment. Recall that we have an assignment object for every <user, HIT> pair, so this is the natural place to store a bonus. Then we call the method `addPayment()` to increment the bonus by a fixed amount (in this case, 10 cents).
You should start seeing a pattern emerging with these TurkServer API calls. First we grab a particular JavaScript object -- e.g. a batch, an experiment, or an assignment. Then we call one of its methods. If you're curious about what methods are available on these objects, see the relevant files in turkserver-meteor/lib, namely batches.coffee, instance.jsx, and assignment.jsx.
## Deployment
First we have to deploy our app. There are two good options: [Modulus](https://modulus.io/) (very beginner-friendly) and [DigitalOcean](https://modulus.io/) (less beginner-friendly, but much more flexible). A fantastic tutorial that covers both options is given [here](http://meteortips.com/deployment-tutorial/). We'll only be covering Modulus in this tutorial. If you choose to deploy on DigialOcean (or any other cloud server) instead, simply skip ahead to the next section. (But first be sure to install an SSL certificate on your server, because otherwise Mechanical Turk will block the external HIT.)
Go [here](http://meteortips.com/deployment-tutorial/modulus/) to the Modulus section of the tutorial referenced above, and follow all steps up to and including the `modulus deploy` part, at which point you should receive the URL of your new Meteor application. Before continuing, make the following changes to your settings.json file:
1. Make sure you have added your Mechanical Turk key and secret.
2. Set the sandbox field to have a value of `true`.
3. Set the external url field to the URL you received above -- but add 'https://' as a prefix!
4. Set the frame height field to be the desired size of your iFrame in the external HIT (usually 600 - 800 will do).
Now continue the tutorial. When it comes time to add your environment variables, you'll need to add another one with a key of METEOR_SETTINGS and a value equal to the *entire contents of your settings.json* file. Simply cut and paste that entire JSON blob. It should look something like this:
![screenshot](img/meteor-settings.png)

Hit Save, and redeploy your app with `modulus deploy`. Go to your URL and verify that you can login both to the admin console and as a test user.