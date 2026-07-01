This repository demonstrates a Fabric CI/CD scenario using [Fab Inspector](https://github.com/NatVanG/fab-inspector) to run rules-based validation against Fabric items under source control.

## Repository Layout

- `src/`: Fabric artifacts to validate (reports, semantic model, notebook, pipeline).
- `fab-inspector-rules/`: Rule sets used by Fab Inspector.
- `.github/workflows/`: GitHub Actions workflow examples.

## GitHub Workflow Examples

This repo includes two manual (`workflow_dispatch`) examples:

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

## How To Use The Workflows

1. Push this repository (or your fork) to GitHub.
2. Open **Actions** in your repo.
3. Choose one of these workflows:
	 - `fab-inspector-docker`
	 - `fab-inspector-cli`
4. Click **Run workflow**.
5. For `fab-inspector-cli`, optionally set `version` (for example `v3.3.0`) and run.
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
