# docs
Documentation related to code used by and created by PCLAP.

This repo uses the `mkdocs` python package.  To use the same packages that have been tested already, you can set up a virtual environment.   After cloning this repository:

```
cd plap-docs
python -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
```

**Note:** You will have to activate the virtual environment by "sourcing" the `.venv/bin/activate` file every time you want to use this repo; however, this repo's packages will be self-contained and not affected by other python projects you may have on your machine, which is nice.  

Or you may already have `mkdocs` installed; or you may wish to install it yourself, using the [`mkdocs` instructions](https://www.mkdocs.org/getting-started/).

Once you've got your python environment set up, to view the documentation that this repo holds, navigate to the folder that contains `mkdocs.yml` and run `mkdocs serve`.

```
cd pclap-docs/pclap-e3sm
mkdocs serve
```

**Math:** You can write mathematical expressions via [MathJax](https://docs.mathjax.org/en/latest/) javascript with inline delimiters `\(` `\)`, e.g., `\(f(x) = x^2\)`; a centered equation can be written between $$ and $$,

```
$$ g(x) = {x^3} \over 3 $$
```

Then open the locally hosted server `http://127.0.0.1:8000/` in your browser.
