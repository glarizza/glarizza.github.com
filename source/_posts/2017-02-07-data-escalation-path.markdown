---
layout: post
title: "Data Escalation Path: Does it go in the Profile or Hiera?"
date: 2017-02-07 11:39:13 -0800
comments: true
categories: ["hiera", "data", "workflow", "r10k", "code manager"]
---

**NOTE**: This post is lacking a lot of the humor and playful tone I normally
use when writing posts because it was originally designed for another purpose.
I'm posting it in its entirety here because I consider the information useful
and want to get it out to people who ask me about this topic. Insert some Harry
Potter references if it gets a bit dry...


# Data Escalation Paths

Writing reusable Puppet manifests involves a combination of static and dynamic
data, but the decision of whether to "hardcode data", use a variable, set
a parameter, do a Hiera lookup, or do a combination of the above is one of the
most frequently asked questions by Puppet users. This guide will both describe the
available data lookup methods and advise on when it is best to "escalate"
based on the requirements at hand.


## What is "Data" and what is "Code"?

This issue about what constitutes "data" is the first wrinkle in devising
a data escalation path. [The Puppet docs page on Roles and Profiles][rnp]
does a great job of describing the difference between a component module and
a profile.

To quickly summarize: a component module is a general-purpose module
designed to model the configuration of a piece of technology (e.g. apache,
tomcat, ntpd), and a profile is an organization-specific Puppet module that
describes an organization's IMPLEMENTATION of a piece of technology. For
example, an apache profile that an organization creates might use the official
Puppet apache module to install and configure apache, but the profile might
also contain resources for an organization's SSL certificates or credentials that
would be layered atop of the configuration that the Puppet apache module
provides. The resource(s) modeling the SSL certificate(s) are ONLY necessary for
that organization, which is why they don't show up in the official Puppet
apache module.

By this account the offical Puppet apache module itself represents the "code",
or the generic and re-usable aspect of the configuration (as most good
component modules would), but the profile contains the
organizational/site-specific data that is fed TO the component module (or
"code") when that module is declared/used.  This separation - and the fact that
data can be represented within the same constructs as as code can - is
frequently a source of confusion/frustration for new Puppet users (and
especially for users with a background in object-oriented programming).

