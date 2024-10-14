
GitHub Action: Create Zip of Changed Files
==========================================

This GitHub Action is designed to create a zip archive of all files that have been changed in the latest commit. It works on any repository where you want to automate the process of archiving only the changed files, including those in subfolders, while excluding specific directories such as `.github`.

How It Works
------------

This action does the following:

1.  **Checks Out the Code**: The action checks out your repository to access the files.
2.  **Gets the Current Commit Hash**: Retrieves the current commit hash for reference.
3.  **Counts the Commits**: If there is more than one commit, it runs a `git diff` command to list all files changed between the current commit (`HEAD`) and the previous commit (`HEAD^`). If there is only one commit (initial commit), it skips this step and generates an empty list of changed files.
4.  **Copies Changed Files**: The action creates a new directory (`changed_files`) and copies all changed files into it, preserving the original directory structure.
5.  **Zips the Files**: The copied files are then zipped into a `changed_files.zip` archive. The `.github` folder is explicitly excluded from this archive.
6.  **Uploads the Zip File**: The zip file is uploaded as an artifact, which can then be downloaded from the GitHub Actions tab.

Prerequisites
-------------

*   This action requires that your repository has multiple commits. For an initial commit (when there are no prior commits), no files will be archived.

Setup and Usage
---------------

To use this GitHub Action in your repository, follow these steps:

### 1\. Create a Workflow

Add a workflow YAML file to the `.github/workflows` directory in your repository. For example, create a file called `archive-changed-files.yml` with the following content:

    
    name: Create Zip of Changed Files
    
    on:
      push:
        branches:
          - main  # Change this to your target branch
    
    jobs:
      zip_changed_files:
        runs-on: ubuntu-latest
    
        steps:
        - name: Checkout code
          uses: actions/checkout@v3
          with:
            fetch-depth: 0  # Fetch all commits
    
        - name: Print OS Information
          run: echo "Running on: $(uname -a)"
    
        - name: Get the current commit hash
          id: get_commit
          run: |
            commitHash=$(git rev-parse HEAD)
            echo "Current commit: $commitHash"
            echo "::set-output name=commit::$commitHash"
    
        - name: Get list of changed files
          id: changed_files
          run: |
            commitCount=$(git rev-list --count HEAD)
            echo "Number of commits: $commitCount"
            if [ "$commitCount" -gt 1 ]; then
              git diff --name-only HEAD^ HEAD > changed_files.txt
              echo "Changed files:"
              cat changed_files.txt
            else
              echo "No previous commit found. No files changed."
              touch changed_files.txt
            fi
    
        - name: Copy changed files to a new directory
          run: |
            mkdir changed_files
            while IFS= read -r file; do
              if [ -f "$file" ]; then
                destination="changed_files/$(dirname "$file")"
                mkdir -p "$destination"
                cp "$file" "$destination"
              fi
            done < changed_files.txt
    
        - name: Zip the changed files
          run: zip -r changed_files.zip changed_files -x "changed_files/.github/*"
    
        - name: Upload zip file as an artifact
          uses: actions/upload-artifact@v3
          with:
            name: changed-files
            path: changed_files.zip
    

### 2\. Customize the Workflow

You can customize the workflow by:

*   **Branch Targeting**: Change the `main` branch to any branch you want this action to run on.
*   **File and Folder Exclusions**: If you want to exclude other directories or files from being included in the zip, you can modify the `-x` parameter in the zip command.

### 3\. Push Changes to the Repository

Once the workflow file is added to your repository, it will automatically run on every push to the target branch. After the action completes, the zipped archive of changed files will be available as an artifact in the GitHub Actions tab for download.

Detailed Workflow Explanation
-----------------------------

### Step 1: Checkout Code

The `actions/checkout@v3` step checks out the repository’s latest state, including all files and directories. The `fetch-depth: 0` option ensures that all commits are fetched so that the `git diff` command works with the full commit history.

### Step 2: Print OS Information

This step is optional and prints out the OS information for debugging purposes, verifying that the workflow is running on the correct environment.

### Step 3: Get the Current Commit Hash

This step captures the current commit hash and stores it in the workflow’s output. This hash can be useful for debugging or referencing the exact commit for the changes being processed.

### Step 4: Get List of Changed Files

This step runs a `git diff` command to compare the current commit (`HEAD`) with the previous commit (`HEAD^`). It generates a list of files that have changed between these two commits and stores them in `changed_files.txt`. If this is the first commit in the repository, it creates an empty `changed_files.txt`.

### Step 5: Copy Changed Files to a Directory

This step reads the `changed_files.txt` and copies each file to the `changed_files` directory, preserving the directory structure. Only files (not directories) are copied.

### Step 6: Zip the Changed Files

The action zips the `changed_files` directory into `changed_files.zip`. The `.github` directory is explicitly excluded from the zip archive using the `-x` option.

### Step 7: Upload Zip as an Artifact

The zipped file is uploaded as an artifact named **changed-files**. You can download this artifact from the GitHub Actions tab.

Excluding Specific Files and Folders
------------------------------------

By default, the `.github` directory is excluded from the zip file. If you want to exclude additional directories or specific files, you can modify the `-x` option in the `zip` command. For example, to exclude the `docs` folder and `.env` files, update the command as follows:

    run: zip -r changed_files.zip changed_files -x "changed_files/.github/*" "changed_files/docs/*" "*.env"

Example Use Case
----------------

This action is useful in scenarios where you want to:

*   Archive only the files that were changed in a specific commit.
*   Keep track of changes made between commits.
*   Automate file archiving as part of a CI/CD pipeline.

License
-------

This GitHub Action is licensed under the MIT License. Feel free to customize and use it in your projects.