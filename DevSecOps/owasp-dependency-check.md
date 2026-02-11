
## What is OWASP Dependency-Check 

OWASP Dependency-Check is a free, open-source Software Composition Analysis (SCA) tool that identifies project dependencies and checks them for known, publicly disclosed vulnerabilities. It helps developers address "Vulnerable and Outdated Components," which is a primary security risk on the OWASP Top 10 list



## Prepare Your Python Project

```bash
deactivate
mkdir owasp-demo && cd owasp-demo
```

```bash
python3.13 -m venv owasp
source owasp/bin/activate
```
Generate a vulnerable requirements.txt

```cat > requirements.txt```

```bash
requests==2.20.0
flask==0.12.2
jinja2==2.10

```
_(Note: These versions are old and contain known CVEs like CVE-2018-18074 for requests.)_

Install the application requirement from the requirement.txt file

```pip install -r requirement.txt```

Create a dummy `app.py` code

```cat > app.py```

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

## Set Up Docker Directories

The scanner needs a persistent location to store its vulnerability database (NVD data) so it doesn't re-download several gigabytes every time you run it. 

Create a data and a reports directories

```bash
mkdir -p odc-data
mkdir -p odc-reports
```
## Get a Free NVD API Key - Optional

- Visit the NVD API Key Request page
    https://nvd.nist.gov/developers/request-an-api-key

- Provide your email and organization details.

- Activate the key via the link sent to your email.


## Run the Scan

OWASP Dependency-Check support for Python is currently categorized as **experimental**. To enable to scan Python files, use the `--enableExperimental` flag.

Execute the following Docker/Podman Command from your project root.

```bash
# disable temporarily SELinux
sudo setenforce 0

# Run the owasp dependency-check container
podman run --rm \
    --volume $(pwd):/src \
    --volume $(pwd)/odc-data:/usr/share/dependency-check/data \
    --volume $(pwd)/odc-reports:/report \
    owasp/dependency-check:latest \
    --project "Python Demo Scan" \
    --scan /src \
    --format "HTML" \
    --out /report \
    --enableExperimental \
    --nvdApiKey 0c8a7bb6-7f7f-45fc-8c69-a858ffc4c332
```

**- `--enableExperimental`:** Required to activate the Python Analyzer.
**- `--scan /src`:** Points the scanner to the mounted volume containing your code.
**- `--out /report`:** Saves the results into your local odc-reports folder.
**- `-v ...:/usr/share/dependency-check/data`:** Persists the NVD database locally to save time on future scans.


## Run the Scan

Once the scan finishes, open the generated report in your browser.

**Path:** `odc-reports/dependency-check-report.html` 
The report will list the specific **CVE IDs**, **Severity Scores**, and **Vulnerable Dependencies** found in your `requirements.txt`