Data within a profile can come in different forms - a value to a parameter when
a class is declared, a variable assigned within the profile, or even a Hiera
lookup (explicitly done or through Puppet's automatic parameter lookkup). Since
it's all data, which method do you use? That's the question this document means
to answer.

A note for clarity: this document specifically covers data escalation paths
within profiles and NOT component modules. Unless explicitly noted, assume that
recommendations apply ONLY to profiles and not component modules (since
profiles represent site-specific data).

## Why an escalation path?

Seldom are things cut-and-dry within Puppet manifests. Instead of focusing on
whether something is in the "right" place, it's better to think about the ways
that complexity can be added to solve problems. You can absolutely put
everything you would consider data inside Hiera, and that would immediately
provide you a way to handle most corner cases, but then the legibility of
your Puppet manifest suffers when you have to jump back to Hiera every time you
need to retrieve a data value (and currently that's a very labor-intensive
thing to do if you don't have direct access to the Puppet masters). Consider
every way to handle data another tool in your Puppet belt, and certain tools
are better at solving problems that others. This document, then, is a rundown
on the pros and cons of every tool.

## "Hardcoding" variables

The word hardcoding is wrapped in quotes because traditionally the term evokes
negative connatations. When we use the word "hardcoding" we mean
directly-editing an item within a Puppet manifest without assigning a variable.
In the below example if you opened up the Puppet manifest and changed the owner from
'root' to 'puppet', then that would be considered hardcoding the value.

```puppet
file { '/etc/puppetlabs/puppet/puppet.conf':
  ensure => file,
  owner  => 'root',
  group  => 'root',
  mode   => '0644',
  source => 'puppet:///modules/mymodule/puppet.conf',
}
```

Hardcoding has a negative connotation because typically when someone would
hardcode a value in a shell script it represented a workaround where a data
item is injected into the code (and mixing data and code means that your code
is no longer as generic/extensible as it once was).  That concern is still
valid with Puppet: if you open up the official Puppet apache module and
change/add a site-specific value within that component module, then you ARE
mixing data with code. If, instead, you edit the apache profile for your
organization and change a value there, then you're changing site-specific data
to something that is already considered site-specific. The difference is that
the official Puppet apache module is desiged to be extensible and used where
necessary, but the profile is only meant to be used by your
organization/site/group.

Hardcoding a value is the easiest "tool" to understand: something was
previously set to one value, and now it's set to another value. It's also the
easiest change to implement: simply change the value and move along. If done
correctly, someone could change the value without needing to understand the
Puppet DSL (Domain Specific Language, i.e. the rules governing Puppet code in
a Puppet manifest). Finally, because it's simply text, a hardcoded value cannot
be overridden and the value is the same for all nodes.


### Pros

* The easiest 'tool' to understand: something was changed from one value to
  another
* The easiest change to implement

### Cons

* If you hardcode the *same* value in multiple places, then changing that value
  requires multiple individual changes.

### Recommendations

* Hardcode values ONLY when their values apply to EVERY NODE being
  managed by Puppet
* Hardcode values within a manifest that only occur once - if it occurs more
  than once within a manifest use a variable

## Assigning a variable

The next logical progression after hardcoding a value is to create and assign
a variable within a Puppet manifest. Assigning a variable is useful when a value
is going to be used in more than one place within a manifest. Because
variables within the Puppet DSL cannot be reassigned, and because variables
within a manifest cannot be assigned/changed by Hiera, variables are considered
private to the implementation and thus can only be changed by users with access
to change Puppet manifests (versus having Puppet Enterprise Console access or
access to Hiera data).  More simply: variables assist writers of Puppet code
versus users of Puppet code.

```puppet
$confdir = '/etc/puppetlabs/puppet'

file { "${confdir}/puppet.conf":
  ensure => file,
  owner  => 'root',
  group  => 'root',
  mode   => '0644',
  source => 'puppet:///modules/mymodule/puppet.conf',
}

file { "${confdir}/puppetdb.conf":
  ensure => file,
  owner  => 'root',
  group  => 'root',
  mode   => '0644',
  source => 'puppet:///modules/mymodule/puppetdb.conf',
}
```

### Pros

* Assigning a variable provides a single point within a manifest where data can
  be assigned/changed. This means if the value of the variable needs to be changed,
  it can be done in ONE place within the manifest (i.e. where the variable has
  been assigned), versus hardcoding which requires you to find and replace for
  all instances within a manifest
* Assigning a variable within the DSL is visible to anyone reviewing the Puppet
  manifest, which means that you don't need to flip back and forth between
  Hiera and Puppet to look up data values

### Cons

* The value applies to EVERYONE - it must be changed if a different value is
  desired, and that change applies to everyone
* No ability to override a value

### Recommendations

* Assign a variable when a value shows up more than once within a manifest and
  when that value applies to EVERY node


## CONDITIONALLY assigning a variable

The previous section on assigning a variable recommends that variables are only
used when a value applies to EVERY node, but there is a way to work around
that: conditional statements. Conditional statements in the Puppet DSL (such as
"if", "unless", "case", and the selector operator) allow you to assign
a variable once, but assign it differently based on some condition. Using the
previous example of Puppet's configuration directory, let's see how that would
be assigned differently based on the system's kernel fact:

```puppet
# Using the "selector" conditional statement to assign the $confdir variable
# based on the kernel fact.  https://docs.puppet.com/puppet/latest/lang_conditional.html#selectors

$confdir = $facts['kernel'] ?
  'linux'   => '/etc/puppetlabs/puppet',
  'windows' => 'C:\\ProgramData\\PuppetLabs\\puppet\\etc',
  default   => fail("${facts['kernel']} is not supported"),
}

file { "${confdir}/puppet.conf":
  ensure => file,
  owner  => 'root',
  group  => 'root',
  mode   => '0644',
  source => 'puppet:///modules/mymodule/puppet.conf',
}

file { "${confdir}/puppetdb.conf":
  ensure => file,
  owner  => 'root',
  group  => 'root',
  mode   => '0644',
  source => 'puppet:///modules/mymodule/puppetdb.conf',
}
```

The reason conditionally assigning a variable has its own section is because
when people think about "data" and "choice" with Puppet, they USUALLY think
of Hiera. Hiera is an excellent tool for conditionally assigning a value based
its internal hierarchy, but what if the conditional logic you need to use
doesn't follow Hiera's configured hierarchy? Your choices are to edit Hiera's
hierarchy to add the logic you need (which is disruptive to both your servers
and, potentially, individual Hiera lookups), or use conditional logic within
the DSL. Since we're talking about an escalation path, conditionally assigning
a variable is the next logical progression when complexity arises.

### Pros

* Values can be assigned based on whatever conditional logic is necessary
* Values are assigned within the Puppet DSL and thus are more visible to Puppet
  code reviewers (versus needing to jump into Hiera)
* Reusability remains - the variable is assigned once and used throughout the
  manifest

### Cons

* Variables still cannot be reassigned/overridden
* Conditional logic can grow to become "stringy" and overly-complex if left
  unchecked
* Conditional logic is syntax-heavy and requires knowledge of the Puppet DSL
  (i.e. not something easily done by people who don't know Puppet)

### Recommendations

* Use conditional logic to assign a variable when conditional logic is
  necessary to assign the variable, the conditional logic isn't overly complex,
  and the logic is different from the Hiera hierarchy
* Use conditional logic when visibility of the data values within the Puppet
  DSL is a priority

## Hiera lookups and class parameters

Puppet's data lookup tool *du jour* is Hiera, and Hiera is an excellent way to
model data in a hierarchical manner based on layers of business logic.
Demonstrating how Hiera works is the easy part, implementing it (and knowing
when to do Hiera calls) is another story. Before we get there though, it's
important to understand that Hiera lookups can be done ad-hoc [through the use
of the `hiera()` or `lookup()` functions][lookup], or through the [automatic
class parameter lookup functionality][apl]. The link will give you a detailed
explanation, but, for brevity, if a class is declared and a value is not
explicitly assigned for any of the class's parameters, Hiera will automatically
do a lookup for the full parameter name (i.e. if the class is called '`apache`'
and the parameter is called '`port`', then Hiera does an automatic parameter
lookup for `apache::port` should the `port` parameter not be explicitly assigned).

We'll get back to automatic parameter lookups in a second, but for now let's
focus on explicit lookups. Here's an example using lookup functions:

```puppet
$apache_port    = hiera('apache_port')
$apache_docroot = lookup('apache_docroot')
```

Explicit lookups using one of the above functions are easier to see/understand
when you're new to Puppet because [the automatic parameter lookup][apl]
functionality is relatively hidden to you (should you not be aware of its
existence). The downside is that you need to search through Puppet manifests to
find all instances of these explicit lookups.

### Explicit lookup pros:

* No need for conditional logic since Hiera is configured independently, simply
  do a lookup for a value and assign it to a variable
* Usage of a lookup function is a visible indicator that the data lives outside
  the DSL (in Hiera)

### Explicit lookup cons:

* Loss of visibility - the data is inside Hiera's hierarchy and determining the
  value requires invoking Hiera in some manner (as opposed to simply
  observing a value in the DSL)
* If the lookup you want to perform doesn't conform to Hiera's existing
  hierarchy, then Hiera's hierarchy will need to be changed (which is disruptive)

### Explicit lookup recommendations:

* We generally discourage explicit lookups within the implementation of
  a profile in favor of exposing a parameter. If there's a case where you would
  NOT want to expose a parameter and make an API guarantee but you DO want/need
  Hiera, then an explicit lookup is recommended (and be sure to document the
  intent for posterity)

There's also one more very important distinction to make...

### Class parameters and API vs internal logic

When building a profile, the implementation (i.e. anything between the open and
closing curly braces {} ) is considered to be private. This means that there
are really no guarantees around what's happening as long as the technology is
configured properly in the end. Class parameters are considered to be part of
the profile's API, and thus there's a guarantee that existing parameters won't
be removed or have their functionality changed within a major release (if you
follow semantic versioning). More specifically, exposing a parameter indicates
to your Puppet code users that this is something that can be set/changed. Think
of computer hardware and the differentiation between phillips-head screws and
Torx screws: the phillips-head screws usually mean that customer intervention
is allowed, much the same way as parameters indicate data values that can be
changed, and Torx screws usually mean that customer intervention is disallowed,
much the same way as variables within a profile cannot be reassigned or
overridden.

One final word on class parameters - the Puppet Enterprise Console is able to
discover class parameters and present them visually, and it does this because
Puppetserver has an API that exposes this data. This means that parameters and
classes can be queried and enumerated. Explicit Hiera lookups are not
discoverable in the same way (you will need to search through your codebase
manually).

Because these two lookups have two fundamentally different purposes, we're
treating their usages as separate.

### Class parameter lookup pros:

* Signals to users of Puppet code that this data item is configurable
* Allows the value to be assigned either by the Puppet Enterprise Console (or
  other configured ENC) or Hiera
* Classes and parameters are discoverable through Puppetserver API

### Class parameter lookup cons:

* Automatic parameter lookup is unexepected if you don't know it exists
* Loss of visibility - the data is inside Hiera's hierarchy and determining the
  value requires invoking Hiera in some manner (as opposed to simply
  observing a value in the DSL)
* Each parameter is unique, so even if multiple profiles expose a parameter of
  the same name that requires the same value, there needs to be a value in
  Hiera for each unique parameter

### Class parameter recommendations

* Expose a parameter if/when you require the logic in Hiera's hierarchy to determine
  the value of a data item
* Expose a parameter if/when you need to override the value using the Puppet
  Enterprise Console (or other configured ENC)
* Expose a parameter to indicate that this part of the profile is configurable
  to users of Puppet code

## Summary

This document was designed to start with the least amount of complexity and
work its way up. In the end, it's all about choosing the right data mechanism
for the problem. If you're a more visual learner, the flowchart below will help
clarify when to use each method:

[{% img left /images/data/data_escalation_path %}](http://garylarizza.com/images/data/data_escalation_path.png)

[rnp]: https://docs.puppet.com/pe/latest/r_n_p_intro.html
[apl]: https://docs.puppet.com/hiera/latest/puppet.html#automatic-parameter-lookup
[lookup]: https://docs.puppet.com/puppet/latest/lookup_quick.html
