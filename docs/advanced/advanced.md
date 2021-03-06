# How-to and FAQ

This page contains more advanced and complete information about the
[`jupyter-book` repository](https://github.com/executablebooks/jupyter-book). See the sections below.

## Enable Google Analytics

If you have a Google Account, you can use Google Analytics to collect some
information on the traffic to your Jupyter Book. With this tool, you can find
out how many people are using your book, where they come from and how they
access it, whether they are using the Desktop or the mobile version etc.

To add Google Analytics to your Jupyter Book, navigate to
[Google Analytics](https://analytics.google.com/analytics/web/), create a new
Google Analytics account and add the url of your Jupyter Book to a new
*property*. Once you have set everything up, your Google Analytics property
will have a so-called Tracking-ID, that typically starts with the letters UA.
All that you need to do is to copy this ID and paste it into your
configuration file:

```yaml
html:
  google_analytics_id: UA-XXXXXXXXX-X
```

## Check external links in your book

If you'd like to make sure that the links outside of your book are valid,
run the Sphinx link checker with Jupyter Book. This will check each of your
external links and ensure that it resolves.

```{margin}
Note that you must ensure each link is
the *right* target, the link checker will only ensure that it resolves.
```

To run the link checker, use the following command:

```bash
jupyter-book build mybookname/ --builder linkcheck
```

It will print the status of each link in your book so that you may resolve
any incorrect links later on.

(clean-build)=
## Clean your book's generated files

It is possible to "clean up" the files that you generate when you build
your book. This is often useful if you have recently changed a lot of content
in order to ensure that you build your book from a clean slate.

You can clean up your book's generated content by running the following command:

```bash
jupyter-book clean mybookname/
```

By default, this will delete all folders inside `mybookname/_build` *except*
for a folder called `.jupyter_cache`. This ensures that the *content* of your book
will be regenerated, while the cache that is generated by *running your book's code*
will not be deleted (because regenerating it may take some time).

To delete the `.jupyter_cache` folder as well, add the `--all` flag like so:

```bash
jupyter-book clean mybookname/ --all
```

This will entirely remove the folders in the `_build/` directory.

(jupyter-cell-tags)=

## How should I add cell tags and metadata to my notebooks?

You can control the behaviour of Jupyter Book by putting custom tags
in the metadata of your cells. This allows you to do things like
{doc}`automatically hide code cells <../interactive/hiding>`) as well as
{ref}`adding interactive widgets to cells <launch/thebelab>`.

There are two straightforward ways to add metadata to cells:

1. **Use the Jupyter Notebook cell tag editor**. The Jupyter Notebook ships with a
   cell tag editor by default. This lets you add cell tags to each cell quickly.

   To enable the cell tag editor, go click `View -> Cell Toolbar -> Tags`. This
   will enable the tags UI. Here's what the menu looks like.

   ![tags_notebook](../images/tags_notebook.png)

2. **Use the JupyterLab Cell Tags plugin**. JupyterLab is an IDE-like Jupyter
   environment that runs in your browser. It has a "cell tags" plugin built-in,
   which exposes a user interface that lets you quickly insert cell tags.

   You'll find tags under the "wrench" menu section.
   Here's what the tags UI in JupyterLab looks like.

   ![tags_jupyterlab](../images/tags_jupyterlab.png)

Tags are actually just a special section of cell level metadata.
There are three levels of metadata:

* For notebook level: in the Jupyter Notebook Toolbar go to `Edit -> Edit Notebook Metadata`
* For cell level: in the Jupyter Notebook Toolbar go to `View -> Cell Toolbar -> Edit Metadata` and a button will appear above each cell.
* For output level: using e.g. `IPython.display.display(obj,metadata={"tags": [])`, you can set metadata specific to a certain output (but jupyter-book does not utilise this just yet).

![NB Metadata GIF](../images/metadata_edit.*)

### Add tags to notebook cells based on their content

Sometimes you'd like to quickly scan through a notebook's cells in order to
add tags based on the content of the cell. For example, you might want to
hide any cell with an import statement in it using the `remove-input` tag.

Here's a short Python snippet to accomplish something close to this.
First change directories into the root of your book folder, and then
run the script below as a Python script or within a Jupyter Notebook
(modifying as necessary for your use case).
Finally, check the changes that will be made and commit them to your repository.

```python
import nbformat as nbf
from glob import glob

# Collect a list of all notebooks in the content folder
notebooks = glob("./content/**/*.ipynb", recursive=True)

# Text to look for in adding tags
text_search_dict = {
    "# HIDDEN": "remove-cell",  # Remove the whole cell
    "# NO CODE": "remove-input",  # Remove only the input
    "# HIDE CODE": "hide-input"  # Hide the input w/ a button to show
}

# Search through each notebook and look for th text, add a tag if necessary
for ipath in notebooks:
    ntbk = nbf.read(ipath, nbf.NO_CONVERT)

    for cell in ntbk.cells:
        cell_tags = cell.get('metadata', {}).get('tags', [])
        for key, val in text_search_dict.items():
            if key in cell['source']:
                if val not in cell_tags:
                    cell_tags.append(val)
        if len(cell_tags) > 0:
            cell['metadata']['tags'] = cell_tags

    nbf.write(ntbk, ipath)
```

(raw-html-in-markdown)=
## Use raw HTML in Markdown

Jupyter notebook markdown allows you to use pure HTML in markdown cells.
This is discouraged in most cases,
because it will usually just be passed through the build process as raw text, and so will not be subject to processes like:

- Relative path corrections
- Copying of assets to the build folder
- Multiple output type formatting (e.g. it will not show in PDFs!).

So, for instance, below we add, and you will find that the HTML link is broken:

```md
 <a href="../intro.md">Go Home HTML!</a>

 [Go Home Markdown!](../intro.md)
```

 <a href="../intro.md">Go Home HTML!</a>

 [Go Home Markdown!](../intro.md)

:::{tip}
Note that MyST markdown now has some extended syntax features,
which can allow you to use certain HTML elements in the correct manner.

Such as:

```html
<img src="../images/fun-fish.png" alt="the fun fish!" width="200px"/>
```

<img src="../images/fun-fish.png" alt="the fun fish!" width="200px"/>

See the [image appearance section](content-blocks-images) for details.
:::

## Adding extra HTML to your book

There are a few places in Jupyter Book where you can add arbitrary extra HTML.
In all cases, this is done with a configuration value in your `_config.yml` file.

### Extra HTML in your footer

To add extra HTML in your book's footer, use the following configuration:

```yaml
html:
    extra_footer: |
        <div>
            your html
        </div>
```

The contents of `extra_footer` will be inserted into your page's HTML after
all other footer content.

### Extra HTML to your left navbar

To add extra HTML in your book's left navbar, use the following configuration:

```yaml
html:
    extra_navbar: |
        <div>
            your html
        </div>
```

The contents of `extra_navbar` will be inserted into your page's HTML after
all other html content.

## Adding a license to your HTML footer

If you'd like to add a more detailed license for your book, or would like to
add a link to an external page for a license, the easiest way to do so is to
use a custom footer for this. You can disable the "copyright" text that is
automatically added to each footer, and add whatever footer HTML you'd like.

For example, see this configuration:

```yaml
html:
  extra_footer: |
    <p>
    ... Add license info here...
    </p>
sphinx:
  config:
    html_show_copyright: false
```

Note that this may not work in latex-generated PDF builds of your page.

(working-on-windows)=
## Working on Windows

Jupyter Book is now also tested against a Windows environment on Python 3.7 😀

For its specification, see the [`windows-latest` runner](https://docs.github.com/en/actions/reference/virtual-environments-for-github-hosted-runners#supported-runners-and-hardware-resources) used by GitHub CI.

However, there is a known incompatibility for notebook execution, when using Python 3.8
(see issue [#906](https://github.com/executablebooks/jupyter-book/issues/906)).

If you're running a recent version of Windows 10 and encounter any issues, you may also wish to try
[installing Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

As of June 5, 2020, there were three open issues that required Windows-specific changes.
We hope these are now fixed in jupyter-book version 0.8 but, in case any issues still arise,
leave these community tips, which are known to work for some users.
Note that there is no guarantee that they will work on all windows installations.

1. Character encoding

    Jupyter-book currently reads and writes files on windows in the native windows
    encoding, which causes encoding errors for some characters in UTF-8 encoded
    notebooks.

    **Work-around:**  Beginning with
    [Python 3.7](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONUTF8)
    cmd.exe or powershell enviroments that set PYTHONUTF8=1  override the native
    locale encoding and use UTF8 for all input/output.

    :::{tip}
    To make it easier to use this option,
    the EOAS/UBC notebook courseware project has created a Conda package [runjb](https://anaconda.org/eoas_ubc/runjb) which [does this automatically for powershell](https://github.com/eoas-ubc/eoas_tlef/blob/master/converted_docs/wintools/binwin/runjb.ps1)
    :::

2. A new windows event loop

   The asyncio event loop [has been changed for Python 3.8](https://github.com/sphinx-doc/sphinx/issues/7310)
   causing sphinx-build to fail.

   **Work-around:**  Pin to Python 3.7.6.  This
   [environment_win.yml](https://github.com/eoas-ubc/quantecon-mini-example/blob/windows/environment_win.yml)
   file does that, and also installs runjb to fix issue 1.

3. Nested tables of contents

   Currently, `_toc.yml` files that reference markdown files
   in sub-folders are failing for some windows users.  That is, this
   [original _toc.yml](https://github.com/eoas-ubc/quantecon-mini-example/blob/master/mini_book/_toc.yml)
   file will fail with a message saying jupyter-book "```cannot find index.md```"

   **Work-around**: Flatten the layout of the book to a single level, i.e.
   [this _toc.yml](https://github.com/eoas-ubc/quantecon-mini-example/blob/windows/mini_book/docs/_toc.yml)
   file works with windows.

**Summary**

The following workflow should succeed using a miniconda powershell terminal on Windows 10:

1. `conda install git`
2. `git clone https://github.com/eoas-ubc/quantecon-mini-example.git`
3. `cd quantecon-mini-example`
4. `git checkout windows`
5. `conda env create -f environment_win.yml`
6. `conda activate wintest`
7. `cd mini_book`
8. `runjb docs`

After the build, view the html with:

`start docs\_build\html\index.html`

## What if I have an issue or question?

If you've got questions, concerns, or suggestions, please open an issue at
[at the jupyter book issues page](https://github.com/executablebooks/jupyter-book/issues)
