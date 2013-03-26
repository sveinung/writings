Infrastructure as Code: Frameworks vs the shell
===============================================

For provisioning infrastructures nowadays there are a wealth of fancy frameworks ready to be used. Among these are [Puppet](https://puppetlabs.com/), [Salt](http://saltstack.com/)
and [Chef](http://www.opscode.com/chef/). For many projects, however, these tools might be overkill. In many cases, simply applying bash might suffice. In more complex cases, using a little more
powerful language like ruby or python might be justified.

This blogpost will try to map out a provisioning maturity model, which can be used for reasoning about your projects specific needs.

Use the shell when:
- Simple resource graphs
- Few resources
- Few provisioned nodes

When to use bash (or some other shell)
--------------------------------------

Simple graphs, no need for templating

When to use a scripting language (Ruby, Python, etc)
----------------------------------------------------

Need templating

When to use frameworks
----------------------

Complex graphs, need efficient tracing of inter-resource dependencies

