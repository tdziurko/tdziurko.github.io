## [1.0.0] - 2022-04-04

- Initial Release


## [1.1.0] - 2022-04-23

- Improved background image loading in the subscribe section.
- Reduced large font on the author's page in articles.

### Changed
_includes/subscribe.html
_sass/3-modules/_subscribe.scss
_sass/4-layouts/_authors.scss


## [1.2.0] - 2023-01-30

- Replaced deprecated SASS properties.

### Changed
_sass/1-tools/_grid.scss


## [1.3.0] - 2023-04-25

- Fixed incorrect display of the authors page when using Github Pages.

### Changed
_pages/authors.html


## [1.4.0] - 2025-03-14

- Added support for callouts
- Added a copy code button
- Replaced Ionicons with Font Awesome icons
- Fixed a warning about deprecated SASS properties.

### Changed
**_sass** to **css**
_data/settings.yml
_includes/head.html
_includes/header.html
_includes/section-blog.html
_includes/section-tag.html
_includes/section-videos.html
_includes/search.html
_includes/footer.html
_layouts/author.html
_layouts/post.html
_pages/tags.html
css/_0-settings/color-scheme.scss
css/_1-tools/syntax-highlighting.scss
css/_2-base/base.scss
css/_3-modules/search.scss
css/_3-modules/header.scss
css/_4-layouts/authors.scss
css/_4-layouts/post.scss
js/common.js
_config.yml