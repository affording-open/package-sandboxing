Software package level sandboxing and isolation
===============================================

Status: drafting \
Git origin: https://gitlab.com/affording-open/package-sandboxing \
Git origin backup: https://github.com/affording-open/package-sandboxing

This collects ideas and discussions for isolation of software packages from
each other and sandboxing or restriction of their capabilities on the system.

While some package managers already offer this, many more do not and also do
not offer an easy way to take package initiated actions like install scripts
with restricted capabilities.

The xz compromise did not use this route, but it was a case of malicious build
scripts not being caught during the software distributions process. While
library sandboxing would have prevented it, there would still then be one other
way open for a package with a malicious build script that provides this
library. It could influence the package build so that the resulting package
runs malicious code as root on installation. There are many packages with too
few people to review all of them in sufficient detail. But most packages
luckily do not need to run code as root.

It seems that for at least for years classical \*nix package managers will
remain popular. Possible replacements that have full isolation are mentioned
below. But they need further implementation work to fulfill more use cases.

The choice thus is between implementing this sandboxing in more package
managers or leaving their users unprotected.

Beware: Security boundaries on much popular hardware and software are leaky, so
sadly sandboxing is sometimes not enough to run untrusted code. Distinct
computers can help, but even there constructing physical isolation to prevent
side channels is also challenging. This tries to add a security boundary to
existing designs, which usually has worse outcomes than designs crated with
such boundaries in minds. However it might be worthwhile here as moving to a
new design might not succeed soon enough.


## Strategies for system package managers

If you know of any examples of designs or implementations, please contribute.

Some ideas:

### Capabilities during installation

Package manageres that allow multiple repositories need to add a mechanism to
specify which additional capabilities gets passed to each repo. There needs to
be a way to specify in a repo which capabilities get passed to which package.
It is also necessary to allow user overrides of this.

Passing capabilities to packages might be declared like dependencies are often
declared with name and version selectors, however these need to be scoped to
repos. A good way to pass these across repos needs to be thought up.

The package manager enforces capabilities of packages to put specific or any
files into specific directories.

There needs to be a way for packages to pass the capability of puting files
into directories they created to specific other or all packages.

A danger of this remains is that certain tools execute from files when the user
doesn't expect it.

Many package managers support running scripts from a package during
installation as root. One way that is already widely in use is to replace these
is by declarative actions that can be safe.

An example of such actions are creating a system user, and installing,
enabling, and starting a service that then is run as that user. While
traditionally this was often done by a bash script, on some systems this can
now be done by creating a file in a known location that defines which users to
create, and another file that defines the service.
TODO link to specifc docs of this

This can also be adapted to capabilities. A capability to add services that run
as a specific user can be passed to other packages. As services are declared in
files, this could be implemented as a more constrained file creation capabiliy
that also restricts the file content.

### Verify packages after build

As an alternative, for declarative information like file content and path it is
possible to verify the rules from the previous heading after the build instead
of at install time.  This needs to be isolated from the build, so it can not
influence the verifier.

The downside of this is that it would need multipe verifiers to not make one a
single point of failure. This could be done with signed assertions and the
package manager verifying those. The package manager still needs to keep track
of which repos are passed capabilities, to check against the verified ones.

Taken together it seems the idea in this header would be more complicated and
more work than the previous one.


## Status of specific packages managers

Where discussion, planing, progress on implementation happens, links are welcome.

### System package managers

* TODO
* Image based systems avoid this by entirely avoiding system package managers.
  * Android
  * TODO image based Linux distributions

### User level packer managers

Package managers that only install packages for users to use, but not software
that makes up more core parts of the systems have it easier, as they do not
have to deal with software that needs higher capabilities.

* Flatpak has full isolation
* Podman and other container implementations have full isolation
* Android has full isolation

### Library / Language package managers

* rpm https://github.com/rpm-software-management/rpm/discussions/3030
* TODO


## Related sandboxing strategies

In general capabilities based designs help sandboxing.

TODO extend, links

Proper sandboxing additionaly requires that sandboxing applications is
supported by the shell that launches programs. While that is out of scope here,
there is a collection of projects that deal with this.

It is additionally useful when applications use sandboxes for parts of their
running code.

* Linux
  * seccomp
  * landlock

There are also general library sandboxes and capability based programming
languages.

It is also possible to statically verify code is constrained by a capability
design.

