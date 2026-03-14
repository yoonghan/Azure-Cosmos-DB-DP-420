# Project Overview

This is a learning project for Azure Cosmos DB DP-420 (Designing and Implementing Cloud-Native Applications Using Microsoft Azure Cosmos DB). It serves as a study guide and a reference for Cosmos DB NoSQL.

The notes are open for community contribution and meant to cover key concepts in point structures.

## Technology Stack

*   **Content**: Written entirely in Markdown (`.md`) files inside the `docs/` directory.
*   **Static Site Generator**: [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).
*   **Hosting**: GitHub Pages.
*   **Important Note**: This is **NOT** a Node.js or npm project. There is no `package.json`, and no `npm install` or `npm run` commands are used.

## Building and Running

### Development

1.  **Prerequisites**: Python 3.x is required.
2.  **Dependencies**: Install the required package using `pip install mkdocs-material`.
3.  **Local Preview**: Run `mkdocs serve` in the project root to start a live-reloading local development server.
4.  **Content Structure**: When adding new pages, ensure you update the `nav` structure in `mkdocs.yml`.
5.  **Markdown Extensions**: The project utilizes extensions like `admonition` (to create hints, notes, tips), `pymdownx.details`, and `pymdownx.superfences` to enhance markdown formatting.

### CI/CD

The project includes a GitHub Actions CI/CD pipeline (`.github/workflows/ci.yml`) triggering on pushes to the `master` branch. The pipeline automatically:
*   Installs dependencies.
*   Builds and forcefully deploys the static site to GitHub Pages using `mkdocs gh-deploy --force`.

## Objective

This website is for documentation and community-driven learning purposes for the Cosmos DB NoSQL DP-420 Azure examination. It may not contain every single exam topic and is not purely accurate for the exam, but rather serves as a reference for learning and continuous improvement based on peer reviews.
