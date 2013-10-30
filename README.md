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

# Thanks to alnewkirk on #perl-help channel
# http://pastie.org/8443394
# usage: profile::utility::cpan { 'Dancer': using => 'cpanm', module => 'Dancer' }
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

