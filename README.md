ansible-role-tripleo-collect-logs
=================================

An Ansible role for aggregating logs from TripleO nodes.

Requirements
------------

This role gathers logs and debug information from a target system and
collates them in a designated directory, `artcl_collect_dir`, on the localhost.

Additionally, the role will convert templated bash scripts, created and used by
TripleO-Quickstart during deployment, into rST files. These rST files are
combined with static rST files and fed into Sphinx to create user friendly
post-build-documentation specific to an original deployment.

Finally, the role optionally handles uploading these logs to a rsync server.

Role Variables
--------------

* `artcl_collect_list` -- A list of files and directories to gather from
  the target. Directories are collected recursively. Can include joker
  characters that bash understands. Should be specified as a YaML list,
  e.g.:

```yaml
artcl_collect_list:
    - /etc/nova
    - /home/stack/*.log
    - /var/log
```

* `artcl_collect_dir` -- A local directory where the logs should be
  gathered, without a trailing slash.
* `artcl_gzip_only`: false/true  -- When true, gathered files are gzipped one
  by one in `artcl_collect_dir`, when false, a tar.gz file will contain all the
  logs.
* `artcl_gen_docs`: true/false -- If true, the role will use build artifacts
  and Sphinx and produce user friendly documentation.
* `artcl_docs_source_dir` -- a local directory that serves as the Sphinx source
  directory.
* `artcl_docs_build_dir` -- A local directory that serves as the Sphinx build
  output directory.
* `artcl_create_docs_payload` -- Dictionary of lists that direct what and how
  to construct documentation.
    * `included_deployment_scripts` -- List of templated bash scripts to be
      converted to rST files.
    * `included_deployment_scripts` -- List of static rST files that will be
      included in the output documentation.
    * `table_of_contents` -- List that defines the order in which rST files
      will be laid out in the output documentation.

```yaml
artcl_create_docs_payload:
  included_deployment_scripts:
    - undercloud-install
    - undercloud-post-install
  included_static_docs:
    - env-setup-virt
  table_of_contents:
    - env-setup-virt
    - undercloud-install
    - undercloud-post-install
```

* `artcl_publish`: true/false -- If true, the role will attempt to rsync logs
  to the target specified by `artcl_rsync_url`. Uses `BUILD_URL`, `BUILD_TAG`
  vars from the environment (set during a Jenkins job run) and requires the
  next to variables to be set.
* `artcl_rsync_url` -- rsync target for uploading the logs. The localhost
  needs to have passwordless authentication to the target or the
  `PROVISIONER_KEY` Var specificed in the environment.
* `artcl_artifact_url` -- a HTTP URL at which the uploaded logs will be
  accessible after upload.

Example Playbook
----------------

```yaml
---
- name: Gather logs
  hosts: all:!localhost
  roles:
    - tripleo-collect-logs
```

Templated Bash to rST Conversion Notes
--------------------------------------

Templated bash scripts used during deployment are converted to rST files
during the `create-docs` portion of the role's call. Shell scripts are
fed into an awk script and output as restructured text. The awk script
has several simple rules:

1. Only lines between `### ---start_docs` and `### ---stop_docs` will be
  parsed.
2. Lines containing `# nodoc` will be excluded.
3. Lines containing `## ::` indicate subsequent lines should be formatted
  as code blocks
4. Other lines beginning with `## <anything else>` will have the prepended
   `## ` removed. This is how and where general rST formatting is added.
5. All other lines, including shell comments, will be indented by four spaces.

License
-------

Apache 2.0

Author Information
------------------

RDO-CI Team
