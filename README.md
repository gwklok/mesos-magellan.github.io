# [mesos-magellan.github.io](https://mesos-magellan.github.io)

These docs use [MkDocs](http://www.mkdocs.org/).

## Setup

```bash
pip install mkdocs
```

## Creating, and Deploying Docs

* Docs live in the `docs` directory. Write and structure using subdirectories (with the `index.md` being home for a directory).
* `mkdocs serve` to see how they look before deploying
* When it looks good, use `./deploy.sh` to push the rendered documentation up for view at [mesos-magellan.github.io](https://mesos-magellan.github.io).
* Commit your changes when ready.
```bash
git commit -am "Relevant short commit message"
git push origin content
```
