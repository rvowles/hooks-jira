Plugin @PLUGIN@
===============

This plugin allows to associate Jira issues to Git commits.

Comment links
----------------

Git commits are associated to Jira issues reusing the existing Gerrit
[commitLink configuration][1] to extract the issue-id from the commit
messages.

[1]: ../../../Documentation/config-gerrit.html#__a_id_commentlink_a_section_commentlink

Additionally you need to specify the enforcement policy for git commits
with regards to issue-tracker associations; the following values are supported:

MANDATORY
:	 One or more issue-ids are required in the git commit message, otherwise
	 the git push will be rejected.

SUGGESTED
:	 Whenever a git commit message does not contain any issue-id,
	 a warning message is displayed as a suggestion on the client.

OPTIONAL
:	 Issue-ids are linked when found in a git commit message. No warning is
	 displayed otherwise.

Example:

    [commentLink "Jira"]
    match = (\\[[A-Z][A-Z]+-[1-9][0-9]*\\])
    html = "<a href=\"http://jira.example.com/browse/$1\">$1</a>"
    association = SUGGESTED

Once a Git commit with a comment link is detected, the Jira issue ID
is extracted and a new comment is added to the issue, pointing back to
the original Git commit.

Jira connectivity
-----------------

In order for Gerrit to connect to Jira/SOAP-API URL and credentials
are required in your `gerrit.config` / `secure.config` under the
`[jira]` section.

Example:

    [jira]
    url=http://jira.example.com
    username=admin
    password=jirapass

Jira credentials and connectivity details are asked and verified during the Gerrit init.

Comment configuration
---------------------

It is possible to choose which kind of Gerrit event will trigger the plugin to comment on
the Jira issue by adding the following in your `gerrit.config` file under the `[jira]` section:

commentOnChangeAbandoned
:	If true, abandoning a change adds a comment to the issue.

commentOnChangeCreated
:	If true, creating a change adds a comment to the issue.

commentOnChangeMerged
:	If true, merging a change adds a comment to the issue.

commentOnChangeRestored
:	If true, restoring an abandoned change adds a comment to the issue.

commentOnCommentAdded
:	If true, adding a comment or reviewing a change adds a comment to the issue.

commentOnFirstLinkedPatchSetCreated
:	If true, creating a patch set for a change adds a comment to the issue if
	the issue has not been mentioned in previous patch sets of the same change.

commentOnPatchSetCreated
:	If true, creating a patch set for a change adds a comment to the issue.

commentOnRefUpdatedGitWeb
:	If true, updating a ref adds a comment to the issue.

By default all parameters are set to true.

Example:

    [jira]
    commentOnCommentAdded=false
    commentOnRefUpdatedGitWeb=false

Gerrit init integration
-----------------------

The Jira plugin comes with a Gerrit init step that simplifies the
initial configuration. It guides through the configuration of the Jira
integration and checks the connectivity.

Gerrit init example:

    *** Jira connectivity
    ***

    Jira URL (empty to skip)       [http://jira.example.com]:
    Jira username                  [admin]:
    Change admin's password        [y/N]? y
    admin's password               : *****
                  confirm password : *****
    Test connectivity to http://jira.example.com [y/N]: y
    Checking Jira connectivity ... [OK]

    *** Jira issue-tracking association
    ***

    Jira issue-Id regex            [([A-Z]+-[0-9]+)]:
    Issue-id enforced in commit message [MANDATORY/?]: ?
           Supported options are:
           mandatory
           suggested
           optional
    Issue-id enforced in commit message [MANDATORY/?]: suggested

GitWeb integration
----------------

When Gerrit gitweb is configured, an additional direct link from Jira to GitWeb
will be created, pointing exactly to the Git commit ID containing the Jira issue ID.

Issues workflow automation
--------------------------

Jira plugin is able to automate status transition on the issues based on
code-review actions performed on Gerrit; actions are performed on Jira using
the username/password provided during Gerrit init.
Transition automation is driven by `$GERRIT_SITE/etc/issue-state-transition.config`
file.

Syntax of the status transition configuration file is the following:

    [action "<issue-status-action>"]
    change=<state-change-type>
    verified=<verified-value>
    code-review=<code-review-value>

`<issue-status-action>`
:	Action to be performed on the Jira issue when all the condition in the stanza are met.

`<state-change-type>`
:	Gerrit state change type on which the action will be triggered.
	Possible values are: `created`, `commented`, `merged`, `abandoned`,
	`restored`

`<verified-value>`
:	Verified label added on the Gerrit change with a value from -1 to +1

`<code-review-value>`
:	Code-Review label added on the Gerrit change with a value from -2 to +2

Note: multiple conditions in the action stanza are possible but at least one must be present.

Example:

    [action "Start Progress"]
    change=created

    [action "Resolve Issue"]
    verified=+1
    code-review=+2

    [action "Close Issue"]
    change=merged

    [action "Stop Progress"]
    change=abandoned

The above example defines four status transitions on Jira, based on the following conditions:

* Whenever a new Change is created on Gerrit, start progress on the Jira issue
* Whenever a change is verified and reviewed with +2, set the Jira issue to resolved
* Whenever a change is merged to the branch, close the Jira issue
* Whenever a change is abandoned, stop the progress on the Jira issue
