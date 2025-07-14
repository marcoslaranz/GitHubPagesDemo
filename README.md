# Deploying a Blazor WASM Application to GitHub Pages

This guide explains how to deploy a Blazor WebAssembly (WASM) application to GitHub Pages, both manually and using GitHub Actions for automation.

---

## Table of Contents

- [Deploying a Blazor WASM Application to GitHub Pages](#deploying-a-blazor-wasm-application-to-github-pages)
  - [Table of Contents](#table-of-contents)
  - [What is GitHub Pages?](#what-is-github-pages)
  - [Deploy Static Files to GitHub Pages](#deploy-static-files-to-github-pages)
    - [Steps](#steps)
    - [Manual Deployment: Static `index.html`](#manual-deployment-static-indexhtml)
  - [Manual Deployment: Blazor WASM](#manual-deployment-blazor-wasm)
  - [Automating Deployment with GitHub Actions](#automating-deployment-with-github-actions)
  - [Summary](#summary)

---

## What is GitHub Pages?

- **Static site hosting service** for HTML, CSS, JavaScript, etc.
- Supports **custom domains**
- Sites are **always public**, even if the repository is private
- Built-in support for **Jekyll** static site generator (not required)
- **Free**, but no SLA (Service Level Agreement)
- Check: Usage limits & Prohibited uses

---

## Deploy Static Files to GitHub Pages

### Steps

1. **Create a GitHub repository.**
2. **Commit static files** (e.g., `index.html`) to the repository.
3. **Configure a branch** as the GitHub Pages source:
   - In repository settings, select a branch (e.g., `gh-pages`) as the source.
   - Or create a branch named `gh-pages`; GitHub will use it automatically.
4. **Wait for GitHub** to deploy the site.

---

### Manual Deployment: Static `index.html`

1. **Open Command Prompt** and create your project folder:
    ```
    D:
    cd Projects
    mkdir GitHubPagesDemo
    cd GitHubPagesDemo
    code .
    ```
2. **Create an `index.html` file** in VS Code:
    - File → New → HTML → Save as `index.html`
    - Type `html:5` and press `<TAB>` to scaffold HTML.
    - Edit the body to include `Hello World!`.

3. **Initialize Git repository:**
    ```
    git init
    git status
    git add --all
    git status
    git commit -m "Initial commit."
    ```
4. **Create GitHub repository** (online) with the same name (`GitHubPagesDemo`).

5. **Connect local repo to GitHub and push:**
    ```
    git remote add origin https://github.com/YourUserName/GitHubPagesDemo
    git branch -M main
    git push -u origin main
    ```

6. **Create and switch to `gh-pages` branch:**
    ```
    git branch gh-pages
    git checkout gh-pages
    git status
    git push --set-upstream origin gh-pages
    ```

7. **Check deployment:**
    - Go to repository Settings → Pages → Deployments.
    - Your site should be live at `https://yourusername.github.io/GitHubPagesDemo/`.

---

## Manual Deployment: Blazor WASM

1. **Switch to main branch:**
    ```
    git checkout main
    git status
    ```

2. **Create a new Blazor WASM project:**
    ```
    dotnet new blazorwasm
    del index.html
    dotnet new gitignore
    git add --all
    git commit -m "Add Blazor WASM."
    git push
    ```

3. **Publish Blazor app:**
    ```
    dotnet publish -c Release -o release
    move release c:\temp
    ```
#### Note: You are moving the published files to a temporary location (`c:\temp\release`). to avoid to push the entire `release` folder to GitHub to the 'main' branch.


4. **Switch to `gh-pages` branch:**
    ```
    git checkout gh-pages
    git status
    ```

5. **Copy published files and prevent GitHub file mangling:**
    ```
    xcopy c:\temp\release\wwwroot\*.* . /s /e
    echo "* binary" > .gitattributes
    git add --all
    git commit -m "Add published Blazor WASM"
    git push
    ```
#### Note: Now that you are in the 'gh-pages' branch, you can copy the published files from the temporary location to the current branch.


6. **Fix the `<base>` tag in `index.html`:**
    - Change:
      ```
      <base href="/" />
      ```
    - To:
      ```
      <base href="/GitHubPagesDemo/" />
      ```
    - Then:
      ```
      git add --all
      git commit -m "Fix index.html"
      git push
      ```

7. **Disable Jekyll processing:**
    ```
    echo > .nojekyll
    git add --all
    git commit -m "Adding .nojekyll file"
    git push
    ```

#### Note: The `.nojekyll` file prevents GitHub from processing your site with Jekyll, which is necessary for Blazor apps. The files starts with _ uderline will not be processed by Jekyll, so the folders won't be pushed to the github pages branch.

8. **Add a custom 404 page:**
    ```
    copy index.html 404.html
    git add --all
    git commit -m "Adding 404 page"
    git push
    ```

---

## Automating Deployment with GitHub Actions

1. **Go to your GitHub repository → Actions.**
2. **Set up a new workflow** and paste the following YAML:

    ```
    name: Deploy to GitHub Pages

    on:
      push:
        branches: [ main ]

    jobs:
      deploy-to-github-pages:
        runs-on: ubuntu-latest
        permissions:
          contents: write
        steps:
          - name: Install libssl
            run: sudo apt-get update && sudo apt-get install -y libssl-dev

          - name: Checkout source code
            uses: actions/checkout@v4

          - name: Setup .NET SDK
            uses: actions/setup-dotnet@v4
            with:
              dotnet-version: '9.0.x'

          - name: Cache NuGet packages
            uses: actions/cache@v3
            with:
              path: ~/.nuget/packages
              key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj') }}
              restore-keys: |
                ${{ runner.os }}-nuget-

          - name: Publish Blazor Project
            run: dotnet publish GitHubPagesDemo.csproj -c Release -o release --nologo

          - name: Update <base> tag in index.html
            run: |
              sed -i 's|<base href="/" />|<base href="/GitHubPagesDemo/" />|g' release/wwwroot/index.html

          - name: Add fallback page
            run: cp release/wwwroot/index.html release/wwwroot/404.html

          - name: Add .nojekyll
            run: touch release/wwwroot/.nojekyll

          - name: Deploy to GitHub Pages
            uses: JamesIves/github-pages-deploy-action@4.1.4
            with:
              branch: gh-pages
              folder: release/wwwroot
              token: ${{ secrets.GITHUB_TOKEN }}
    ```

3. **Every push to the `main` branch** will automatically deploy your Blazor app to GitHub Pages.

---

## Summary

- Create your Blazor WASM app and publish it to static files.
- Commit the output to the `gh-pages` branch.
- Fix the `<base>` tag and disable Jekyll with `.nojekyll`.
- Optionally, automate deployment using GitHub Actions.

---
