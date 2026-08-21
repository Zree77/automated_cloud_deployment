# From Local Code to Automated Cloud Deployment — Lab Walkthrough

## Part 1 — Git & Docker

```bash
# 3. Build and run locally
docker build -t lab-web:latest .
docker run -d --name lab-web -p 8080:80 lab-web:latest

# verify
curl http://localhost:8080
# or open http://localhost:8080 in a browser
```

```bash
# 4. Push to a private GitHub repo
git init
git add .
git commit -m "Initial commit: Dockerized web page"
git branch -M main
git remote add origin git@github.com:<your-username>/<your-repo>.git
git push -u origin main
```
Create the repo on GitHub first (Settings → visibility: Private) or via `gh repo create <name> --private`.

## Part 2 — Terraform & Cloud

```bash
cd terraform
cp terraform.tfvars.example terraform.tfvars
# edit terraform.tfvars: set key_name to an existing AWS key pair, and your IP for ssh_cidr

terraform init
terraform plan
terraform apply        # type "yes" to confirm

# grab the public IP for Part 3
terraform output public_ip
```
This provisions a `t2.micro` EC2 instance and uses the `user_data` script to install and start Docker automatically on boot — check the AWS Console (EC2 → Instances) to confirm it's running.

## Part 3 — Ansible & Configuration

```bash
cd ../ansible
# edit inventory.ini and put the terraform output public_ip in place of 203.0.113.10

# quick connectivity check
ansible web -m ping

# push the image name to Docker Hub first if not already:
#   docker tag lab-web:latest <dockerhubuser>/lab-web:latest
#   docker push <dockerhubuser>/lab-web:latest
# then update image_name in playbook.yml to match

ansible-playbook playbook.yml
```

Verify:
```bash
curl http://<public_ip>:8080
# or open http://<public_ip>:8080 in a browser
```

## Part 4 — Jenkins Pipeline

1. In Jenkins: **New Item → Pipeline**, name it e.g. `lab-web-pipeline`.
2. Under **Pipeline → Definition**, choose "Pipeline script from SCM", SCM = Git, point it at your GitHub repo, credentials as needed, script path = `Jenkinsfile`.
3. (Optional) Add a GitHub webhook so pushes trigger builds automatically: repo → Settings → Webhooks → Payload URL `http://<jenkins-host>/github-webhook/`.
4. Save, then **Build Now** to confirm the pipeline runs Build → Test stages successfully.
5. Make a small edit to `app/index.html` (e.g. change the version text), then:
   ```bash
   git add app/index.html
   git commit -m "Update homepage text"
   git push
   ```
6. Watch Jenkins pick up the change (via webhook or polling) and go green.

## Checklist mapping

- Repo history: Part 1 Task 4
- `terraform apply` clean: Part 2 Task 3, verified via `terraform output public_ip` + AWS Console
- Site reachable on Public IP: Part 3 Task 3 (`curl http://<public_ip>:8080`)
- Jenkins green after code change: Part 4 Task 3
