# Privacy

Varis Plan Tracker is published by True North Consulting. It is a skill and
an HTML template; it runs no hooks and no scripts of its own.

**What we receive: nothing.** The plugin has no server. We collect no data,
no telemetry and no usage statistics.

**Where your plan goes.** The skill writes a copy of the template to a folder
on your machine (by default `~/.claude/plan-runs/<plan>/`) and publishes it as
a private artifact on your own claude.ai account. The plan's packages, states,
PR numbers and open questions are stored in that artifact's database, under
your account's terms. Only you and the people you share the artifact with can
open it. Delete the artifact to delete the data.

**One third-party request.** When the tracker page is opened, the browser
loads two web fonts from Google Fonts. Google receives the viewer's IP address
and browser details; it receives no plan content.

Do not put secrets, credentials or personal data in a plan's notes: they end
up on the page.

**Contact.** Questions or concerns: open an issue at
https://github.com/True-North-Consulting/varis-plan-tracker/issues.
