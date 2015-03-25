Jenkins CI
===========================================
`Jenkins <https://jenkins.libtoxcore.so>`_ is our cross platform/architecture build system, it deals with Windows libraries on Fedora boxes to Qt on OS X boxes, and everything in between

What does Jenkins do?
---------------------
Here's a rough overview of what it does in list format, its been simplified for your sake

* Clone repos from github pushes
* Verify the PGP commit chain on the repo
* Archive the files and make them open to other jobs
* trigger downstream projects to build
* Move artifacts to other machines used for building things
* build all downstream jobs who's upstream got pushed (when the repo who got a commit was a library)
* PGP sign the lowest downstream job and archive it, making it downloadable