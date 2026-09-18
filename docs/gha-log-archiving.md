---
layout: default
title: GitHub Actions Log Archiving Guide
nav_order: 10
---

# GitHub Actions Log Archiving Guide

This guide explains how to configure, use, and manage the automated **GitHub Actions Log Archiving** workflow ([archive_GHA_logs.yml](file:///e:/test_archiving_GHA_logs/.github/workflows/archive_GHA_logs.yml)), which backs up workflow execution logs to Cloudflare R2 object storage.

---

## Overview

GitHub Actions retains logs for a maximum of 90 days by default. To maintain long-term audit trails, compliance, and diagnostic history, the `archive_logs_to_r2` workflow automatically extracts logs from completed workflow runs and syncs them to a Cloudflare R2 bucket (S3-compatible storage).

### Key Features
- **Automatic Execution**: Triggers automatically whenever any workflow in the repository finishes (`types: [completed]`).
- **Self-Loop Protection**: Automatically skips execution when the archiving workflow itself completes to prevent infinite loops.
- **Manual Trigger**: Supports manual execution (`workflow_dispatch`) with an optional `target_run_id` parameter.
- **Structured Storage**: Unzips log archives and preserves folder structure in R2 under `YYYYMMDD-<repo>-<workflow>-<runId>/`.

---

## Prerequisites & Setup

Before enabling the workflow, you must prepare Cloudflare R2 storage and configure GitHub Repository Secrets.

### 1. Cloudflare R2 Configuration

1. Log in to your [Cloudflare Dashboard](https://dash.cloudflare.com/) and navigate to **R2 Object Storage**.
2. Create a new bucket (e.g., `gha-logs-archive`).
3. Under **R2 Overview** (on the right sidebar), copy your **Account ID**.
   - Your S3 API Endpoint will follow this format:  
     `https://<ACCOUNT_ID>.r2.cloudflarestorage.com`
4. Go to **Manage R2 API Tokens** and click **Create API Token**.
5. Set permissions to **Object Read & Write** for your target bucket.
6. Copy the generated **Access Key ID** and **Secret Access Key**.

---

## Required GitHub Repository Secrets

In your GitHub repository, navigate to **Settings** > **Secrets and variables** > **Actions**, and add the following repository secrets:

| Secret Name            | Description                              | Example / Format                                |
| :--------------------- | :--------------------------------------- | :---------------------------------------------- |
| `R2_ACCESS_KEY_ID`     | Cloudflare R2 API Access Key ID          | `a1b2c3d4e5f6...`                               |
| `R2_SECRET_ACCESS_KEY` | Cloudflare R2 API Secret Access Key      | `7890abcdef...`                                 |
| `R2_ENDPOINT`          | Cloudflare R2 S3 Endpoint URL            | `https://<account-id>.r2.cloudflarestorage.com` |
| `R2_BUCKET_NAME`       | Name of your target Cloudflare R2 bucket | `gha-logs-archive`                              |

---

## How to Use

### 1. Automatic Archiving
Once the secrets are configured, the workflow runs automatically whenever **any workflow** in the repository completes. No manual action is required.

### 2. Manual Archiving (`workflow_dispatch`)
If you need to archive a specific past workflow run manually:

1. Go to the **Actions** tab in your GitHub repository.
2. Select **archive_logs_to_r2** from the left sidebar.
3. Click **Run workflow**.
4. *(Optional)* Enter the specific `Workflow Run ID` into the **target_run_id** field.
5. Click **Run workflow** to initiate archiving.

---

## Storage Structure in Cloudflare R2

Logs are extracted and synchronized to your Cloudflare R2 bucket using the following naming structure:

```text
s3://<R2_BUCKET_NAME>/<YYYYMMDD>-<repo-name>-<workflow-name>-<run-id>/
├── 1_build.txt
├── 2_test.txt
└── ...
```

---

## Security & Resource Management

- **Permissions Scope**: Uses GitHub's default `${{ secrets.GITHUB_TOKEN }}` with `actions: read` permission scope to retrieve workflow logs securely via the GitHub REST API.
- **Disk Cleanup**: Downloaded ZIP files and temporary extraction directories are deleted immediately after uploading to free up runner disk space.

