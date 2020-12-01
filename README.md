# Deploy ML API serving predictions

Here you'll be able to deploy a minimal API to return predictions from pretrained model.

## Prerequisite

In the following, we suppose that:

1. You have previously trained a pipeline
2. You have saved this model either on GCP Cloud Storage or inside `data/`
3. If your pipeline includes custom transformers, they should be present inside  `TaxiFaremodel/encoders.py`
    🚨 This point is very important here, please call TA if unclear 🚨
4. You have installed [heroku cli](https://devcenter.heroku.com/articles/heroku-cli)
5. you have created an [free heroku account](https://signup.heroku.com/)

## Summary

Here you will deploy an api that will:
- load a `model.joblib` using `pipeline = joblib.load('model.joblib')`
  👉 either locally or directly from GCP Storage
  👉 this `model.joblib` contains the whole pipeline (preprocssing + model)
- receive through route `/predict_fare`, jsons looking like:
```python
input = {"pickup_datetime": 2012-12-03 13:10:00 UTC,
        "pickup_latitude": 40.747,
        "pickup_longitude": -73.989,
        "dropoff_latitude": 40.802,
        "dropoff_longitude": -73.956,
        "passenger_count": 2}
```
- apply predictions
- return predictions

# First local API

We will use `flask` python package to develop our API
Lets also install the requirements provided inside `requirements.txt`
```bash
pip install flask flask_cors
pip install -r requirements.txt
```

Inspect `app.py` and check the code of two routes `/predict_fare` and `/`.

For simplicity purpose you will develop your first api locally with a local `model.joblib` stored in `data/` directory that we provide you.
We will see later on how to get your stored model from gcp.

Now you can run your api locally by simply running:
```bash
python app.py
```

And check that it works by pinging it: [http://127.0.0.1:8080/](http://127.0.0.1:8080/)

## Use your api te obtain predictions

Open `Predict.ipynb` under `jupy` folder and start interrogating your api

# Deploy to heroku

Now that you checked your app works locally, you might want it to run free on a remote server.

## Create folder outside data-challenges
Here as heroku is based on git, we will create a folder outside our gitted `data-challenges` folder.
```bash
mkdir -p ~/code/taxifare_api
cp -rf * ~/code/taxifare_api/
cd ~/code/taxifare_api/
```
Version it for future heroku use:
```bash
cd ~/code/taxifare_api/ && git init
```

 ## Deploy to heroku
 You'll see once again how heroku is easy to use, here you simply need to:
 - Create a `Procfile` with following line inside it:
 ```bash
 web: gunicorn app:app
 ```
 ```bash
 git add . && git commit -am "add api first version"
 ```

- Login to heroku
```bash
heroku login
```

- Create an app on heroku
```bash
heroku create YOUR_APP_NAME
```

- Note that a heroku remote has been added to your git repository. This allows us to push the code to heroku using `git push heroku master`
```bash
git remote -v
```

- Build the app on heroku servers using git
```bash
git push heroku master
```

- Then deploy the app
```bash
heroku ps:scale web=1
```
click on the link `https://YOUR_APP_NAME.herokuapp.com/` and it should return `OK`

# Bonus: Load your own model from GCP
🛑 You can skip this step and come back to it after exercice 2 and/or 3 🛑

Here we will simply add a function downloading your `model.joblib` from your own storage.

Inspect new functions under `TaxiFareModel/gcp.py`:
- `download_model()` to get file from storage
- `get_credentials()` to read your json GCP credentials from your env variable `GOOGLE_APPLICATION_CREDENTIALS`

Replace following variable inside `TaxiFareModel/params.py`
- `BUCKET_NAME`, `PROJECT_ID`, and `MODEL_VERSION`
  👉 the most important is  `MODEL_VERSION` indicating folder inside your bucket where you previously uploaded your model
- Make sure that the `storage_location` built from these variables in `gcp.py` line 40 matches the path of 'model.joblib' in your gcp bucket.

Test that you correctly get your model from GCP:
```bash
python -m TaxiFareModel.gcp
```
or with `ipython`
```bash
$ ipython
Python 3.7.2 (default, Feb 20 2020, 16:34:30)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.12.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: %run TaxiFareModel/gcp.py
```

# Bonus: Add route to change model

Inspect already implemented route `/set_model` inside `app.py`:
- What does it do ?
- Test it by launching your app locally
- Deploy this new version on Heroku

⚠️ Beware here, you will need to indicate to heroku your GCP credentials, for that, inspect following `heroku_set_gcp_env` command from `Makefile`, or run:
```make
heroku config:set GOOGLE_APPLICATION_CREDENTIALS="$(cat $GOOGLE_APPLICATION_CREDENTIALS)"
```
and run it right after you redeployed your app
```bash
make deploy_heroku heroku_set_gcp_env
```

⚠️ If you get an error in your web app, remember to check the logs:
```bash
heroku logs --tail
```
