---
name: front-end-overhaul
description: Overhaul the front end of a Professional Training keyword microsite built from the MasterMicroSite template (usually Eleventy 0.12, upgraded to Eleventy 3 as part of this). Use when asked to redesign, modernise or re-skin one of these microsites, or when asked to run this SKILL.md.
---

# Redesign the front end of a training microsite

These projects are microsites built from the MasterMicroSite template, usually still
on Eleventy 0.12. Each one sells instructor-led training for one keyword set
(`searchKeywords` in `src/_data/site.json`, e.g. "AI", "Adobe") and pulls course
data from the professional.ie API.

The goal is the end state described below. It was refined over several rounds of
feedback, so build it directly rather than proposing alternatives. Everything that
mentions a topic should use the site's own keyword, never a hard-coded one.

Before designing, load the frontend-design guidance (the `frontend-design` skill).

## Design principles

- Modern, simple and clear, with careful use of colour.
- Meets modern usability and accessibility standards (WCAG 2.1 AA).
- Optimised for Google search rankings.
- Some motion so the page doesn't feel static: an entrance for the banner text, and
  a few elements that ease in as they scroll into view. Only the elements listed in
  section 4 move. Never animate every section.

## 1. Get the project building first

Do this before any design work, and confirm the build is clean.

1. **Node.js:** if `npm` isn't found, Node isn't installed. Ask the user to run
   `! winget install OpenJS.NodeJS.LTS --accept-source-agreements --accept-package-agreements`.
   The current shell won't see it on PATH until restarted, so call it by full path
   (`C:\Program Files\nodejs`) or prepend that folder to PATH in each command.
2. Run `npm install`, then do a one-off build: `npx @11ty/eleventy`.
3. Fix these known template bugs if the build shows them:
   - **`DuplicatePermalinkOutputError` on `training_courses/c/`:** "C" and "C#" slugify
     the same way. Before `slug`, replace `#` with ` sharp` and `+` with ` plus`, in
     both the `keywordpagination.md` permalink and the course page "related topics"
     links. Do the same at the start of `slugify()` in `src/_includes/makeSection.js`,
     which also keeps `#` out of saved-search URLs.
   - **Duplicate saved-search pages:** the landing-page API can return the same
     keyword twice. In `src/_data/coursesapifull.js`, de-duplicate the landing-page
     JSON by keyword (case-insensitive) before building `msKeywordData`.
   - **"contains invalid WIN32 path characters":** some keywords contain `*`.
     Extend the `removeInvalidChars` filter in `.eleventy.js` to also strip
     `* ? " < > |`.
4. **Save a baseline** once it builds: copy `build/` to a scratch folder and save its
   sorted file list. Every later step is checked against it, and the upgrade below
   must not change any URL.
5. Keep this skill at `.claude/skills/front-end-overhaul/` in the project root, not
   inside `src/`. Anything under `src/` is built into the site, and on Windows
   Eleventy 0.12 can't exclude it with `.eleventyignore` or `.gitignore`.
6. **Dev server: check before starting one.** The user usually starts
   `npm run start:dev` themselves at the start, to look at the site before any
   changes. Before you start a server (or ask for one), check whether one is
   already running:
   - List the listening ports from 8080 to 8090 (`netstat -ano | findstr LISTENING`).
     A second server doesn't fail; it quietly takes the next free port (8081, 8082
     and so on), so check the whole range.
   - For each one, fetch `http://localhost:<port>/` and check that its `<title>`
     contains this site's `site.searchKeywords[0]`. Another microsite's server may be
     running too.
   - **If this site's server is running, use it and its port. Don't start a second
     one.** Two watchers write to the same `build/` folder.
   - Only start one (`npm run start:dev`, in the background) if none is serving this
     site. Note which port it reports.
   - **A server started before the upgrade goes stale.** It is the project's original
     Eleventy 0.12. While it watches `src/`, it keeps rebuilding `build/` with the old
     version, which spoils the step 2.7 comparison. Before starting section 2, ask the
     user to stop it (Ctrl+C in its terminal); don't kill their process yourself.
     After step 2.7, check again and start a fresh one (or ask them to).
   - Restart the server the same way after deleting any templates, such as the
     `src/svg/` pages. A running server still has them in memory and writes their
     pages back into `build/`. A running server also locks `build/`, so
     `rm -rf build` fails while one is running.
   - **Stopping a server you started:** stopping the background task only ends the
     `npm`/`cmd` wrapper. Eleventy's `node` child keeps serving and watching. Find it
     by port (`Get-NetTCPConnection -State Listen`) and check its command line
     (`Get-CimInstance Win32_Process`) is this site's `eleventy ... --serve`. Then
     stop that process and its `cmd` parent, and confirm the port is free. Never stop
     another site's server or one the user started.

