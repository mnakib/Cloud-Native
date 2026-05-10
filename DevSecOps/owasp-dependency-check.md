
## What is OWASP Dependency-Check 

OWASP Dependency-Check is a free, open-source **Software Composition Analysis (SCA)** tool that identifies project dependencies and checks them for known, publicly disclosed vulnerabilities. It helps developers address _"Vulnerable and Outdated Components"_, which is a primary security risk on [the OWASP Top 10 list](https://owasp.org/Top10/2025/).



## Prepare Your Python Project

```bash
deactivate
cd
mkdir ~/owasp-depcheck-demo
cd ~/owasp-depcheck-demo
```

```bash
python -m venv .venv
source .venv/bin/activate
```
Generate a vulnerable requirements.txt file

```bash
cat > requirements.txt
```

```bash
requests==2.20.0
flask==0.12.2
jinja2==2.10
```
>   _Note: These versions are old and contain known CVEs like CVE-2018-18074 for requests._

Install the application requirement from the requirement.txt file

```bash
pip install -r requirements.txt
```

Create a dummy `app.py` code

```bash
cat > app.py
```

```bash
import flask
import requests

app = flask.Flask(_name_)

@app.route('/')
def hello():
    return "Checking for vulnerabilities!"

if _name_ == '_main_':
    app.run()
```

## Setting Up Docker Scan Report Directories

The scanner needs a persistent location to store its vulnerability database (NVD data) so it doesn't re-download several gigabytes every time you run it. 

Create a data and a reports directories

```bash
mkdir -p odc-data
mkdir -p odc-reports
```
## Get a Free NVD API Key - Optional

- Visit the NVD API Key Request page
    https://nvd.nist.gov/developers/request-an-api-key

- Provide your email and organization details. Personal emails also work.

- Activate the key via the https://nvd.nist.gov/developers/confirm-api-key link sent to your email.


## Run the Scan

OWASP Dependency-Check support for _Python_ is currently categorized as **experimental**. To enable to scan Python files, use the `--enableExperimental` flag.

Execute the following Docker/Podman Command from your project root.

```bash
# disable temporarily SELinux
sudo setenforce 0
```

```bash
APIKey=<TheGeneratedAPIKey>
# Run the owasp dependency-check container
docker run --rm \
    -- name owasp-dependency-check \
    --volume $(pwd):/src \
    --volume $(pwd)/odc-data:/usr/share/dependency-check/data \
    --volume $(pwd)/odc-reports:/report \
    owasp/dependency-check:latest \
    --project "Python Demo Scan" \
    --scan /src \
    --format "HTML" \
    --out /report \
    --enableExperimental \
    --nvdApiKey ${APIKey}
```

- **`--enableExperimental`:** Required to activate the Python Analyzer.

- **`--scan /src`:** Points the scanner to the mounted volume containing your code.

- **`--out /report`:** Saves the results into your local odc-reports folder.

- **`-v ...:/usr/share/dependency-check/data`:** Persists the NVD database locally to save time on future scans.


## Check the Scan

Once the scan finishes, open the generated report in your browser.

**Path:** `odc-reports/dependency-check-report.html` 
The report will list the specific **CVE IDs**, **Severity Scores**, and **Vulnerable Dependencies** found in your `requirements.txt`
