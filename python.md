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

## Serve files from local directory:
```
# python -m SimpleHTTPServer 80
```