## 2. Upgrade to Eleventy 3

Do this after the baseline and before the redesign, so everything else is built on
the new version. Check that the project isn't already on v3 (`package.json`).

1. `npm uninstall @11ty/eleventy-cache-assets`, then
   `npm install --save-dev @11ty/eleventy@3`, `npm install @11ty/eleventy-fetch@5` and
   `npm install --save-exact slugify@1.5.3`.
2. In `src/_data/coursesapifull.js`, require `@11ty/eleventy-fetch` in place of
   `eleventy-cache-assets` (the API is the same), and delete the
   `require("node-fetch")` line. It's unused and isn't a declared dependency.
3. **Keep URLs identical.** v3's `slug` filter collapses repeated dashes, so a course
   name with an en dash ("Situations – Call") gets a different URL. Override it in
   `.eleventy.js` with the slugify version 0.12 used:
   `config.addFilter("slug", str => slugifyPackage(str, { replacement: "-", lower: true }))`,
   where `slugifyPackage = require('slugify')`. Leave a comment saying why it's
   pinned.
4. **`page` is reserved in v3.** Replace every front-matter `page:` block with a
   top-level `pageType:` key (keeping its `type` value), and change `page.type` to
   `pageType` in `htmlhead.njk`. Drop any old `page.title` values. They're stale
   copies from other sites (e.g. "Search For Angular Training").
5. **Liquid rejects unknown filters in v3.** Remove `| safe` (a Nunjucks filter) from
   `.md` pages; Liquid doesn't escape output anyway.
6. In the config's returned object, drop `templateFormats` (the
   `setTemplateFormats` call already sets them) and write the dirs without a leading
   slash: `input: 'src'`, `output: 'build'`, `layouts: '_includes/layout'`.
