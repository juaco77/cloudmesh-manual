
# ######################################################################
# Arguments
# ######################################################################
# {'--kind': 'rst', '--noheader': False, 'COMMAND': 'open'}
# ######################################################################

open
====

::

    Usage:
        open chameleon baremetal tacc
        open chameleon baremetal uc
        open chameleon vm
        open chameleon openstack
        open FILENAME
        open doc local
        open doc


    Arguments:

        FILENAME  the file to open in the cwd if . is
                  specified. If file in in cwd
                  you must specify it with ./FILENAME

                  if the FILENAME is doc than teh documentation from the Web
                  is opened.

    Description:

        Opens the given URL in a browser window.

        open chameleon baremetal tacc
           starts horizon for baremetal for chameleon cloud at TACC

        open chameleon baremetal uc
            starts horizon for baremetal for chameleon cloud at UC

        open chameleon vm
            starts horizon for virtual machines

Timer: 0.0124s (man --kind=rst open)
