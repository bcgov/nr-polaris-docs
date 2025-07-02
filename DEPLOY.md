# Deploy using the pipeline

This guide will help you use the pipeline to deploy a Java/Maven or Node.js artifact.

## 1. Prerequisites

- A deployable artifact published to GitHub Packages.
- NR Broker account linked to GitHub (see [How to Link Your Broker Account to GitHub](https://apps.nrs.gov.bc.ca/int/confluence/display/OSCAR/Linking+to+a+GitHub+account)).

## 2. Configure Application Settings

Applications are configured for each environment using Ansible variables defined in `playbooks/vars/custom/<env>.yaml` files, where `<env>` is one of (`dev`, `test`, `prod`, or `all`).

For example, an application may require a specific version of Java (`jdk_major_version`), Tomcat (`tomcat_major_version`) or a particular WebADE datastore (`webade_datastore`).

Refer to the documentation for the Ansible roles used in [bcgov/nr-polaris-collection](https://github.com/bcgov/nr-polaris-collection/blob/main/README.md) for a list of configurable variables and their behaviour.

**Important:** Do not store sensitive information (such as passwords, API keys, secrets, etc.) directly in variable files. Instead, store sensitive values in [NR Vault](https://knox.io.nrs.gov.bc.ca). Vault key-value pairs are available during deployments as environment variables prefixed with `PODMAN_` and they can be referenced using the `ansible.builtin.env` lookup plugin.

For example, to reference the value for the key `database_password`, add the following line to a custom yaml file:

```
database_password: "{{ lookup('ansible.builtin.env', 'PODMAN_database_password') }}"
```

## 3. Add custom Jinja2 templates

Configuration files are stored in the `playbooks/templates/` directory and should be updated with the required template variables.

Each template must also be referenced in the `playbooks/custom-tasks.yaml` file.

The following is an example template for a Tomcat context.xml file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context>
    <Parameter name="APP_ENV" value="{{ app_env }}"/>
</Context>
```

The value for the application environment is placed in `playbooks/vars/context/dev.yaml`:

```
app_env: development
```

Add an item for the template to the `with_items` list in the `playbooks/custom-tasks.yaml` file:

```yaml
---
- name: configure nonstandard files
  template:
    src: "{{ playbook_dir }}/templates/{{ item.src }}"
    dest: "{{ item.dest }}"
    mode: "0775"
  become: yes
  become_user: "{{ polaris_install_user }}"
  with_items:
    - {
        src: "context.xml.j2",
        dest: "{{ polaris_apps_service_install_home }}/webapps/{{ alt_app_dir_name | default(context) }}/META-INF/context.xml"
      }
```

## 4. Trigger a development build

A development build is the first step before deploying code to production. The development build is based on a pseudo-merge of the target branch and `main`. Development builds are triggered automatically by opening a pull request with the `main` branch as the base. They can also be triggered manually when `main` is the target branch, or when there is an open pull request with the `main` branch as the base. Do the following steps to trigger a development build manually:

**Prerequisites:**

- Open pull request for the target branch to merge to `main` OR the target branch is `main`.
- No merge conflicts.

1. Go to the **Actions** tab in the GitHub repository.
2. Select the `Build and release` workflow.
3. Choose the target branch.
4. Click **"Run workflow"** to start the build.

After triggering a build, click the workflow run in the **Actions** tab to view logs and progress.

## 5. Deploy a development build

Development builds can only be deployed to the dev and test environments. Do the following steps to trigger a deployment of the target branch to the dev or test environments:

**Prerequisites:**

- the build artifact has been built by the build-release workflow (see above: 1. Trigger a development build)
- the target branch is `main`; or there exists an open pull request for the target branch to merge to `main` without merge conflicts

**Trigger a deployment to the dev and test environments:**

1. Go to the **Actions** tab in your GitHub repository.
2. Select the `Run Deploy` workflow.
3. Select the branch to deploy and the target environment
4. Click **"Run workflow"** to start deployment.
5. Click the link to the deployment job in the workflow logs to view the deployment job progress.
6. The workflow will automatically deploy to development.
7. If the deployment is targeted for test, the workflow will pause for approval before proceeding with the deplyoment to test.

## 6. Trigger a release build

When ready to deploy code to production, create a release:

  - Go to the Releases page of your repository (eg. https://github.com/<org>/<repo>/releases).
  - Click on **"Draft a new release"**.
  - Select the tag you just pushed, or create a new one.
  - Fill in the release title and description (e.g., changelog, highlights).
  - (Optional) Attach binaries or other assets.
  - Click **"Publish release"**.

The Build and release workflow will be triggered automatically to build the release. After it builds successfully, you may trigger a deployment of the release to production.

## 7. Deploy a production build

1. Go to the **Actions** tab in your GitHub repository.
2. Select the Deploy workflow.
3. Select the tag (release) to deploy.
4. Click **"Run workflow"** to start deployment.
5. Click the link to the deployment job in the workflow logs to view the deployment job progress.
6. The workflow will automatically deploy to development.
7. The workflow will pause for approval before proceeding with each deployment to test and prod.
