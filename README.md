Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the 
[Attorney-General of New Zealand](https://www.wikidata.org/wiki/Q4818600) 
was pretty good already: all I had to add was it was part of the Cabinet.

Step 2: Tracking page
=====================

Initial PositionHolderHistory list set up at https://www.wikidata.org/w/index.php?title=Talk:Q4818600&oldid=1232964762

Current status: knows of only one officeholder (and with only
year-precision dates)

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js) 
to configure the Item ID and source URL.

Step 4: Scrape
==============

Comparison/source = [Minister for Māori Development](https://en.wikipedia.org/wiki/Minister_for_M%C4%81ori_Development)

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

Scraped cleanly on first pass.

Step 5: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

47 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/792dbb9a1a8ad

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

No additions to make, but two suggested corrections:

1. David Lange start: 1989 → 1989-08-04
1. David Lange end: 1990 → 1990-11-02

Those both look fine, so I'll accept them using

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json 2>&1 >/dev/null |
      egrep "\t" | wd uq --batch --summary \
        "Update qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

-> https://tools.wmflabs.org/editgroups/b/wikibase-cli/b526eb7317bf6

Step 8: Refresh the Tracking Page
=================================
Final version at https://www.wikidata.org/w/index.php?title=Talk:Q4818600&oldid=1232970250

