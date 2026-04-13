# GitHub Pages Build Fix Plan

## the Issue
GitHub Pages was failing with a Jekyll build error even though the logs didn't show a clear stack trace. 

The problems identified during local debugging and log analysis were:
1. **Invalid Liquid Filter Use:** The limit filter was being incorrectly piped in unexpected locations (inside expressions or logic that expected strings/arrays, or applied correctly but sometimes Jekyll strict mode complains about limit on specific array constructs vs slice). Actually, it was because limit is a or loop parameter in Liquid (e.g. {% for post in posts limit:5 %}), NOT a standard pipe filter (| limit: 5). Piping | limit: N causes a silent exception in some Jekyll implementations, stopping rendering at the file causing it.
2. **Duplicated Gem Configurations:** The Gemfile contained duplicate instances of gems like jekyll-feed, jekyll-sitemap, and jekyll-seo-tag.
3. **Missing Include Path:** The _pages/ directory wasn't explicitly included in the _config.yml, requiring an explicit include: configuration to guarantee rendering.

## Execution Steps (Completed)
1. Replace all incorrect uses of | limit: N with the valid | slice: 0, N array filter across templates (_layouts/home.html, _layouts/post.html, _includes/sidebar.html).
2. Remove the duplicated globally defined gems in the Gemfile that were also in the jekyll_plugins group.
3. Add include: ['_pages'] to _config.yml to explicitly instruct Jekyll to compile markdown files within that directory.
4. Commit the changes and push to origin main to trigger a successful GitHub Actions deployment.
