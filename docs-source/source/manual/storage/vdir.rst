
# ######################################################################
# Arguments
# ######################################################################
# {'--kind': 'rst', '--noheader': False, 'COMMAND': 'vdir'}
# ######################################################################

vdir
====

::

  Usage:
        vdir mkdir DIR
        vdir cd [DIR]
        vdir ls [DIR]
        vdir add [FILEENDPOINT] [DIR_AND_NAME]
        vdir delete [DIR_OR_NAME]
        vdir status [DIR_OR_NAME]
        vdir get NAME DESTINATION

  Arguments:
      DIR           a directory name
      FILEENDPOINT  location of file
      DIR_AND_NAME  path of file link
      DIR_OR_NAME   name of directory or link
      DESTINATION   directory to download to
      NAME          name of link

  Options:
      -f      specify the file

  Descripton:

      A virtual directory is explained in our NIST doecumentation. It
      contains a number of links that point to other storage services on
      which the file is stored. The links include the provider, the name
      of the profider and its type are identified in the
      ~/.cloudmesh4.yaml file.

      the location is identified as

         {provider}:{directory}/{filensme}

      A cloudmesh directory can be used to uniquely specify the file:

         cm:
            name: the unique name of the file
            kind: vdir
            cloud: local
            directory: directory
            filename: filename
            directory: directory
            provider: provider
            created: date
            modified: date

      vdir get NAME DESTINATION

         locates the file with the name on a storage provider,
         and fetches it from there.

Timer: 0.0151s (man --kind=rst vdir)
