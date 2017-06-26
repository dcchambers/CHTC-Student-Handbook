**CHTC User Mapping Guide**

  * The goal of user-mapping is to account for all of the usage hours in the
  various HTCondor pools at the UW Campus, including the CHTC pool. Ideally,
  each user should have a "project" they are working on. We
  will "map" their username to that project so we can keep track of what
  professors and research groups are using HTCondor and CHTC
  resources.

1. Open up one of the usage reports at: https://monitor.chtc.wisc.edu/uw_condor_usage/
  * I typically use the 90-day report available at https://monitor.chtc.wisc.edu/uw_condor_usage/usage90.shtml
    * The first spreadsheet on the page has a list of all projects and their
    usage hours across various pools.
    * The second spreadsheet is a list of every user, what project they are
    working on, and their usage hours.
2. SSH into monitor.chtc.wisc.edu
3. Search for "other" on the usage report website.
  * The first "other" on the page is in the project spreadsheet. This is a total
  number of unmapped usage hours.
  * The subsequent "other" listings on the page should be individual users that
  aren't yet mapped to a specific project.
4. Find a user with the project "other". We will have to figure out what project
this user is working on and then update the mapping.
5. Search for the user in the userapp.
  * Log in to https://wwwapps.chtc.wisc.edu/chtcuser/
  * Search for the user, if they have a project listed, you can go ahead and map
  them (skip to step **9** below).
6. If the user is not listed in the userapp, you will have to do some detective
work to try and figure out what project/group/professor they are working with.
  * Their username is often times a combination of their first and last name, or
   just their last name with a first initial. Be creative!
  * Check for them in the WIDMIR employee directory.
    * https://morgridge.org/directory/
    * https://wid.wisc.edu/about/directory/
  * Use Google.
    * Searching for their username and "wisc" or "@wisc.edu" is often a good way
    to find people.
  * Search for them on the UW Directory.
    * http://www.wisc.edu/directories/
  * Email them.
    * If you figure out what person the username belongs to, send them an email,
    explain that you work with the CHTC, and ask them what group or professor
    they work with so that you can map their usage hours.
7. If a username ends with "\_hep" you will have to email Dan Bradley and ask
him what group that person works with.
8. Other cases.
  * If a username ends with "\_discovery" and you can't find them online, you
  can map that user as "discovery".
9. **Map the User**

  * On monitor.chtc.wisc.edu (or any CHTC server) run
    * `$ condor_userprio --allusers | grep <username>` to figure out what domain
    the user is logged in from.
  * Once you have that (it will typically be something like `<user>@chtc.wisc.edu`)
  we can add them to the mapping script.
    * Open up map_glow_uesrs.py in /opt/condor_usage.
      * `$ cd /opt/condor_usage`
      * `$ sudo vim map_glow_users.py`
    * Add the username and project to this file. Do a search for the project name
    and add them in alphabetical order to that project grouping. You can create a
    new project if needed. Map their complete login domain (eg 'dakota@chtc.wisc.edu')
      * The mapping for me (Dakota) should look like: `CHTC       dakota@chtc.wisc.edu`
        * maintain the same whitespace as the previous entries.
    * Save and quit the file.
  * Once you have completed the mapping, you should be able to run the script.
  If it completes, you mapped the user successfully.
    * `$ python map_glow_users.py`
    * You can also use a username as an argument and it will print out what project
    the user is mapped to.
    * `$ python map_glow_users.py dakota@submit.chtc.wisc.edu`
      * Output: `>> dakota@submit.chtc.wisc.edu CHTC`
