Refactored CSS structure
========================

HTML files:
- index.html
- publications.html
- projects.html

CSS files:
- assets/css/common.css       Shared site-wide styles: colors, typography, buttons, nav/header, page-title cards, footer, fade-in.
- assets/css/index.css        Index-only styles: hero, contact icons, particles area, journey timeline.
- assets/css/publications.css Publications-only styles: publication cards and actions.
- assets/css/projects.css     Projects-only styles: project cards, filtering chips, dark-mode chip colors.

How to make global design changes:
Edit assets/css/common.css. For example, changing --brand or .site-header there updates all pages.

How to make page-only changes:
Edit the matching page CSS file, for example assets/css/index.css for only the homepage.

Important:
Keep the assets folder structure the same:
- assets/css/
- assets/images/
- assets/logos/
