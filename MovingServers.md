# Moving to different servers

It is important to exercise care when applying migrations to the environment servers.

Do not permit the migration script for an older version to run after a more recent version has been applied.

Using an automation server can help minimize the possibility of this occurring by being a central authority on the triggering and applying of migrations to the dev, test and prod database instances.

Once the automation server is in place, moving the database version forward will be similar to how we communicate the release version of our web applications: by moving the Mercurial bookmark forward.

Once the bookmark has been set to the desired *newer* version in the Migration repository, the bookmark will be updated on the central repo to match by running the `push` command.

With the bookmark moved to the desired changeset, the automation server (Jenkins) will pull down the change, and run the migration tool for the desired version and database.

## Specifics on RoundhousE tool operation inside of Jenkins

The Database Migration Automation will use an "Execute Windows batch command" Build Task. The command is specified in Jenkins and will work much in the way the LOCAL.DBDeployment.bat script does.

Items worth noting:

1. The RoundhousE exe (`rh.exe`) has been placed in the root of the Jenkins' server's D: drive, making it unnecessary to add it to the path as long as the command specified contains the full path.
2. As will be done in the LOCAL.DBDeploy.bat script, the Mercurial command line program is called supplying the revision id (the hash) as a value for the migrator to use.
3. the environment value supplied to RoundhousE is optional at this point the docs state: "This allows RH to be environment aware and only run scripts that are in a particular environment based on the naming of the script. **LOCAL**.something.**ENV**.sql would only be run in the `LOCAL` environment. Defaults to `LOCAL`" --- not sure if we want to use this or supply this right away...
4. All the migrations run by Jenkins need to be done with the `/transaction` switch, this insures that the `RoundhousE.Version` table contains only successfully applied entries.
5. `/disableoutput` from the docs: "Disable output of backups, items ran, permissions dumps, etc. Log files are kept. Useful for example in CI environment."
6. Since the task failure is based on the last command's exit code, it's important that the `rh.exe` run is the last item in the task.

***

[Back to table of contents](README.md)
