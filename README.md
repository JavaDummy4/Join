(https://javadummy4.github.io/Join/)



<img width="1366" height="768" alt="Screenshot (5)" src="https://github.com/user-attachments/assets/2fbf4cae-406e-46e2-83cd-6ae0bb8ef5b7" />

<img width="976" height="2168" alt="sticker_1789578707490" src="https://github.com/user-attachments/assets/cf6e0ef9-2c39-4b7a-b3da-ab7e47caaf51" />

<img width="976" height="2168" alt="sticker_1789579234236" src="https://github.com/user-attachments/assets/53107921-343a-4ecb-9859-e7200447cc0c" />

<img width="976" height="2168" alt="sticker_1789579461075" src="https://github.com/user-attachments/assets/fe38e1a6-34ad-459d-bb42-3b7846fd20e2" />

Join
One HTML file. Any subject. Every relationship, visible.

Join is a self-contained object-relationship modeling tool — think of it as a lightweight, personal knowledge-graph editor that lives entirely in a single .html file. No install, no server, no account needed.

A city council and its 180 employees. A photo of a table and everything sitting on it. A 1930 pulp sci-fi story and how its characters relate. A car brand and the country it's built in. Join doesn't care what you model—it just keeps track of what's connected to what.

Why this exists
Most relationship-modeling tools are either too generic (a whiteboard app that doesn't know what a "class" is) or too narrow (a genealogy tool that only understands people). Join sits in between: it knows that objects have types, and relationships have meaning, but it never presumes what you're modeling.

And because the whole thing is one plain HTML file with a documented JSON format underneath, any AI can generate a new, ready-to-load model for you — just point it at the model format specification.

## Topics

`knowledge-graph` `data-modeling` `single-file` `no-code` `html` `javascript` `visualization` `graph-visualization` `offline-first` `personal-tools`

Quick start
Download Interaktives Beziehungsmodellierungs-Tool.html.
Open it in any modern browser (Chrome, Edge, Safari, Firefox).
Click + Object, give it a class, and start connecting things.
That's it. No dependencies, no npm install, nothing to configure.

Want it hosted so you can share a link instead of a file? See Hosting on GitHub Pages below.

What it can do
Model anything

80+ built-in classes — municipal administration, tourism, aviation alliances, the automotive industry, camera equipment, name etymology, literature, household items, freshwater/saltwater fish, whales, birds, and more.
Add a brand-new class in minutes; every object still gets a universal Remark field for anything that doesn't fit
Bilingual by design: every field, every relationship label, every UI string exists in German and English, switchable live
See it, however you like

Flip between a flat 2D diagram and a freely explorable 3D space
Select an object (by click or search) and the view automatically narrows to its first-degree relationships — a small "+" on any neighbor reveals what's hidden behind it, one step at a time
A collapsible class legend and an in-app, bilingual "?" quick-start guide
Build it faster

Voice input — dictate any field instead of typing it
📷 Photo → instant model — point your camera at a scene (say, a table with a few things on it) and Claude Vision identifies the room, the surface, and every object on it, wiring up the relationships automatically.
Combine and share without losing data

Merge two models and objects with the same name and class are automatically unified into one — filled-in fields are combined, and if two models disagree on a value, the conflict is recorded in Remarks.
Couple two models side by side to connect them manually with your own edges
Pick the second file from previously saved models or straight from disk
Share in one tap — as a file, by email, on WhatsApp, or through your device's native share sheet (with a graceful fallback on desktop browsers that don't support it)
Own your data

Everything lives in one JSON file per model: readable, diffable, git-friendly
No cloud, no lock-in, no telemetry — it's a file on your computer
The class system
Every object belongs to exactly one class, and every class belongs to a group (e.g. "Tourism & Gastronomy," "Aquatic Animals," "Household"). A class defines an icon, a color, and up to a handful of fields (a string, a number, a yes/no, a date, etc.).

This is what lets the same tool hold a municipal org chart, a fleet of aircraft, and a character map for a short story — without ever feeling like the wrong tool for any of them.

The photo feature needs its own API key
Photo analysis calls the Anthropic API directly from your browser (bring-your-own-key). A Claude.ai subscription does not cover this — the chat subscription and API usage are billed separately. To use this feature:

Create a key at platform.claude.com → API Keys → Create Key.
The first time you use Menu → 📷 Analyze Photo, you'll be asked to paste it in.
Your key is stored only in your browser's local storage and sent only directly to Anthropic — it never touches this repo or any server of ours.
Cost is small: with Claude Haiku 4.5 (the model this feature uses), a single photo typically costs a fraction of a cent. We'd still recommend setting a low monthly spending limit in the console, just to be safe.

Model files
A model is just JSON: a list of objects (nodes) and a list of relationships (edges) between them. The exact schema — including the full, current list of every class and its fields — is documented in Modelfile-Format-Spezifikation.

models/ in this repo collects a few finished examples to load and explore:

| Model | What's in it |
|-------|--------------|
| Fluglinien-Allianzen und Flotten | Star Alliance, oneworld, and SkyTeam, 61 airlines, fleet composition by aircraft type |
| Europäische und asiatische Autohersteller | Manufacturer groups, brands, models by powertrain, production countries and continents |
| Canon RF Objektive EOS R7 | Every current Canon RF/RF-S autofocus lens with specs, grouped by category |
| Spawn of the Comet – Personenbeziehungen | Character relationships from a 1930 public-domain sci-fi story |

Hosting on GitHub Pages
Since it's a single static HTML file, GitHub Pages works out of the box:

Push this repo, with the tool file renamed (or copied) to index.html.
Settings → Pages → Source: Deploy from a branch, pick main / root.
Your live link appears at https://<username>.github.io/<repo>/ within a minute or two.
This also gives you a secure (https://) origin, which some browser features (speech recognition, camera capture) prefer over a locally opened file.

Contributing / working together
Because everything is plain JSON and plain HTML, this repo works well with ordinary git workflows — meaningful diffs, pull requests, and no merge tooling beyond what GitHub already gives you. If you'd like to contribute a new model, add it to models/ with a meaningful filename, and open a pull request.

New feature since 15.09.2026: An integrated AI chat function connected to the displayed model. The AI can also place objects and define relationships using either chat or voice commands.
