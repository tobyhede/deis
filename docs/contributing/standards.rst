:title: Coding Standards
:description: Deis project coding standards. Contributors to Deis should feel welcome to make changes to any part of the codebase.

.. _standards:

Coding Standards
================

Deis is a Go_ and Python_ project (with some shell scripts
and Makefiles). In the spirit of those programming languages, Deis
code should be simple rather than clever. An open source project
benefits from many eyes upon it, so readability counts.

Proposed changes to Deis should be discrete, self-contained, and
ideally small units. It may seem convenient just to fix that bug
you found while you were implementing a feature, but both changes
are more likely to be accepted if they are presented as separate
pull requests.

Pull Request Checklist
----------------------

1. Code changes address exactly one issue
2. Git commit message(s) are in Deis' required format
3. Includes documentation (docs/ dir and/or README.md files)
4. Code comments added or updated
5. Tests all passed

.. _pull_request:

Pull Request
------------

Now create a GitHub `pull request`_ with a description of what your code
fixes or improves.

Before the pull request is merged, make sure that you squash your
commits into logical units of work using ``git rebase -i`` and
``git push -f``. Include documentation changes in the same commit,
so that a revert would remove all traces of the feature or fix.

Commits that fix or close an issue should include a reference like
*Closes #XXX* or *Fixes #XXX* in the commit message. Doing so will
automatically close the `GitHub issue`_ when the pull request is merged.

Merge Approval
--------------

Deis maintainers add "**LGTM**" (Looks Good To Me) in code
review comments to indicate that a PR is acceptable. Any code change--other than
a simple typo fix or one-line documentation change--requires at least two of
Deis' maintainers to accept the change in this manner before it can be merged.
If the PR is from a Deis maintainer, then he or she should be the one to merge
it. This is for cleanliness in the commit stream as well as giving the
maintainer the benefit of adding more fixes or commits to a PR before the
merge.

.. _Python: http://www.python.org/
.. _Go: http://golang.org/
.. _flake8: https://pypi.python.org/pypi/flake8/
.. _pep8_tool: https://pypi.python.org/pypi/pep8/
.. _pyflakes: https://pypi.python.org/pypi/pyflakes/
.. _mccabe: https://pypi.python.org/pypi/mccabe/
.. _PEP8: http://www.python.org/dev/peps/pep-0008/
.. _`The Zen of Python`: http://www.python.org/dev/peps/pep-0020/
.. _`pull request`: https://github.com/deis/deis/pulls
.. _`GitHub issue`: https://github.com/deis/deis/issues


.. _commit_style_guide:

Commit Style Guide
------------------

There are several reasons why we try to follow a specific style guide for commits:

- it allows us to recognize unimportant commits like formatting
- it provides better information when browsing the git history

