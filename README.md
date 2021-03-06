# ultra-docs
[![Documentation Status](https://readthedocs.org/projects/ultra-docs/badge/?version=latest)](https://ultra-docs.readthedocs.io/en/latest/?badge=latest)

Documentation for Flipkart Ultra Platform published using MkDocs.

To read the docs visit https://ultra-docs.readthedocs.io/en/latest/

# Making changes :
* Install MkDocs python plugin : http://www.mkdocs.org/
* Install themes : `pip install mkdocs-material`
* `mkdocs.yml` in the root folder of this repo contains the website structure. Make changes to `mkdocs.yml` if you want to add a new page. Changes to this file affects the side bar navigation on the left side of the page. Unless an entry to a file is not present in the mkdocs.yml, you will get a 404 error when trying to access it.
* `index.md` inside the docs folder is the index page, similarly other pages have the `.md` extension. Adding a new page means adding a new `.md` file and editing the `mkdocs.yml` to contain this path.
* All files use markdown to represent content. You can write markdown directly or use tools like https://stackedit.io/ to write your documentation.
* Once changes are made run `mkdocs serve` to preview it locally. 
* Commit your changes to your own branch and issue a PR to `master`
* Once PR gets merged, your changes will be live automatically.

# Deployment :
Deploying the documentation is automated and you dont have to do anything. All you have to do is merge the changes to master. Thats all !

Once your PR is merged to `master`, readthedocs.io will automatically pick the changes and compile them using mkdocs. Once compiled successfully, it will get auto deployed to the above readthedocs [public link](https://ultra-docs.readthedocs.io/en/latest/)

Please contact @thekirankumar for getting access to readthedocs admin account.
