******************
Ways to Contribute
******************

Getting Started
===============

---------------

Get UnifyCR
-----------

You can build and run UnifyCR by following :doc:`these instructions <build-intercept>`.

Getting Help
------------

To contact the UnifyCR team, send an email to the
`mailing list <ecp-unifycr@exascaleproject.org>`_.

Reporting Bugs
==============

Please contact us via the `mailing list <ecp-unifycr@exascaleproject.org>`_ if
you are not certain that you are experiencing a bug.

You can open a new issue and search existing issues using the
`issue tracker <https://github.com/LLNL/UnifyCR/issues>`_.

If you run into an issue, please search the 
`issue tracker <https://github.com/LLNL/UnifyCR/issues>`_ first to ensure the
issue hasn't been reported before. Open a new issue only if you haven't found
anything similar to your issue.

.. important::

    **When opening a new issue, please include the following information at the top of the issue:**

    - What operating system (with version) you are using
    - The UnifyCR version you are using
    - Describe the issue you are experiencing
    - Describe how to reproduce the issue
    - Include any warnings or errors
    - Apply any appropriate labels, if necessary

When a new issue is opened, it is not uncommon for developers to request
additional information.

In general, the more detail you share about a problem the more quickly a
developer can resolve it. For example, providing a simple test case is
extremely helpful. Be prepared to work with the developers investigating your
issue. Your assistance is crucial in providing a quick solution.

Suggesting Enhancements
=======================

Open a new issue in the `issue tracker <https://github.com/LLNL/UnifyCR/issues>`_
and describe your proposed feature. Why is this feature needed? What problem
does it solve? Be sure to apply the *enhancement* label to your issue.

Pull Requests
=============

- All pull requests much be based on the current *dev* branch and apply without
  conflicts.
- Please attempt to limit pull requests to a single commit which resolves one
  specific issue.
- Make sure your commit messages are in the correct format. See the
  :ref:`Commit Message Format <commit-message-label` section for more
  information.
- When updating a pull request, squash multiple commits by performing a
  `rebase <https://git-scm.com/docs/git-rebase>`_ (squash).
- For large pull requests, consider structuring your changes as a stack of
  logically iindependent patches which build on each other. This makes large
  changes easier to review and approve which speeds up the merging process.
- Try to keep pull requests simple. Simple code with comments is much easier to
  review and approve.
- Test cases should be provided when appropriate.
- If your pull request improves performance, please include some benchmark
  results.
- The pull request must pass all regression tests before being accepted.
- All proposed changes must be approved by a UnifyCR project member.

Testing
=======

All help is appreciated! If you're in a position to run the latest code,
consider helping us by reporting any functional problems, performance
regressions, or other suspected issues. By running the latest code on a wide
range of realistic workloads, configurations, and architectures we're better
able to quickly identify and resolve issues.
