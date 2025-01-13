# qScope_doc

Documentation of the Q-Scope framework by quarree100 @FGRes.
The built version can be found at [q-scope.readthedocs.io](https://q-scope.readthedocs.io)

## install

1. setup python venv: `python3 -m venv qscope_doc && source qscope_doc/bin/activate`
2. install dependencies: `pip3 -r requirements.txt`
3. build project: `sphinx-build -b html source/ build/`


## known issues with sphinx

- external links must include http(s):// prefix!
