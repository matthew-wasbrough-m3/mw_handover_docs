# Clinaccess <!-- omit in toc --> 

Clinaccess is the system for authentication and tracking that was developed for multi-community use prior to oauth. While it is still used for the multi community authentication on certain design builds, the main location it is likely to be encountered for general dev work is in the forum where it acts as the main authentication.

## Contents <!-- omit in toc --> 
- [Apps that make up Clinaccess](#apps-that-make-up-clinaccess)
  - [Github](#github)
  - [Gitlab](#gitlab)
- [Extra Docs](#extra-docs)

## Apps that make up Clinaccess

### Github

[clinaccess](https://github.com/m3europe/clinaccess)

This is a Java app connecting to a postgres database. Its purpose is to store syndication keys and community access details for any community implementing clinaccess authentication and tracking. The clinaccess functions (discussed later) are implemented in two files that are hosted by this app and the tracking implmented by these files is done within the app into the postgres database. This data is subsequently downloaded to the MSSQL database (as described later M3.Global.Datasync) for viewing in the .NET Clinaccess Dashboard (described later).

Live url: https://clinaccess.eu.m3.com/admin
Integ url: https://clinaccess-global.integration.intl.eu.m3.com/admin

Both of these require authentication, the user is admin and the password can be found in gogs ([here on integ](https://gogs.intl.eu.m3.com/integration/clinaccess-global/src/master/tomcat-users.xml)).

### Gitlab

[Dnuk.Api.ClinAccess](http://gitlab.internal.doctors.net.uk/Dnuk/Dnuk.API.ClinAccess)
[M3.Global.Api](http://gitlab.internal.doctors.net.uk/Dnuk/M3.Global.Api)
  - M3.Global.ClientAccess
  - M3.Global.DataSync

Although there are two projects there are three parts that are all .NET. The stand alone part (Dnuk.Api.Clinaccess) is an api for determining the logged in status of the user and for passing profile information from Doctors.net.uk to the Java clinaccess app discussed above. The endpoints in this app are also used by the OAuth Bridge allowing DNUK to act as an OAuth enabled community for Marvin. The second project (M3.Global.Api) has two parts, M3.Global.ClientAccess and M3.Global.DataSync. 

The M3.Global.DataSync has been process was written up last time I worked on it. The documentation is at http://gitlab.internal.doctors.net.uk/Dnuk/M3.Global.Api/tree/master/src/M3.Global.DataSync.

M3.Global.ClientAccess is a .Net project know by the comms team as the Clinaccess Dashboard. The login page is https://www.doctors.net.uk/ClinAccess/dnuk. Once logged in an admin has the ability to create accounts for users (internal and external) to access either the dashboard itself to get the stats about the campaign or to access a stripped down version of the site to enable the pull in of the dnuk header and footer into the campaign for testing purposes. *N.B. Known issue here: if you create a test user to access the site then you also need to manually set a altemailverification date in the user_tbl. /N.B.* Clinaccess has been setup on https://clinaccess.eu.m3.com/tests/test.html and https://clinaccess-global.integration.intl.eu.m3.com/tests/test.html to show the header and footer being pulled in in practice. 

If you need to test out a partner header and footers then you can use the live site above after logging into the partner site. This isn't a fool proof list of logins but the ones I have used in the past are at https://github.com/m3europe/nih_guest_accounts.

## Extra Docs
