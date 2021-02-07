Python 3 with VSCode:
1. Install pyenv
https://realpython.com/intro-to-pyenv/:
```
brew install pyenv
pyenv install --list
pyenv install -v 3.8.3
ls ~/.pyenv/versions/
pyenv versions
python -V
which python
pyenv which python
pyenv global 3.8.3
python -m test
pyenv which python
python -V
```
2. Install the Python extension for Visual Studio Code.
3. Select `pyenv` as the python interpreter in VSCode (F1)

```
mkdir project_dir
cd project_dir
python3 -m venv project_dir_env
source project_dir_env/bin/activate
which python or pip list # to confirm that the env is working
pip3 install package_name 
pip freeze > requirements.txt
pip install -r requirements.txt
deactivate
```
or

```
cd project_dir
virtualenv -p python3 venv
source venv/bin/activate
pip install .
python3 tests/core_test.py
```

## Serve files from local directory:
```
# python -m SimpleHTTPServer 80
```

## Interactive TTYs with autocompletion:
```
python -c 'import pty; pty.spawn("/bin/bash")'
CTRL+Z
stty raw -echo
fg
[ENTER][ENTER]
```
