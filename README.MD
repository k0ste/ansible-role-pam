ansible-pam
=============

Deploy pam - Pluggable Authentication Modules for Linux.

Requirements
---------------

* Ansible 2.2;
* jmespath;

Example configuration
-------------------------

```yaml
---
pam:
# The service is typically the familiar name of the corresponding application:
# 'login' and 'su' are good examples. The service-name 'other' is reserved for
# giving default rules. Only lines that mention the current service (or in the
# absence of such, the other entries) will be associated with the given
# service-application.
- service: 'system-auth'
# The 'type' is the management group that the rule corresponds to. It is used to
# specify which of the management groups the subsequent module is to be
# ssociated with. Valid entries are:
#
# account:
# Performs non-authentication based account management. It is typically used to
# restrict/permit access to a service based on the time of day, currently
# available system resources (maximum number of users) or perhaps the location
# of the applicant user - 'root' login only on the console.
#
# auth:
# Provides two aspects of authenticating the user. Firstly, it establishes that
# the user is who they claim to be, by instructing the application to prompt the
# user for a password or other means of identification. Secondly, the module can
# grant group membership or other privileges through its credential granting
# properties.
#
# password:
# Required for updating the authentication token associated with the user.
# Typically, there is one module for each 'challenge/response' based
# authentication (auth) type.
#
# session:
# Associated with doing things that need to be done for the user before/after
# they can be given service. Such things include the logging of information
# concerning the opening/closing of some data exchange with a user, mounting
# directories, etc.
  types:
  - type: 'auth'
# The 'control' is the behavior of the PAM-API should the module fail to succeed
# in its authentication task. There are two types of syntax for this control
# field: the simple one has a single simple keyword; the more complicated one
# involves a square-bracketed selection of value=action pairs. Control:
#
# required:
# Failure of such a PAM will ultimately lead to the PAM-API returning failure
# but only after the remaining stacked modules (for this service and type) have
# been invoked.
#
# requisite:
# Like required, however, in the case that such a module returns a failure,
# control is directly returned to the application or to the superior PAM stack.
# The return value is that associated with the first required or requisite
# module to fail. Note, this flag can be used to protect against the possibility
# of a user getting the opportunity to enter a password over an unsafe medium.
# It is conceivable that such behavior might inform an attacker of valid
# accounts on a system. This possibility should be weighed against the not
# insignificant concerns of exposing a sensitive password in a hostile
# environment.
#
# sufficient:
# If such a module succeeds and no prior required module has failed the PAM
# framework returns success to the application or to the superior PAM stack
# immediately without calling any further modules in the stack. A failure of a
# sufficient module is ignored and processing of the PAM module stack continues
# unaffected.
#
# optional:
# The success or failure of this module is only important if it is the only
# module in the stack associated with this service+type.
#
# include:
# Include all lines of given type from the configuration file specified as an
# argument to this control.
#
# substack:
# Include all lines of given type from the configuration file specified as an
# argument to this control. This differs from include in that evaluation of the
# done and die actions in a substack does not cause skipping the rest of the
# complete module stack, but only of the substack. Jumps in a substack also can
# not make evaluation jump out of it, and the whole substack is counted as one
# module when the jump is done in a parent stack. The reset action will reset
# the state of a module stack to the state it was in as of beginning of the
# substack evaluation.
    control: 'required'
# module_path is either the full filename of the PAM to be used by the
# application (it begins with a '/'), or a relative pathname from the default
# module location: '/lib/security/'.
    module_path: 'pam_unix.so'
# module_arguments are a space separated list of tokens that can be used to
# modify the specific behavior of the given PAM. Such arguments will be
# documented for each individual module. Note, if you wish to include spaces in
# an argument, you should surround that argument with square brackets.
    module_arguments:
    - 'try_first_pass'
    - 'nullok'
  - type: 'auth'
    control: 'optional'
    module_path: 'pam_permit.so'
  - type: 'auth'
    control: 'required'
    module_path: 'pam_env.so'
  - type: 'account'
    control: 'required'
    module_path: 'pam_unix.so'
  - type: 'account'
    control: 'optional'
    module_path: 'pam_permit.so'
  - type: 'account'
    control: 'required'
    module_path: 'pam_time.so'
  - type: 'password'
    control: 'required'
    module_path: 'pam_unix.so'
    module_arguments:
    - 'try_first_pass'
    - 'nullok'
    - 'sha512'
    - 'shadow'
  - type: 'password'
    control: 'optional'
    module_path: 'pam_permit.so'
  - type: 'session'
    control: 'required'
    module_path: 'pam_limits.so'
  - type: 'session'
    control: 'required'
    module_path: 'pam_unix.so'
  - type: 'session'
    control: 'optional'
    module_path: 'pam_permit.so'
```
