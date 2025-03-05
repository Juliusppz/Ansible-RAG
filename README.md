# RAG Service Deployment with Ansible & GitHub Actions

This repository deploys a RAG (Retrieval Augmented Generation) service to Google Cloud Platform (GCP) instances using Ansible. 
It supports both manual (local) execution and automated, versioned deployments via GitHub Actions.

---

## Overview

- **Local Deployment:** Run the Ansible playbook manually to deploy the service.
- **CI/CD Deployment:** Trigger deployments via GitHub Actions when you push versioned tags (e.g., `v1.0.0`, `v2.1.3`) or manually using the workflow_dispatch event.

---

## Prerequisites

- **Google Cloud Account:** Ensure you have a GCP account and a service account with permissions to manage instances and update metadata.
- **GCP Credentials:** A JSON file for your service account.  
- **Ansible:** Installed locally along with Python3 and pip.
- **Gcloud CLI:** (Optional for local debugging) Installed if you need to run gcloud commands manually.
- **Docker & Docker Compose Plugin:** Installed on target instances (handled by the playbook).
- **GitHub Repository Secrets:**  
  - `GCP_CREDENTIALS`: Your GCP service account JSON.
  - `GCP_PROJECT`: Your GCP project ID.
  - `API_KEY`: An API key used during deployment.

---

## Running Locally

### 1. Install Local Dependencies

On an Ubuntu-based system, install Ansible and required packages:

```bash
sudo apt update && sudo apt install -y ansible python3-pip
pip3 install --user --upgrade ansible
ansible-galaxy collection install google.cloud
```

### 2. Set Environment Variables

Export your credentials and project details as environment variables:

```bash
export GCP_PROJECT=your-gcp-project-id
export GCP_CREDENTIALS="$(cat /path/to/your/gcp_credentials.json)"
export API_KEY=your_api_key
```

*Ensure that `GCP_CREDENTIALS` contains the full JSON content of your service account file.*

### 3. Run the Playbook

Execute the playbook with the provided inventory file:

```bash
ansible-playbook playbook.yml -i inventory/gcp_compute.yaml
```

This command connects to your GCP instances (filtered in `inventory/gcp_compute.yaml`) and performs the deployment tasks.

---

## GitHub Actions Deployment

The workflow file at `.github/workflows/deploy.yml` automates deployments on GCP.

### Setup in GitHub

1. **Add Secrets:**  
   In your repository settings, add the following secrets:
   - `GCP_CREDENTIALS`
   - `GCP_PROJECT`
   - `API_KEY`

2. **Trigger a Deployment:**

   - **Versioned Deployments:**  
     Push a tag to your repository:
     ```bash
     git tag v1.0.0
     git push origin v1.0.0
     ```
     The workflow triggers automatically on tags starting with `v`.

   - **Manual Trigger:**  
     You can also run the workflow manually via the GitHub Actions UI (using the `workflow_dispatch` event).

---

## Project Structure

- **.github/workflows/deploy.yml:**  
  Configures the GitHub Actions workflow to authenticate with GCP, install dependencies, and run the Ansible playbook.

- **inventory/gcp_compute.yaml:**  
  Uses the GCP compute plugin to dynamically create an inventory from your GCP instances, filtered by the production environment label.

- **playbook.yml:**  
  Main Ansible playbook that prepares the server, fetches instance metadata, and updates GCP instance metadata with the API key.

- **roles/rag_server/**  
  Contains tasks for:
  - Updating the apt cache.
  - Installing Docker and prerequisites.
  - Cloning the RAG service repository.
  - Re-updating instance metadata with the API key.

---

## Troubleshooting & Notes

- **Permissions:**  
  Verify your service account has permissions to manage instances and update metadata in GCP.

- **Network:**  
  Ensure that your local machine (or the GitHub Actions runner) can access your GCP instances.

- **Task Duplication:**  
  Note that the playbook updates instance metadata with the API key in both the main playbook and within the role. This may be redundant; adjust as necessary for your environment.

- **Logs:**  
  Check the Ansible and GitHub Actions logs for any errors during deployment.

---

## Contributing

Contributions are welcome! Feel free to fork the repository, create issues, or submit pull requests. For major changes, please open an issue first to discuss your ideas.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
