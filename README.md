# ultra-docs
Documentation for Flipkart Ultra Platform published using MkDocs.

The public link is https://ultra-docs.readthedocs.io/en/latest/

Making changes :
* Install MkDocs python plugin : http://www.mkdocs.org/
* Install themes : pip install mkdocs-material
* `mkdocs.yml` in the root folder of this repo contains the website structure. Make changes to `mkdocs.yml` if you want to add a new page.
* `index.md` inside the docs folder is the index page, similarly other pages have the .md extension. Adding a new page means adding a new .md file and editing the `mkdocs.yml` to contain this path.
* Once changes are made run `mkdocs serve` to test it locally. `mkdocs build` to convert the md files to html files.
* `mkdocs gh-deploy` to deploy it to github pages. Alternatively issue a pull request once you issue the `mkdocs build` command.
* Then push your changes made to MD and YML files to master. If `mkdocs gh-deploy` was run, then it would deploy the html files to a different branch on github, but the MD files have to be pushed anyway, so you need to push the master branch too.