7. Build and compare with the baseline: the file list must be identical, and only
   expected pages may differ in content. The dev server (`npm run start:dev`,
   http://localhost:8080) is now Eleventy's own server. Check for a running server
   (step 1.6), then start one on v3 if none is serving this site.

## 3. Tidy the template leftovers

- **Stop building stub and debug pages** by adding `permalink: false` to their front
  matter: `testpagination.md` (`/test/...`), `ss.md` (a duplicate of
  `saved_searches.md`), an empty `about.md`, and `saved_searches.md` itself (its
  index page dumps the raw API data; the `/saved_searches/<city>/...` landing pages
  come from `landingpagespagination.md` and are unaffected).
- **Delete unused images.** Grep for each file name before deleting; words like
  "coding" or "background" give false matches. Old product logos (e.g. `adobe*.png`)
  and oversized testimonial photos are the usual leftovers.
- **`site.json` `logo`** is used as the image in the course structured data. Point it
  at an absolute URL that suits every keyword:
  `https://www.professional.ie/img/ProfessionalTrainingOffice.jpg`. Template copies
  often still point at a product logo.
- **Branding:** delete `original_branding.json` and the unused `neutral` and
  `gradient_*` keys, along with their lines in `branding.njk`.
- Check that `site.url` is the microsite's real domain. Template copies can still
  point at another site (e.g. `adobetraining.ie`), and every canonical link will be
  wrong.

## 4. Replace the front-end stack

- Remove Bootstrap 4, jQuery, Popper, Font Awesome, animate.css and AOS. Write one
  hand-made stylesheet (`src/css/style.css`) using CSS custom properties, plus a
  small `src/js/site.js` that progressively enhances the page. Every page must
  still work without JavaScript.
- Remove every `document.write`:
  - The copyright year becomes a `year` shortcode.
  - Booking buttons are always rendered. `site.js` swaps any whose date has passed
    for a "Completed" label.
- Layout shell (`base_layout.njk`): `lang="en-IE"`, a skip link, then `<header>`
  (nav), then `<main id="main">`, then `<footer>`. `htmlhead.njk` adds the `js` class
  to `<html>` in an inline script before anything is drawn; the scroll-in animations
  rely on it.

## 5. Visual system

**Colour.** Tokens live in `src/_data/branding.json` and are output as CSS variables
by `branding.njk`. Keep the same keys so each microsite can re-theme, and give every
new key a fallback in `branding.njk` so sites without it don't break. Never hard-code
a colour in `style.css`; always use the variables.

**Every microsite gets its own palette.** Visitors move between these sites, so no
two may look alike. Don't reuse another site's palette or the example below as is.

1. **See what's taken.** Read `primary_color` and `highlight_color` from every
   `C:\work\MicroSite\*MicroSite\src\_data\branding.json` (skip this site and
   `MasterMicroSite`). Sites change, so always check the files rather than relying
   on the list below.
2. **Pick a new hue family** for `primary_color`:
   - Its hue must be at least about 30° from every other site's primary on the colour
     wheel. Lightness alone doesn't count: a darker green is still green.
   - It must be dark enough for white button text, at least 4.5:1 (aim for 7:1).
   - Where you can, let the subject suggest it. For example, a product with a known
     brand colour can use a hue near it, as long as it stays clear of the other sites.
   - Then pick a `highlight_color` that also differs from the other sites'
     highlights. It needs at least 3:1 against `primary_color` and
     `accent_deep_color`, because it is the focus outline on both.
3. **Derive the rest from that hue:**
   - `primary_strong_color`: a darker shade of the primary.
   - `secondary_color` and `accent_soft_color`: very light tints of the primary.
   - `foreground_color` and `secondary_foreground_color`: near-neutral with a touch of
     the hue.
   - `accent_deep_color`: a very dark shade of the primary.
   - `warm_soft_color`: a pale tint of the highlight.
4. **Check contrast** with a script before using the palette. Text needs at least
   4.5:1; the focus outline needs at least 3:1. Check these pairs:
   - white on primary, white on primary_strong, white on accent_deep;
   - primary on white, on secondary and on accent_soft (links, topic tags);
   - primary_strong on accent_soft;
   - foreground and secondary_foreground on white, secondary and accent_soft;
   - foreground on highlight (text selection);
   - highlight against primary and against accent_deep (focus outline).
   Never convey meaning with colour alone.
5. **Recolour the illustrations** to the new primary (see Illustrations below).
   If a site is re-themed later, recolour its SVGs too, and add or bump a `?v=N` on
   their `src` in `home.json` so browsers don't keep the old colours.
6. In the report, give the palette and say which sites' colours it was chosen to
   avoid.

Example palette (the Technical Writing site), to show what each key is for:

| Key | Example | Use |
|---|---|---|
| `primary_color` | `#7B2D6B` (plum) | links, buttons, accents, testimonials band |
| `primary_strong_color` | `#5F2253` | hover states, text on the soft accent |
| `secondary_color` | `#F6F1F5` | alternating section bands, course dates band |
| `foreground_color` | `#2A1E27` | body text |
| `secondary_foreground_color` | `#5B4D57` | secondary text |
| `highlight_color` | `#F7B267` (apricot) | focus outline on dark or primary backgrounds, text selection |
| `accent_soft_color` | `#F1E1EC` | banner, closing call to action, inner page headers, topic tags |
| `accent_deep_color` | `#3A1233` | footer |
| `warm_soft_color` | `#FDE9D4` | soft disc behind the banner illustration |

Palettes in use as of November 2026 (primary / highlight). Check the files anyway:

| Site | Primary | Highlight |
|---|---|---|
| Adobe | `#A3123A` crimson | `#3FD1C0` aqua |
| AI | `#4338CA` indigo | `#FF8F6B` coral |
| Java | `#8A3B12` rust | `#F28C28` orange |
| JavaScript | `#1B5FA8` blue | `#F7DF1E` yellow |
| MS Excel | `#0A7756` green | `#F5C84B` yellow |
| Python | `#2E4A62` slate | `#FFD43B` yellow |
| Technical Writing | `#7B2D6B` plum | `#F7B267` apricot |

**Type.** Bricolage Grotesque (headings, 500/700) and Atkinson Hyperlegible (body),
from Google Fonts with `display=swap`. Body text is about 17px with 1.6 line height,
and readable text blocks are capped at about 68 characters per line.

**Avoid these generic patterns:** all-caps labels above headings, gradient washes, an
identical card-and-shadow on every element, fade-in on every section, and arrows
appended to link text.

## 6. Home page (`src/_data/home.json` + `makeSection.js`)

The section order is: banner, About, Testimonials, the closing call to action, then
the low-key topic and location index. The menu shows Home, About, Testimonials,
Schedule and Courses. All the copy lives in `home.json` so the user can edit it.
If the site has no testimonials, the Testimonials section and its menu link are
left out (see Testimonials below).

- **Banner (`banner`):**
  - The `<h1>` is the site title (e.g. "AI Training Courses Ireland"), followed by
    the lead (`subtitle`). The lead **must always begin with this exact sentence**,
    word for word, on every site: "Practical, instructor-led training in-person at
    our Dublin, Cork and Limerick offices, at your premises nationwide or live
    online." Follow it with one short, keyword-specific sentence (e.g. "Learn to
    write, test and maintain reliable Java applications, from the core language
    through to Spring and Hibernate."). Never reword, shorten or reorder the
    opening sentence.
  - Two buttons: "Browse courses" (`/courses/`) and "See course dates"
    (`/schedule/`).
  - On the right, an unDraw illustration on the soft accent background
    (`accent_soft_color`), with a soft disc (`warm_soft_color`) behind it.
  - **Motion, pure CSS, on load:** `makeSection.js` wraps each headline word in
    `<span class="hero__word" style="--w:N">` and sets `--words` on `.hero__copy`.
    The words rise in about 90ms apart, then the lead, then the buttons, all within
    about 1.5 seconds. A title containing HTML is left whole. The illustration fades
    in and rises 8px.
  - Keep this look. These alternatives were mocked up and rejected: a typed AI
    prompt and reply, upcoming dates in the banner, a "pick your role" course
    finder, and big kinetic type with a ticker.
- **About (`text` type with a `content` array):** one simple section with a heading
  ("About our {keyword} courses") and two short paragraphs in a single column, with
  an illustration beside it. Do not split it into several About sections or
  columns. If the template's home.json has one section per product, merge their
  text into these two paragraphs.
- **Testimonials (`quotes` type, `id: "testimonials"`):**
  - Heading "What our attendees say". Each quote is a white card on a solid primary-colour
    band, with a large primary-colour opening quote mark and the name and job title below a
    divider.
  - Cards sit side by side, two or three to a row, and stack on phones. Cards in a
    row are equal height.
  - Use only real testimonials: ones already in the site's `home.json`, or ones the
    user gives you. Never invent realistic testimonials, and don't add a placeholder
    quote.
  - **No testimonials, no section.** If the site has none, keep the section in
    `home.json` with an empty list (`"quotes": []`) so the user can add quotes
    later. An empty list removes the whole section from the page and its
    "Testimonials" link from the menu:
    - `makeSection.js` has a `hasQuotes(section)` helper. The `quotes` case returns
      `""` when it is false.
    - `.eleventy.js` adds an `inMenu` filter (`link != "no"`, and for `quotes`
      sections `hasQuotes`). `nav.njk` uses `{% if item | inMenu %}`.
    - Adding the first quote to the list brings back both the section and the
      menu link, with no other change.
    - The microsite-qa tool reads `home.json` and only expects `#testimonials` and
      the menu link when there is at least one quote.
- **Closing call to action (`cta` type, `id: "get-started"`, `link: "no"`):**
  - Fields: `title`, `content` (a string or a list of paragraphs), `actions` (buttons,
    defaulting to Browse courses and See course dates), and an optional `note`, which
    may contain a link.
  - A soft accent band (`accent_soft_color`), so it bookends the page with the banner. The heading and text are on
    the left and the buttons are bottom-right. The note runs full width below a thin
    divider. It stacks to one column on phones, with full-width buttons.
  - Default copy: "Find the right {keyword} course for you"; a line on the range of
    courses ("From a one-day introduction … to in-depth courses for …. See what each
    course covers, then book a date in Dublin, Cork or live online."); and the note
    "Training a team? We can run any course at your premises or online, on dates
    that suit you. <a href="#contact">Contact us</a>". `#contact` is the footer's
    contact column.
- **Topic and location index (`asAccordion` in `makeSection.js`):** this exists for
  search engines, so keep it low-key, just above the footer:
  - No visible heading, intro text or illustration. Use a visually-hidden
    `<h2>Training by topic and location</h2>` for screen readers.
  - A `<details>` whose summary is small, muted text with a small chevron: "All
    topics and locations (N)". It is not a button, but keep a 44px tap height.
  - Inside, a compact grid in small text: each topic name (a small `<h3>` in the body
    font) with its location links. The visible link text is the location; add
    visually-hidden text with the full name.
- **Scroll-in motion** (`site.js` plus the "Scroll-in animations" block in
  `style.css`): this applies to the About illustration (`.topic .split__image`), the
  testimonial cards (`.quote`, when there are any) and the call-to-action content
  (`.cta__inner`).
  - `.js .element` starts hidden from the first frame. This works only inside
    `@media (prefers-reduced-motion: no-preference)`, so with no JavaScript or with
    reduced motion nothing is ever hidden.
  - An IntersectionObserver adds `.is-visible`, so elements already on screen at
    load animate straight away. When the transform transition ends (or after a 2s
    fallback), `site.js` adds `.is-settled`.
  - The illustration slides in from the right. Cards rise in, staggered by
    `--i`, and the quote mark pops in shortly after.
  - Once `.is-settled`, the illustration drifts slowly (8px over 7s, using the
    `translate` property) and cards lift 4px on hover.
  - The print stylesheet forces all of these visible.
- Remove the old product logo images (e.g. Adobe logos) from home sections, and
  don't leave product names in section ids or anchors.

**Illustrations (unDraw):**
- Find suitable ones via `https://undraw.co/search/<term>`. The files are at
  `https://cdn.undraw.co/illustration/<name>.svg`.
- Two are needed: one for the banner (e.g. a person with a document and AI) and one
  for About (e.g. an instructor at a board). Pick ones that match the subject and
  avoid ones with large dark shapes.
- Download into a separate scratch folder, check them for `<script>`, event
  handlers and external `href`s, then recolour by replacing `#6c63ff` with this
  site's `primary_color` (not the example palette's).
- Save to `src/img/undraw-<name>.svg`. The passthrough copy only covers the
  `src/img/` top level, not subfolders.
- In home.json, give each one `src`, `width` and `height` (from the SVG viewBox) so
  the page doesn't jump while loading. Use `alt=""` because they're decorative.
  Lazy-load everything below the banner.
- On phones, illustrations shrink to about 16–18rem and sit above their text.

## 7. Inner pages

- **Page header band (`accent_soft_color`):** small "Courses" breadcrumb, `<h1>`, and lead text.
- **Course page:**
  - A facts row: duration, format, and next date. The next date must be the first
    *future* date (`nextDate` filter), not just the first one in the list.
  - **Next date jumps to the booking dates (must keep).** The next date in the
    facts row is a link, `<a href="#dates">Wed 2 Dec 2026</a>`, to the "Course
    dates" section (`<section class="band" id="dates">`) further down the page, so a
    visitor who sees a date they want can click it and land on the table with the
    Book buttons. To make this work:
    - Only render the "Next date" fact when `nextDate` returns a date; leave it
      out entirely when there are no future dates.
    - Use a plain anchor, not JavaScript, so it works without scripts.
    - In `style.css`, set `scroll-behavior: smooth` and `scroll-padding-top`
      (about `5.5rem`, the sticky header's height) on `html`, so the page glides
      down and the "Course dates" heading isn't hidden under the header. Switch
      back to `scroll-behavior: auto` under `prefers-reduced-motion: reduce`.
    - Check it on a built course page: click the date and confirm the dates table
      and its Book links are in view.
  - "What the course covers": each outline block in a responsive grid. The API's
    newline-separated text becomes a `<ul>` (`courseText` filter).
  - "Course dates" table (the `#dates` target): Date, Location and Book only, not
    the course name. Dates are formatted like "Mon 12 Oct 2026" (`displayDate`
    filter). Each Book link goes to
    `https://www.professional.ie/course_schedule/<course file>.html?id=<instance id>`
    and has visually-hidden text naming the course, date and location.
  - "Related topics" as pill links on the soft accent colour.
  - If there are no dates, show a message pointing to "Contact us".
- **Courses list and keyword pages:** a list of courses showing name, a short
  description and "N days, next date …". Keyword pages show a message when no
  courses match.
  - The Courses page header lead (in `src/courses.md`) is exactly this, word for
    word, on every site, with "Contact us" linking to the footer's contact column:
    "Instructor-led courses in Dublin, Cork, Limerick and live online. Choose a
    course to see what it covers and when it runs. Contact us with any queries."
    (`... when it runs. <a href="#contact">Contact us</a> with any queries.`)
- **Landing pages** (`landingpage_layout.njk`, `/saved_searches/<city>/...`):
  - The header band has the "Courses" breadcrumb, the `<h1>` (the saved-search
    title) and a lead: "Instructor-led {topic} courses in {location} and live
    online…". Use "across Ireland" when the location is Ireland.
  - The API's intro paragraphs come next, then `<h2>{topic} courses in {location}</h2>`,
    where topic is the keyword without " Training".
  - The course list matches the Courses page, including duration and next date. In
    `coursesapifull.js`, build each saved search's `courses` from
    `distinctCourseListWithSchedule` so the schedule is available.
- **Schedule page:** one section per course, each with a "Course details" link and
  a dates table. The header lead (in `schedule_layout.njk`) is exactly this, word
  for word, on every site:
  "Upcoming public dates live online. Choose a date and book your place, or contact
  us to run a course for your team. In-person training available for your teams at
  our Dublin, Cork and Limerick offices or at your premises nationwide."
- On phones, tables become stacked rows with the Book button on the right.
- **Footer addresses** use a `formatAddress` filter, which joins `line1`, `line2`,
  `town`, `county` and `postcode` with commas, including between county and
  Eircode (e.g. "Kinsale Road, Cork, T12 TP8H"), and skips blank parts. Printing
  each line with a comma leaves ", , ," when a line is empty.
  The county and Eircode are joined with non-breaking spaces (`\u00A0`) so the
  postcode always stays on the same line as the county: the Dublin address wraps
  as "26 Upper Pembroke Street," / "Dublin 2, D02 X361", never with "D02 X361"
  (or half of it) on a line of its own.

## 8. Structured data (JSON-LD)

The template's JSON-LD is partly broken. Fix it as follows:
- **`instances.njk`:** the template adds commas between dates within a course but not
  between courses, so the schedule page's event list is invalid JSON. Collect the
  events into an array (`{% if events.push(...) %}{% endif %}`) and output
  `[{{ events | join(",") | safe }}]`.
- **`carousel.njk`:**
  - Output the name and description with `| dump | safe` (JSON encoding), because
    descriptions contain `"`.
  - Use course URLs of the form
    `https://{{ site.url }}/courses/{{ course.name | slug | removeInvalidChars }}/`.
  - Remove the nested `@context` lines.
- **Addresses:** keep one `offices` map (Cork, Dublin and Limerick, from `site.json`)
  and a `postalAddress(city)` helper in `.eleventy.js`. Use them in
  `getLocationInfo`, so Limerick no longer falls back to Cork, and in an
  `officeAddresses` shortcode for `educationalOrganisation.njk`.
  - In `educationalOrganisation.njk`, take the phones and email from `site.json`,
    set `sameAs` to the LinkedIn URL (`professionals.ie` only redirects to
    professional.ie), and set `image` to `site.logo`.
- Use `https://schema.org` everywhere, and an `https://` organizer URL.

## 9. Accessibility and SEO checklist

- One `<h1>` per page, with headings in order. Visible `:focus-visible` outlines
  (`highlight_color` on dark or primary-colour backgrounds). Touch targets at least 44px. Mobile menu
  uses a button with `aria-expanded` and closes with Escape. Every animation is
  switched off with reduced motion, and nothing is ever hidden when JavaScript is
  off.
- **Tab icon (favicon) and logo mark:** both show the first letter of
  `site.searchKeywords[0]` in white on this site's `primary_color`, so they work
  for any keyword and never need editing by hand.
  - `htmlhead.njk` builds the icon as an inline SVG in a `{% set iconSvg %}` block
    and outputs `<link rel="icon" type="image/svg+xml" href="data:image/svg+xml,{{ iconSvg | trim | urlencode }}">`.
  - The letter is `{{ site.searchKeywords[0] | first | upper }}`, set in a system
    font (Arial, bold), because a tab icon can't load web fonts.
  - The logo mark in `nav.njk` uses the same letter.
  - Never copy a hard-coded icon from another site; the AI site's has an indigo "A".
- Each page has its own `<title>`, meta description, canonical
  (`https://{site.url}{page.url}`) and Open Graph tags.
  - Course page descriptions come from the course description, cut to 155
    characters.
  - Use a `metaTitle` front-matter field when the menu label (`title`) is too short
    for search results, e.g. on the Courses page.
  - `htmlhead.njk` chooses titles by the front-matter `pageType`.

## 10. Verify before reporting

1. Run a clean build with no errors or warnings, then confirm there is no
   `document.write`, Bootstrap or jQuery in the output.
   - Re-read every other site's `branding.json` and confirm this site's primary
     and highlight colours are still distinct (section 5). Confirm the illustrations
     use this site's primary colour, with no `#6c63ff` or another site's colour left.
2. Compare the build's file list with the baseline from step 1.4. The only changes
   should be the pages and images deliberately removed.
3. Parse every `<script type="application/ld+json">` block in `build/` with a JSON
   parser. All of them must be valid.
   - Confirm the home page's banner lead in `build/index.html` begins with
     "Practical, instructor-led training in-person at our Dublin, Cork and Limerick
     offices, at your premises nationwide or live online." (see section 6).
4. **Screenshots:** with the dev server running (step 1.6: reuse the user's
   server if it's running on v3, and use whatever port it's on), use headless Edge or Chrome:
   `msedge --headless=new --hide-scrollbars --window-size=1400,3400 --virtual-time-budget=5000 --screenshot=<file> http://localhost:8080/`.
   - Check the home page (including the call to action and footer), a course page,
     a landing page and the schedule at desktop width.
   - Capture a course page with `#dates` on the end of the URL and confirm it opens
     at the "Course dates" heading, not hidden under the header.
   - A very short `--virtual-time-budget` (about 350ms) shows the banner animation
     mid-way.
   - Headless browsers won't lay out narrower than about 496px, so a 400px capture
     falsely looks cut off on the right. Check phone layout at 520px, or at a true
     390px inside an iframe.
5. Fix anything that looks off, such as elements overlapping or cropping an
   illustration, cramped spacing or wrong dates, then capture again.
6. **Run the QA tests.** Use the `microsite-qa` skill on this site. It rebuilds and
   runs `C:\work\MicroSite\microsite-qa\run-qa.mjs`. Fix every critical and serious
   issue in `src/`, then rebuild and test again until none are left that the site can
   fix. Problems in the API's course data are listed for the user, not fixed here.
7. Leave the dev server running (only one for this site, see step 1.6), and give its
   URL with the actual port. Report what changed, what was checked, the QA
   summary (counts by severity and the path to `report.html`), and what the user
   needs to replace: the copy, the privacy page and `site.url`. Say whether the
   Testimonials section is shown or hidden (no quotes yet), and how to add quotes to
   bring it back. Don't commit unless asked.

## Reference implementation

`C:\work\MicroSite\AiMicroSite` is a finished example of all of the above, for the
"AI" keyword. When it is available, compare against or copy from these files:
- `.eleventy.js` and `package.json`
- `src/_includes/` (`htmlhead.njk`, `nav.njk`, `footer.njk`, `branding.njk`,
  `makeSection.js`, `layout/*.njk`, `StructuredData/*.njk`)
- `src/css/style.css` and `src/js/site.js`
- `src/_data/branding.json`, `src/_data/home.json` and `src/_data/coursesapifull.js`
- `src/courses.md`, `src/keywordpagination.md` and `src/landingpagespagination.md`