Recognizing Unimportant Commits
```````````````````````````````

These commits are usually just formatting changes like adding/removing spaces/empty lines,
fixing indentation, or adding comments. So when you are looking for some change in the
logic, you can ignore these commits - there's no logic change inside this commit.

When bisecting, you can ignore these by running:

.. code-block:: console

    git bisect skip $(git rev-list --grep irrelevant <good place> HEAD)

Providing more Information when Browsing the History
````````````````````````````````````````````````````

This adds extra context to our commit logs. Look at these messages (taken from the last
few AngularJS commits):

- Fix small typo in docs widget (tutorial instructions)
- Fix test for scenario.Application - should remove old iframe
- docs - various doc fixes
- docs - stripping extra new lines
- Replaced double line break with single when text is fetched from Google
- Added support for properties in documentation

All of these messages try to specify where the change occurs, but they don’t share any
convention. Now look at these messages:

- fix comment stripping
- fixing broken links
- Bit of refactoring
- Check whether links do exist and throw exception
- Fix sitemap include (to work on case sensitive linux)

Are you able to guess what’s inside each commit diff?

It's true that you can find this information by checking which files had been changed, but
that’s slow. When looking in the git history, we can see that all of the developers are
trying to specify where the change takes place, but the message is missing a convention.
Cue commit message formatting entrance stage left.

Format of the Commit Message
````````````````````````````

.. code-block:: console

    {type}({scope}): {subject}
    <BLANK LINE>
    {body}
    <BLANK LINE>
    {footer}

Any line of the commit message cannot be longer than 72 characters, with the subject
line limited to 50 characters. This allows the message to be easier to read on github
as well as in various git tools.

Subject Line
""""""""""""

The subject line contains a succinct description of the change to the logic.

The allowed {types} are as follows:

- feat -> feature
- fix -> bug fix
- docs -> documentation
- style -> formatting
- ref -> refactoring code
- test -> adding missing tests
- chore -> maintenance

The {scope} can be anything specifying place of the commit change e.g. the controller,
the client, the logger, etc.

The {subject} needs to use imperative, present tense: “change”, not “changed” nor
“changes”. The first letter should not be capitalized, and there is no dot (.) at the end.

Message Body
""""""""""""

Just like the {subject}, the message {body} needs to be in the present tense, and includes
the motivation for the change, as well as a contrast with the previous behavior.

Message Footer
""""""""""""""

All breaking changes need to be mentioned in the footer with the description of the
change, the justification behind the change and any migration notes required. Any methods
that maintainers can use to test these changes should be placed in the footer as well. For
example:

.. code-block:: console

    TESTING: to test this change, bring up a new cluster and run the following
    when the controller comes online:

        $ vagrant ssh -c "curl localhost:8000"

    you should see an HTTP response from the controller.

    BREAKING CHANGE: the controller no longer listens on port 80. It now listens on
    port 8000, with the router redirecting requests on port 80 to the controller. To
    migrate to this change, SSH into your controller and run:

        $ docker kill deis-controller
        $ docker rm deis-controller

    and then restart the controller on port 8000:

        $ docker run -d -p 8000:8000 -e ETCD=<etcd_endpoint> -e HOST=<host_ip> \
        -e PORT=8000 -name deis-controller deis/controller

    now you can start the proxy component by running:

        $ docker run -d -p 80:80 -e ETCD=<etcd_endpoint> -e HOST=<host_ip> -e PORT=80 \
        -name deis-router deis/router

    the router should then start proxying requests from port 80 to the controller.

Referencing Issues
""""""""""""""""""

Closed bugs should be listed on a separate line in the footer prefixed with the "closes"
keyword like this:

.. code-block:: console

    closes #123

Or in the case of multiple issues:

.. code-block:: console

    closes #123, #456, #789

Examples
````````

.. code-block:: console

    feat(controller): add router component

    This introduces a new router component to Deis, which proxies requests to Deis
    components.

    closes #123

    BREAKING CHANGE: the controller no longer listens on port 80. It now listens on
        port 8000, with the router redirecting requests on port 80 to the controller. To
        migrate to this change, SSH into your controller and run:

        $ docker kill deis-controller
        $ docker rm deis-controller

        and then restart the controller on port 8000:

        $ docker run -d -p 8000:8000 -e ETCD=<etcd_endpoint> -e HOST=<host_ip> \
        -e PORT=8000 -name deis-controller deis/controller

        now you can start the proxy component by running:

        $ docker run -d -p 80:80 -e ETCD=<etcd_endpoint> -e HOST=<host_ip> -e PORT=80 \
        -name deis-router deis/router

        The router should then start proxying requests from port 80 to the controller.
    ----------------------------------------------------------------------------------
    test(client): add unit tests for app domains

    Nginx does not allow domain names larger than 128 characters, so we need to make
    sure that we do not allow the client to add domains larger than 128 characters.
    A DomainException is raised when the domain name is larger than the maximum
    character size.

    closes #392

Forcing no Build for Jenkins
""""""""""""""""""""""""""""

If you're committing a PR that is just a small typo fix or a README change, you can force
Jenkins not to build your commit by adding [skip ci] below the message body. For example:
.. code-block:: console

    fix(README): typo

    It's spelled tomato, not tomatoe.

    [skip ci]
