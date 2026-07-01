This repository demonstrates a Fabric CI/CD scenario using [Fab Inspector](https://github.com/NatVanG/fab-inspector) to run rules-based validation against Fabric items under source control.

## Repository Layout

- `src/`: Fabric artifacts to validate (reports, semantic model, notebook, pipeline).
- `fab-inspector-rules/`: Rule sets used by Fab Inspector.
- `.github/workflows/`: GitHub Actions workflow examples.

## GitHub Workflow Examples

This repo includes four manual (`workflow_dispatch`) examples:

- `.github/workflows/fab-inspector-docker.yml`
	- Uses the published container image `ghcr.io/natvang/fab-inspector:latest`.
	- Mounts the repository folders into the container:
		- `${{ github.workspace }}/src` -> `/src`
		- `${{ github.workspace }}/fab-inspector-rules` -> `/rules`
	- Runs:
		- `-fabricitem /src`
		- `-rules /rules/sample-report-rules.json`
		- `-formats GitHub`
	- Best when you want a simple, container-based setup with no binary management.

- `.github/workflows/fab-inspector-docker-clientsecret.yml`
	- Variant of the Docker workflow that authenticates using a service principal (client secret).
	- Uses the same container image and volume mounts as the standard Docker workflow.
	- Additionally passes `-authmethod clientsecret` along with `FABRIC_CLIENT_ID`, `FABRIC_TENANT_ID`, and `FABRIC_CLIENT_SECRET` secrets.
	- Writes results to a OneLake output folder (`ONELAKE_OUTPUT_FOLDER` variable) and uses `-formats "GitHub,JSON"`.
	- Provides authenticated access to Fabric resources and want to persist results to OneLake.

- `.github/workflows/fab-inspector-cli-download.yml`
	- Downloads and runs the Linux Fab Inspector CLI release directly in the runner.
	- Accepts an optional `version` input:
		- `latest` (default) resolves the latest GitHub release tag.
		- Or specify an explicit tag, for example `v3.4.0`.
	- Downloads `linux-x64-FabInspCLI.zip`, unzips, marks executable, and runs:
		- `-fabricitem ./src`
		- `-rules ./fab-inspector-rules/sample-report-rules.json`
		- `-formats GitHub`
	- Best when you want explicit control over the Fab Inspector version used in CI.

- `.github/workflows/fab-inspector-cli-download-clientsecret.yml`
	- Variant of the CLI download workflow that authenticates using a service principal (client secret).
	- Accepts the same optional `version` input as the standard CLI workflow.
	- Uses a reusable action (`.github/actions/download-fab-inspector`) to handle the download and install step.
	- Additionally passes `-authmethod clientsecret` along with `FABRIC_CLIENT_ID`, `FABRIC_TENANT_ID`, and `FABRIC_CLIENT_SECRET` secrets.
	- Writes results to a OneLake output folder (`ONELAKE_OUTPUT_FOLDER` variable) and uses `-formats "GitHub,JSON"`.
	- Provides authenticated access to Fabric resources and want version-pinned CLI with results persisted to OneLake.

## How To Use The Workflows

1. Push this repository (or your fork) to GitHub.
2. Open **Actions** in your repo.
3. Choose one of these workflows:
	 - `fab-inspector-docker`
	 - `fab-inspector-docker-clientsecret`
	 - `fab-inspector-cli`
	 - `fab-inspector-cli-clientsecret`
4. Click **Run workflow**.
5. For `fab-inspector-cli` or `fab-inspector-cli-clientsecret`, optionally set `version` (for example `v3.3.0`) and run.
6. For the `clientsecret` variants, ensure the `FABRIC_CLIENT_ID`, `FABRIC_TENANT_ID`, and `FABRIC_CLIENT_SECRET` secrets and the `ONELAKE_OUTPUT_FOLDER` variable are configured in your repository settings. :warning: **Do not commit secrets to source control.** Use GitHub secrets for sensitive values.
6. Review results in the workflow logs and annotations (GitHub format output).

## Customizing For Your Project

- Change `-fabricitem` to target the folder that contains your Fabric item definitions.
- Point `-rules` to your own rule JSON file.
- Keep `-formats GitHub` when running in Actions so failures are visible in the run output.
- If you need to run automatically on commits/PRs, add triggers such as:
	- `push`
	- `pull_request`

## Notes

- Both examples run on `ubuntu-latest`.
- The Docker workflow always uses the image `latest` tag.
- The CLI workflow lets you pin to a specific release tag for reproducible CI behavior.
