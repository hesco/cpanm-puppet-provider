cpanm-puppet-provider
=====================

NOTICE:  
    This is an invitation to collaborate and does not 
    currently offer any useful functionality.  

This project seeks to contribute to the puppet code base 
a new native cpanm provider for Package['perl::module'] 
resources.  I develop primarily in perl, am busy these 
days bringing my infrastructure under puppet management, 
have almost no expereince with ruby and welcome collaborators.  

Jump in, the water is fine.  

I imagine support for installing 

  * a package name 
  * a url to a tarball
  * a carton cpanfile
  * a pinto stack
  * to a locallib

Somehow the native puppet package resource interface 
would need to be extended to account for all of that.  

    package { 'resource title':
      name              => # (namevar) The package name.  This is the name that the...
      ensure            => # What state the package should be in. On...
      adminfile         => # A file containing package defaults for...
      allowcdrom        => # Tells apt to allow cdrom sources in the...
      category          => # A read-only parameter set by the...
      configfiles       => # Whether configfiles should be kept or replaced.  
      description       => # A read-only parameter set by the...
      flavor            => # OpenBSD supports 'flavors', which are further...
      install_options   => # An array of additional options to pass when...
      instance          => # A read-only parameter set by the...
      platform          => # A read-only parameter set by the...
      provider          => # The specific backend to use for this `package...
      responsefile      => # A file containing any necessary answers to...
      root              => # A read-only parameter set by the...
      source            => # Where to find the actual package.  This must be...
      status            => # A read-only parameter set by the...
      uninstall_options => # An array of additional options to pass when...
      vendor            => # A read-only parameter set by the...
      # ...plus any applicable metaparameters.
    }

In a #perl-help conversation, ilmari asserts that cpanm 
is a module installation tool, not a real package manager.  

- Wednesday October 30 2013, 01:20:14 PM -

hesco: Any other puppet using perl developers out there frustrated 
by the lack of a native cpanm provider for package resources in puppet?  
I just ordered "Puppet Types and Providers".  Should be here in two weeks.  
I'm thinking it would be nice to contribute back a new provider to the 
puppet code base to serve our needs as perl developers and as sysadmins 
who have to manage perl code bases.  I would love to collaborate with 
others on such a project.  If interested, please drop me a line at: 
hesco@campaignfoundations.com .  Thanks. 

ilmari_: perl has no package manager, so it's hard 
for puppet to drive it

hesco: ilmari_: how is it you distinguish a package 
manager from a tool like cpanm ?

hesco: or to put that another way: what would a 'real' 
package manager provide which cpanm does not?

ilmari_: uninstallation and querying of installed packages, 
for starters

ilmari_: cpanm does write some metadata to 
$archlib/.meta/$dist-$version/, but that's only a start

ilmari_: (nothing uses that, afaik)

ilmari_: cpanm is a package installer, not a package manager

hesco: Is there any perl native means for uninstalling a 
package short of rm -rf on it?

ilmari_: rm the files in the .packlist

- 04:01:35 PM -

alnewkirk: hesco: 
    https://metacpan.org/pod/release/XAICRON/App-pmuninstall-0.30/bin/pm-uninstall

hesco: Thanks alnewkirk: will check that out

hesco: very nice interface.  Have you tested its use?

hesco: I'll add this to the README and plan to test this.  

hesco: If its working through the MANIFEST in the .packlist as ilmari_ suggested 
and we can sort out how to permit the query of installed packages we start to 
approximate minimal package manager functionality and perhaps can package up 
something to serve that purpose  on the perl side for perl system admins; which 
can then be wrapped as a puppet provider.  This is encouraging.

hesco: alnewkirk: quick skim of this is quite encouraging:
https://github.com/xaicron/pm-uninstall/blob/master/lib/App/pmuninstall.pm  

hesco: it seems to include methods useful for package query functions
as well 

Further on the question of querying an installation about its installed 
packages, this evening I finally removed perlbrew and installed plenv, 
an item which has been on my to-do list for several months now.  

plenv list-modules 

will provide a list of modules installed in the current perl
installation.  To that functionality I would want to add a 
means for querying for: 

    module version              # $VERSION of installed module
    module version-latest       # latest version available upstream
    module installation path    # absolute path to installation 
    module packlist-path        # absolute path to .packlist
    module deps                 # dependencies on perl modules 
    module deps-external        # dependencies on non perl resources 
    module deps-testing         # dependencies used for testing
    module deps-build           # dependenccies used for building module
    module provides-commands    # what cli tools are provided 
    module provides-modules     # what modules are provided 
    module provides-methods     # what public methods are exposed
    module provides-functions   # what public functions are exposed
    module author               # principal author(s)
    module maintainer           # and their contact information 
    module release-date         # for the current version
    module cpan-url             # what it says 
    module metacpan-url         # self-explanatory
    module test-results         # url to published smoke test results
    module repo                 # url for a publically accessible vcs repository
    module issues               # open bug count and 
                                # preferred means for issue tracking
    module irc                  # server and channels supporting 
                                # module users or development

; Thanks to alnewkirk on #perl-help channel
; http://pastie.org/8443394

    usage: 

    profile::utility::cpan { 'Dancer':
        using => 'cpanm',
        module => 'Dancer'
    }

    # cpan dependency installation

    define profile::utility::cpan (
        $using  = 'cpan',
        $perl   = 'perl',
        $module = undef
    )
    {
        exec { "cpan:${module}":
            command => "${using} ${module}",
            onlyif  => "${perl} -e \"!eval q{require ${module}}?exit(0):exit(1)\""
        }
    }

I have already started adapting this in my own puppet manifests 
to use cpanm and locallib

Although I have not yet tested this, and it seems (with the exception 
of a single script) to come without documentation, it also seems like 
a useful building block for what I have in mind:

	https://github.com/DrHyde/CPANdeps

Without any meaningful code review, and on first glance, this project 
appears to provide a web front end to a database of perl dependencies 
built by the scripts it provides.  But I suspect that were we to disentangle 
the user interface from the underlying functionality, that it might 
prove useful.  24pullrequests.com reports of this project: "Given a module 
name, this service will show you its dependencies along with a summary of 
their test results from the CPAN testers."

