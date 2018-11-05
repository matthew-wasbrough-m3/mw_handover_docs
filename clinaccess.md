# Clinaccess <!-- omit in toc --> 

Clinaccess is the system for authentication and tracking that was developed for multi-community use prior to oauth. While it is still used for the multi community authentication on certain design builds, the main location it is likely to be encountered for general dev work is in the forum where it acts as the main authentication.

## Contents <!-- omit in toc --> 
- [Apps that make up Clinaccess](#apps-that-make-up-clinaccess)
  - [Github](#github)
  - [Gitlab](#gitlab)

## Apps that make up Clinaccess

### Github

[clinaccess](https://github.com/m3europe/clinaccess)

This is a Java app connecting to a postgres database. Its purpose is to store syndication keys and community access details for any community implementing clinaccess authentication and tracking. The clinaccess functions (discussed later) are implemented in two files that are hosted by this app and the tracking implmented by these files is done within the app into the postgres database. This data is subsequently downloaded to the MSSQL database (as described later) for viewing in the .NET Clinaccess Dashboard (described later).

Live url: https://clinaccess.eu.m3.com/admin
Integ url: https://clinaccess-global.integration.intl.eu.m3.com/admin

Both of these require authentication, the user is admin and the password can be found in gogs ([here on integ](https://gogs.intl.eu.m3.com/integration/clinaccess-global/src/master/tomcat-users.xml)).

### Gitlab

[Dnuk.Api.ClinAccess](http://gitlab.internal.doctors.net.uk/Dnuk/Dnuk.API.ClinAccess)
[M3.Global.Api](http://gitlab.internal.doctors.net.uk/Dnuk/M3.Global.Api)
  - M3.Global.ClientAccess
  - M3.Global.DataSync

Although there are two projects there are three parts that are all .NET. The stand alone part (Dnuk.Api.Clinaccess) is an api for determining the logged in status of the user and for passing profile information from Doctors.net.uk to the Java clinaccess app discussed above. The endpoints in this app are also used by the OAuth Bridge allowing DNUK to act as an OAuth enabled community for Marvin. The second project (M3.Global.Api) has two parts, M3.Global.ClientAccess and M3.Global.DataSync