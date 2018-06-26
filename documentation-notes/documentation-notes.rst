============================
How to Prepare Documentation
============================

*David Antos*

May 2018

Where
=====

ELIXIR Data Transfer documentation is published on
http://elixir-data-transfer-docs.readthedocs.io/. Its source codes are kept
on https://github.com/david-antos/elixir-data-transfer-docs. When changes
are pushed to *GitHub*, *Read the Docs* published version gets regenerated
automatically.

How
===

reStructuredText
----------------

The documentation is preferrably written in reStructuredText format. Using
Markdown is also possible, but it is much less standardised. For good
reasons to prefer reStructuredText, see
http://ericholscher.com/blog/2016/mar/15/dont-use-markdown-for-technical-docs/.

If you're new to reStructuredText, there are some docs for you:
  - `reStructuredText Primer <http://docutils.sourceforge.net/docs/user/rst/quickstart.html>`_ is a good start
  - `Quick reStructuredText <http://docutils.sourceforge.net/docs/user/rst/quickref.html>`_ for quick reference
  - `Full Specification <http://docutils.sourceforge.net/docs/ref/rst/restructuredtext.html>`_ is good when strange things happen
  - `reStructuredText and Sphinx Cheat Sheet <https://thomas-cokelaer.info/tutorials/sphinx/rest_syntax.html>`_ is a good and concise reference

Please note that reStructuredText is extremely picky to text
indentation. If you run into trouble, check your indentation first.

Necessary local software
------------------------

You'll need
  - git
  - any text editor
  - (optional but recommended) `Sphinx <http://www.sphinx-doc.org/>`_ will
    allow you to generate the documentation locally

Writing new documentation/article
---------------------------------

Kindly add a folder to the repository for a new article to keep it neat.
Write the documentation (using ``.rst`` file suffix is recommended). Make a
link to the documentation from ``index.rst``.

If you have Sphinx installed, you can prepare local preview of your docs.
Just run

::

  make html

(or a windows bat equivalent) in the main folder and point your browser to
``_build/html/index.html``. Or any other format you like.

Please note: to keep things simple, we use built-in Sphinx style for local
html output. It differs from the style on the *Read the Docs* site. The reason
is not to complicate things beyond necessary, you'd have to install the
*Read the Docs* style locally.

Tips and tricks
===============

If you're unlucky enough to have your documentation in Google Docs or any
other format (even lacking proper logical markup), you may try `Pandoc
<http://pandoc.org/>`_ to convert it.

Initial conversion of this group's documentation in Google Docs has been
produced by exporting to ``docx``, converting according to `Mpei's Blog
<https://peintinger.com/?p=365>`_ and heavily edited by hand.


Working with Git repository
===========================

For the time being, there is no urge for a strict “editorial process” to be
put in place. The core team members of data transfer may have write access
to the GitHub repository. Please keep in mind that committing to the
repository directly rebuilds the documentation on the public website. You
are therefore advised to push material suitable for public viewing (it
doesn't have to necessarily be finished, of course) and compilable. Check
your work locally or at least check the public website after pushing
changes. The branch-to-be-published is “master”.

Ask David or any other collaborator to add your GitHub identity to
the project.

If you are an outside contributor or if you feel you'd prefer your
documentation to be reviewed before publishing, use the “fork and pull”
model. Fork the repository, create a branch, do your stuff, and create a
pull request. Refer to
https://help.github.com/categories/collaborating-with-issues-and-pull-requests/
if you need to get familiar with the process.



Contacts
========

For write access to the *GitHub* repo, requirements to add plugins (keep it
reasonable, please), change the config of *Read the Docs*, please contact
David Antos (david (dot) antos (atsymbol) cesnet.cz).

