---
title: "Bug Fixing Steps"
excerpt: "Walking through the steps taken to fix a bug."
categories:
  - Articles
tags:
  - Debugging
---

## Introduction

Prevention is better than cure so you should aim to stop bugs arising in the first place. Having rigorous testing, code reviews and code analysis tools help; however, even with strong bug prevention measures most complex systems will inevitably contain bugs.

This article aims to describe some useful techniques to debug complex systems. 


## Create a ticket

After discovering a bug you should check your project's bug tracker (e.g. JIRA) to see if there is already a ticket tracking the issue.

If there isn’t then you should generate a ticket and clearly explain the problem. You should describe:

* **The context:** what state the system was in before the trigger.
* **The trigger:** what command you ran.
* **Expected behaviour:** what you thought would happen.
* **Actual behaviour:** what actually happened.

Creating a ticket:

* Helps other colleagues who also come across this bug.
* Helps you clarify what the problem is.
* Can give context to coworkers if you need to ask for help.


## Reproduce the problem

If you didn’t discover the bug, but instead are trying to solve a bug someone else discovered, you should first ensure you can reproduce the bug according to the ticket description. This will:

* Ensure the reproduction steps are correct.
* Aid your understanding of the problem.

If the author of the bug report is not familiar with the system, the reproduction steps can sometimes contain superfluous commands. You should aim to trim down the reproduction steps to create the minimal number of steps necessary to reproduce the bug. Doing so is useful because:

* It helps to localise the problem so you likely know which individual component is problematic.
* You can reproduce the bug quicker which speeds up investigative manual testing.


## Check the log

When you suspect a system is not behaving correctly one of the first places to look is at the log files.

Different systems and applications have different logging infrastructure so you’ll have to know how your system handles logging.

Most systems have different levels of logging verbosity. At the lowest level you will only see critical error messages, as you increase the verbosity you will see more informational messages which are benign. 

You should look for logging messages related to your problem. You can do this by filtering the system log to only show messages from components you expect to be causing the issue or by searching for keywords related to the bug.

If you find an error message worth investigating you can try copying and searching for the message on:

* Google (if it’s a public library)
* Your internal bug tracker
* The repositories source code


## Check if it’s a regression

If the system has successfully performed the expected behaviour in previous releases then the bug can be classified as a regression. For regressions, once you establish the most recent version that worked correctly and the oldest version that worked incorrectly, then you can inspect the changes between the 2 versions and identify what introduced the failure. 

To speed up finding which commit introduced the problem, instead of stepping through each commit you can use [git bisect]([https://git-scm.com/docs/git-bisect](https://git-scm.com/docs/git-bisect)) which will do a binary search on the project’s commit history to find the first commit which introduced the bug.

Other useful git commands are:

* `git blame`: See who modified each line of a file.
* `git log -S <some_keyword>`: Find commits which added or removed a keyword.
* `git log --grep=<some_keyword>`: Find commits whose commit message contains a keyword.
* `git log -L :<funcname>:<file>`: Find commits which have changed a certain function.


## Add new unit testing

Once you have identified which component is causing the bug in the system you can stop debugging the system as a whole and instead focus on a single component. This generally makes debugging easier. From the system debugging you want to capture what input was passed to the component.

Using this information you can write a new unit test for the component, which will initially fail. Doing this will:

* Validate your understanding of the problem.
* Provides a test case for the debugger to step through the relevant code paths.
* Provides a test case which can be run as part of the CI system to ensure the bug doesn’t reappear.


## Ask for help

If you have exhausted all avenues and you have the option of asking for advice then it is worth doing.

Before reaching out you should prepare your existing work to make it easier for the person helping to get up to speed. You can:

* Clean up your code (descriptive variable & function names, add comments, etc) and push it to the remote server.
* Write up what you have learned so far (your understanding of the infrastructure, what approaches you have tried so far, etc).
* Write out a clear message explaining what the problem is and where you are stuck.

Before sending off a message for help you should articulate the problem to an inanimate object or an imaginary person. In the process of explaining the problem you may end up evaluating it from a different perspective and find a solution on your own. This technique is commonly known as [rubber duck debugging]([https://en.wikipedia.org/wiki/Rubber_duck_debugging](https://en.wikipedia.org/wiki/Rubber_duck_debugging)).

If you did not find any new ideas worth investigating during the preparation process then send off your message requesting help. Be prepared to be active in jointly finding a solution rather than simply passing off the problem.


## Consider prevention measures

Once the behaviour of the bug is understood it can be useful to document on the ticket what went wrong. This will provide context to your pull request reviewers by allowing them to understand what the problem was before they see how you fixed it.

Knowing how the bug was introduced, you should consider whether it could have been prevented in the first place. For example here are some example problems and example solutions:

<table>
  <tr>
   <td><strong>Possible Problem</strong>
   </td>
   <td><strong>Potential Solution</strong>
   </td>
  </tr>
  <tr>
   <td>The developer misunderstood an internal API.
   </td>
   <td>The API needs more documentation.
   </td>
  </tr>
  <tr>
   <td>The unit tests should have identified the problem.
   </td>
   <td>The unit tests need more code coverage.
   </td>
  </tr>
  <tr>
   <td>The developer made a simple mistake on a mundane chore
   </td>
   <td>Automate the mundane chore.
   </td>
  </tr>
</table>


These potential solutions may prevent a similar bug. 

The priority for implementing the potential solutions will depend on: 

* The likelihood of a similar bug being introduced.
* The severity of a similar bug.
* The ease of implementing the prevention measure.

You may wish to discuss the potential solutions at your team’s next retrospective meeting.

