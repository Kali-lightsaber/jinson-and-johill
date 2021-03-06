Jinson & Johill
===============

Jinson is a tool that takes the `entities.xml` file contained in the zip files
generated by JIRA as backup archives, parses it, turns it to JSON and massages
it to make it more easy to work with (eg. by nesting objects that should
be nested instead of having a mess of not unique ids).

Johill takes the JSON output of Jinson and converts it to the JSON format used
by [Phill](https://git.collabora.com/cgit/user/em/phabricator.git/log/?h=phill)
to import data in Phabricator.

By using Jinson, Johill and Phill it is possible to migrate projects and tasks
from a JIRA instance to Maniphest (if you don't look too close).

Be warned: there are plenty of rough edges, a lot of stuff cannot be migrated
and some stuff that could be migrated is ignored because we didn't care enough
to bother. But it worked for us, so YMMV.


Usage
-----

```
# extract the backup generated by the JIRA web UI
unzip JIRA-backup-20150401.zip -d JIRA

# tell johill the base URL of the JIRA instance, so that it can link back to the original issues
# also process the JSON to only leave the TST project and its tasks, in case
# you have multiple projects in JIRA that you don't care about
./jinson JIRA/entities.xml \
  | ./johill - --base-url https://jira.example.org/browse \
  | jq '{ projects: .projects | map(select(.id == "TST")), tasks: .tasks | map(select(.id | startswith("TST"))) }' \
  > JIRA/tasks.json

# ta-dah! fill Maniphest with useless chatter and let's go shopping
phill JIRA/tasks.json -v 2
```
