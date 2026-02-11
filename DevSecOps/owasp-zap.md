## What is OWASP?

The Open Web Application Security Project (OWASP) is an open, online community that creates methodologies, tools, technologies and guidance on how to deliver secure web applications. It is an international collaborative initiative comprised of both individuals and corporations. The project aims to standardise security approaches in web development and spread associated knowledge.


## What is OWASP ZAP?

OWASP ZAP (ZAP) is one of the world’s most popular free security tools and is actively maintained by hundreds of international volunteers. It can help to find security vulnerabilities in web applications. It’s also a great tool for experienced pen testers and beginners.

ZAP can scan through the web application and detect issues related to:

- SQL injection
- Broken Authentication
- Sensitive data exposure
- Broken Access control
- Security misconfiguration
- Cross Site Scripting (XSS)
- Insecure Deserialization
- Components with known vulnerabilities
- Missing security headers 


## Prepare Your Python Project

```bash
deactivate
mkdir owasp-zap && cd owasp-zap
```

```bash
python3.13 -m venv owasp-zap
source owasp-zap/bin/activate
```

## Creating the Containerized Flask Application

Create a dummy Flask `app.py` application

```cat > app.py```

```bash
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "<h1>Secure App</h1><p>Scanning with ZAP!</p>"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Creating an image using a Containerfile

```cat > Containerfile```

```Dockerfile
FROM python:3.9-slim
WORKDIR /app
RUN pip install flask
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

Build the Flask container image

```bash
# Create the image
podman build -t flask-app .

# Check that the image is created
podman images
```

Create a podman network 

```bash
# Create the network
podman network create zap-net

# List the existing networks
podman network ls
```

_By creating creating the zap-net and using it to create the flask app container, ZAP resolves http://flask-web:5000 internally without needing to expose ports to your host_


Run the container from flask-app image

```bash
# Run the container app
podman run -d --name flask-web-bis -p 5000:5000 --network zap-net flask-app

# Check the container is running
podman image ls
```


## Run the OWASP ZAP Scan 

```bash
podman pull zaproxy/zap-stable:latest
```

```bash
podman run --network zap-net -v $(pwd):/zap/wrk/:rw \
    zaproxy/zap-stable zap-baseline.py \
    -t http://flask-web:5000 \
    -r flask-report.html
```

## Run the OWASP ZAP Interactive Desktop GUI (Webswing)

This is the easiest way to use the full ZAP interface without installing Java locally. It runs ZAP in a browser via Webswing.

```bash
podman run -u zap -p 8080:8080 -p 8090:8090 -d --network zap-net -i zaproxy/zap-stable zap-webswing.sh
```

Browse http://localhost:8080/zap/

Quick start > http://flask-web:5000


